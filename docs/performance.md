# Performance

This page is about where the time goes in a zug application, measured
rather than guessed, and what to do about the parts that are slow.

The short version: zug's own work is rarely the problem. A rebuild of a
thousand-row screen takes under 10 ms, and an animation frame a fraction
of a millisecond. What costs real time is AppKit making and drawing
native views, so the advice comes down to not asking it to make and
draw views nobody can see.

## How these were measured

All of the numbers come from `tools/bench.zu`, which you can run
yourself:

```console
$ zuri run tools/bench.zu
$ zuri run tools/bench.zu --rounds 15 --json tmp/bench.json
```

It waits until the machine is quiet (a one-minute load average under 1.5
by default), runs each measurement once to warm up and then nine times,
and reports the median and the fastest run. On macOS it opens a window
while the AppKit measurements run, which takes keyboard focus for a
moment.

The run below was on an Apple M5 with 10 cores and 16 GB of memory,
macOS 26.5.1, Zuri 0.1.0, at a load average of 1.37. It's a laptop that
was also doing other work, so treat small differences as noise.

A frame is timed until zug has finished with it: built, laid out and
committed to the native views. AppKit draws on screen after that, on its
own schedule, and the measurements that include drawing say so.

## Starting up

| | Median | Fastest |
| --- | --- | --- |
| Start `zuri` and import zug | 32 ms | 28 ms |
| Start a small application until its first frame | 164 ms | 140 ms |

The first frame is a window with a heading, a text field and a button,
counted from the moment the `zuri run` process is started. Importing zug
is about a fifth of it; the rest is starting AppKit, making the window
and its views, and drawing the first frame.

## Crossing into AppKit

| | Median | Fastest |
| --- | --- | --- |
| Call an Objective-C method | 99 ns | 95 ns |
| Call Zuri from Objective-C and return | 211 ns | 203 ns |

zug talks to AppKit through Zuri's FFI, with no compiled glue. A call is
about a tenth of a microsecond, so even a frame that touches a few
hundred views spends well under a millisecond crossing over.

## Building and rebuilding

The test screen is a scroll view holding 1,000 rows, each a label, a
spacer and a button: 2,000 native views in all.

| | Headless | macOS |
| --- | --- | --- |
| Mount the 1,000 rows | 28 ms | 150 ms |
| Mount them and have AppKit draw them | | 986 ms (717 ms fastest) |
| Tear them down | | 88 ms |
| Rebuild them with nothing changed | 6.9 ms | 8.6 ms |
| Rebuild them with one label changed | 8.0 ms | 9.9 ms |
| Rebuild them with every label changed | 10.3 ms | 28.1 ms |
| Lay them out again for a new width | 2.4 ms | 8.5 ms (resizing the window) |
| Rebuild one `Observer` next to them | 0.01 ms | |

The headless column is zug on its own, with views that are just records
in memory. The difference between the two columns is AppKit.

A few things to take from this:

- **Rebuilding is cheap; making views is not.** Rebuilding all 1,000
  rows, comparing every widget with the last build, costs about as much
  as one frame at 120 Hz. Mounting them costs fifteen times that, almost
  all of it AppKit creating and measuring buttons and labels.
- **AppKit's first draw is the expensive part.** Drawing the mounted rows
  for the first time takes most of a second. macOS draws every view in a
  window the first time it shows, including the rows scrolled out of
  sight, which came to between half a millisecond and a millisecond per
  row here.
- **Keep rebuilds close to what changed.** Calling `set_state()` on the
  state that owns the whole screen rebuilds the whole screen. Put the
  changing part in its own `StatefulWidget`, or wrap it in an `Observer`
  of an `ObservableValue`, and the rebuild drops to a hundredth of a
  millisecond, however big the rest of the screen is.
- **Alike controls are cheap to measure.** zug asks AppKit for a
  control's natural size once for each distinct look, so a list of a
  thousand "Open" buttons is measured once. Buttons with different
  titles each cost about a third of a millisecond to measure.

## Long lists and tables

| | Headless | macOS |
| --- | --- | --- |
| Scroll a 100,000-item `ListView.builder` by five items | 1.0 ms | |
| Show a 100,000-row `DataTable` until its cells are in | | 15.7 ms |
| Reload it with new data until its cells are in | | 6.5 ms |

This is the answer to the drawing cost above. `ListView.builder` builds
only the rows in view, plus a few either side, and builds new ones as
you scroll, so a list of a hundred thousand items costs about what a
screenful does. A `DataTable` goes further: it never builds widgets for
its rows at all, and asks for the text of the cells it's about to draw.

So for anything long, or anything whose length you don't control, use
`ListView.builder` or `DataTable`. Save a plain `Column` in a
`ScrollView` for content you know is short, like a settings page.

## Animation

| | Median | Fastest |
| --- | --- | --- |
| zug's work for a frame of a box animating alone | 0.31 ms | 0.05 ms |
| Frames in a second of that animation | 121 | |
| zug's work for a frame of a box animating beside 1,000 rows | 0.11 ms | 0.03 ms |
| Frames in a second of that animation | 121 | |

The display here refreshes at 120 Hz, and both animations kept up with
it. While something animates, zug draws a frame for every refresh and
rebuilds only the widgets that read the animation. When nothing
animates and nothing changes, it draws no frames at all, so an idle
application leaves the processor alone.

The thousand rows next to the animated box didn't slow it down, because
they were already drawn when it started. Start an animation in the same
moment as a big screen first appears and it will stutter while AppKit
draws that screen for the first time.

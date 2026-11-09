# Getting started

This page takes you from an empty directory to a running window, and
explains the handful of ideas everything else in zug builds on.

You need Zuri 0.1 or later and macOS 11 or later.

## A new project

```console
$ zuri init counter
$ cd counter
$ zuri install --git https://github.com/zuri-lang/zug.git
```

## The first window

Replace `app/index.zu` with this:

```zuri
import zug { * }

class Counter < StatefulWidget {
  create_state() {
    return CounterState()
  }
}

class CounterState < State {
  var count = 0

  build(context) {
    return Padding(EdgeInsets.all(20), Column([
      Text('Pressed ${self.count} times', { style: Style.of(context).headline }),
      Button('Press', {
        role: ButtonRole.PRIMARY,
        on_pressed: @{
          self.set_state(@{
            self.count++
          })
        },
      }),
    ], { spacing: 12, align_items: AlignItems.START }))
  }
}

def main() {
  run_app(App({
    title: 'Counter',
    windows: [Window('Counter', Counter(), { size: Size(320, 160) })],
  }))
}
```

and run it with `zuri run`. A window opens with a heading and a real macOS button. Each click
updates the heading.

## What just happened

**Everything on screen is a widget.** `Padding`, `Column`, `Text` and
`Button` are widgets, and so are `Window` and `App`. A widget is a small
description of a piece of interface; you make new ones whenever you like
and throw them away just as freely.

**`build()` describes the interface for the current state.** `Counter` is
a `StatefulWidget`, which means it has a `State` that lives as long as
the counter is on screen. Its `build()` returns the widgets for whatever
`count` is right now. You never reach into a button to change its label.

**`set_state()` asks for a new description.** Change your fields inside
it, and zug calls `build()` again, compares the new widgets with the old
ones, and changes only the native views that differ. Here that's one
label's text.

**The controls are AppKit's.** zug makes an `NSButton` for `Button` and
an `NSTextField` for `Text`, so you get the platform's look, keyboard
behaviour, accessibility and dark mode without asking for them.

**`run_app()` runs until the application quits.** The `App` lists the
windows that are open; closing the last one quits, unless you say
otherwise.

## Controls show what you give them

One idea is worth learning now, because every control follows it. A
control shows the value you pass it, and tells you when the user wants
a different one:

```zuri,ignore
Checkbox(self.remember, {
  label: 'Remember me',
  on_changed: @(value) {
    self.set_state(@{
      self.remember = value
    })
  },
})
```

If `on_changed` doesn't store the new value, the checkbox goes back to
showing `self.remember`. Your state stays the one source of truth, and
what's on screen can't drift away from it. A control with no
`on_changed` at all can't be changed, so it's shown disabled.

Text fields work the same way when you give them `text`, or you can hand
them a `TextController` and let it hold the text; see
[Text input](text-input.md).

## Where to go next

- [Widgets and state](widgets-and-state.md) covers state properly: its
  lifecycle, keys, sharing data down the tree and observable values.
- [Layout](layout.md) explains how rows, columns and boxes decide sizes.
- [The examples](../examples) are single scripts you can run with
  `zuri run examples/controls.zu` and friends, plus a whole calculator
  application in `examples/calculator`.

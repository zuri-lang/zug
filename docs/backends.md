# Writing a backend

Everything zug knows about a platform lives in a backend. The rest of
zug, from widgets to layout to diffing, never mentions a platform, so
bringing zug to Linux, Windows or somewhere else means writing a backend
and nothing more. This page is for whoever writes one.

zug ships two:

- **headless**, where views are records in memory, text is measured by a
  fixed rule and time moves only when told to. Tests run on it, and it's
  the reference for a new backend, because it does everything the
  contract asks in the plainest way.
- **macos**, which drives AppKit through Zuri's FFI and the Objective-C
  runtime.

## The contract

The contract is four classes in `zug.backend`, and a backend subclasses
them.

**`Backend`** is the platform. It starts and runs the event loop, asks
for frames, runs timers, keeps a clock, says how often the display
refreshes (`frame_interval()`, which animations draw at), creates
windows and views, sets the menu bar, shows dialogs, and reaches the
clipboard, the appearance, the screens and the system's standard actions
(`perform_role()`).

**`ViewHost`** is one native view. It's created with all of its props,
then given only the ones that change, through `update()`. It measures
itself when asked, reports its first baseline, says how much room its
own drawing takes around its children (`content_insets()`) and how big
each area for children is (`slot_size()`), takes the frame zug gives it,
and holds child views in slots.

**`WindowHost`** is a window: a `ViewHost` whose content area holds the
top-level views, with its own size, `show()`, `close()` and
`clear_focus()`.

**`BackendDelegate`** is what the backend calls on zug: `on_frame()` to
draw, `on_event_handled()` after each event, and the appearance, quit
and reopen notifications.

`Painter` is what a `Canvas` paint function draws with, and a backend
with canvases supplies one.

The view kinds, and the props and events of each, are listed in
`app/backend/kinds.zu`. That file is the specification: for every prop,
its type, its starting value and whether it can change the view's size;
for every event, its name and the prop it changes, if any.

## The rules

- **Props and event payloads are plain data**: numbers, strings, bools,
  lists, dictionaries, and zug's value classes such as `Color`, `Size`
  and `EdgeInsets`. Native objects never cross the contract, which keeps
  a backend that runs in another process, such as one driving a browser
  over a socket, possible.
- **Layout is zug's.** A backend measures when asked and puts views where
  it's told. Coordinates are logical points from the top left of the
  parent's content area, with y growing downwards.
- **Controlled props follow the widget.** When an event changes a prop,
  such as a checkbox's `changed` changing `value`, the view reports the
  new value in the payload and keeps showing it. zug then sends the
  declared value back unless the application accepted the change. Never
  send a value back to a control that already shows it; for text it
  would move the cursor.
- **One thread.** zug calls a backend only from the thread its `run()`
  was called on, and the backend calls `dispatch` and the delegate only
  from that thread.
- **Events go through `dispatch`, then the delegate.** A view reports an
  event by calling the `dispatch` function it was made with, then
  `delegate.on_event_handled()`. Events that ask a question, such as a
  table asking for a cell's text, take their answer from what `dispatch`
  returns and aren't followed by `on_event_handled()`.
- **Errors stay inside.** An error raised while handling a native
  callback must never unwind into the platform. Catch it and pass it to
  `report_error()`.

## Adding one

1. Make the backend: classes extending `Backend`, `ViewHost`,
   `WindowHost` and `Painter`, and a function that returns a new
   backend.
2. Implement every kind in `kinds.zu`. `supports_kind()` says which are
   done, and `create_view()` raises `UnsupportedError` for the rest, so
   whoever hits a missing kind is told which one on which backend.
3. Report what the platform can do from `capabilities()`. Without
   `menu_bar`, for example, `MenuBar` does nothing.
4. Register it, with a function that loads it and one that says whether
   it suits the running platform:

   ```zuri,ignore
   register_backend('gtk', @{
     return zuri.import_file(os.join_paths(os.dir_name(__file__), 'gtk', 'index.zu')).create()
   }, @() => os.platform == 'linux')
   ```

   The loader runs only when the backend is chosen, so nothing of one
   platform is ever loaded on another. A backend inside zug registers in
   `app/backends/index.zu`; one in its own package registers when it's
   imported.
5. Check it against the contract:

   ```console
   $ zuri run tools/check_backend.zu gtk --load ../zug-gtk/register.zu
   ```

   The check works every view kind through the contract, times the
   timers and frames, and then builds a gallery of every widget, changes
   nearly every prop of it, changes it back and tears it down, listing
   whatever didn't pass. `--load` imports the file that registers your
   backend first. `check_backend()` in `zug.testing` does the same from a
   test, and `examples/conformance.zu` shows it on the platform you're
   on.
6. Once it can run without a display, run zug's own tests on it with
   `ZUG_BACKEND=gtk zuri test`, and write tests of your own that check
   its native views directly, the way `tests/macos.test.zu` does for
   AppKit.

## Platforms beyond the desktop

A desktop platform reached through C, such as GTK on Linux or Win32 on
Windows, needs only its backend, written the way the macOS one is. Two
others need more:

- **Android** needs Zuri itself to run on Android, and a bridge to its
  Java views.
- **The web**, with Zuri running on a server, needs the browser to
  measure text, which means a round trip where the contract expects an
  answer at once. Keeping a model of the fonts' metrics on the server is
  the likely way round that.

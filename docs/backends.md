# Writing a backend

A backend is everything zug knows about a platform. The core lays
widgets out, diffs their props between builds and routes events, and it
does all of that through the classes in `app/backend`. A backend
implements those classes for one platform. Nothing else in zug changes.

Two backends ship with zug:

- `app/backends/headless`: views kept in memory, text measured by a fixed
  rule, time that moves only when told to. It is what the tests run on,
  and it is the reference for a new backend, because it does everything
  the contract asks in the plainest way.
- `app/backends/macos`: AppKit, reached through `ffi` and the Objective-C
  runtime.

## The contract

`app/backend/contract.zu` defines four classes.

**`Backend`** is the platform. It starts and runs the event loop, asks
for frames, runs timers, creates windows and views, sets the menu bar,
shows dialogs, and reaches the clipboard, the appearance, the screens
and the system's standard actions (`perform_role()`).

**`ViewHost`** is one native view. It is created with all of its props,
then given only those that change through `update()`. It measures itself
when asked, reports its first baseline, says how much space its own
drawing takes around its children (`content_insets()`) and how big each
area for children is (`slot_size()`), takes the frame zug gives it, and
holds child views in slots.

**`WindowHost`** is a window: a `ViewHost` whose content area holds the
top-level views, with its own size, `show()`, `close()` and
`clear_focus()`.

**`BackendDelegate`** is what the backend calls on zug: `on_frame()` to
draw, `on_event_handled()` after each event, and the appearance, quit
and reopen notifications. The runtime implements it.

`app/backend/painter.zu` defines `Painter`, what a `Canvas` paint
function draws with.

## The rules

- **Props and event payloads are plain data**: numbers, strings, bools,
  lists, dictionaries, and zug's value classes such as `Color`, `Size`
  and `EdgeInsets`. Native objects never cross the contract. This keeps a
  backend that runs in another process, such as one driving a browser over
  a socket, possible.
- **Layout is zug's.** A backend measures when asked and puts views where
  it is told. Coordinates are logical points from the top left of the
  parent's content area, with y growing downwards.
- **Each view kind takes the props and reports the events
  `app/backend/kinds.zu` lists for it.** That file is the specification:
  a prop's type, its starting value, and whether it can change the view's
  size; an event's name, and the prop it controls if any.
- **Controlled props follow the widget.** When an event controls a prop,
  such as a checkbox's `changed` controlling `value`, the backend reports
  the new value in the payload. zug then sends the declared value back
  unless the application accepted the change. A backend must not send a
  value back to a control that already shows it, because for text that
  would move the cursor.
- **One thread.** zug calls a backend only from the thread its `run()`
  was called on, and the backend calls `dispatch` and the delegate only
  from that thread.
- **Events go through `dispatch`, then the delegate.** A view reports an
  event by calling the `dispatch` function it was created with, then
  `delegate.on_event_handled()`. What `dispatch` returns answers events
  that ask a question, such as a table asking for a cell's text, which are
  not followed by `on_event_handled()`.
- **Errors stay inside.** An error raised by zug while handling a native
  callback must not unwind into the platform. Catch it and pass it to
  `report_error()`.

## Adding one

1. Make `app/backends/<name>/index.zu`, with classes extending
   `Backend`, `ViewHost`, `WindowHost` and `Painter`, and a `create()`
   function that returns the backend.
2. Implement every kind in `kinds.zu`. `supports_kind()` says which are
   done; `create_view()` raises `UnsupportedError` for the others, which
   tells the developer which kind is missing on which backend.
3. Report the capabilities the platform has from `capabilities()`. A
   missing `menu_bar` makes the menu bar a no-op, for example.
4. Register it in `app/backends/index.zu` with a loader and a test of the
   running platform. The loader imports the backend only when it is
   chosen, so nothing of one platform is loaded on another.
5. Run zug's tests on it with `ZUG_BACKEND=<name> zuri test` once it can
   run without a display, and write tests like `tests/macos.test.zu` that
   check its native views directly.

## Platforms beyond the desktop

The contract leaves room for platforms that are not desktops, but two
need more than a backend:

- **Android** needs Zuri itself to run on Android, and a bridge to the
  Java views.
- **The web**, with Zuri running on a server, needs the browser to
  measure text, which means a round trip where the contract expects an
  answer at once. A text-metrics model kept on the server is the likely
  way around that.

A desktop platform reached through C, such as GTK on Linux or Win32 on
Windows, needs only its backend, written the way the macOS one is.

## The macOS backend

Its files, in `app/backends/macos`:

| File | Holds |
| --- | --- |
| `_runtime.zu` | The Objective-C runtime: typed `objc_msgSend` bindings cached per signature, selectors and classes, strings, autorelease pools, classes made at run time, and blocks. |
| `_classes.zu` | The classes zug defines, such as a flipped view and the delegates, and the registry from native object to Zuri host. |
| `_loop.zu` | The run loop observer that draws frames before the loop sleeps, and timers. |
| `_cocoa.zu` | Colours, fonts, alignment and shortcuts in Cocoa's terms. |
| `_views.zu` | The view base class, text, buttons and boxes. |
| `_controls.zu` | Controls that choose values, indicators, separators, group boxes and images. |
| `_text_input.zu` | Text fields, secure fields, search fields, combo boxes and text views. |
| `_containers.zu` | Scroll, tab and split views, tables and visual effect views. |
| `_canvas.zu` | The canvas and its painter, and the pointer area. |
| `_window.zu` | Windows and key events. |
| `_menus.zu` | The menu bar and context menus. |
| `_dialogs.zu` | Alerts and open and save panels. |
| `index.zu` | `MacBackend`. |

Every method zug gives an Objective-C class runs inside a guard that
reports errors instead of raising them into AppKit.

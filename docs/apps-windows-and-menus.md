# Applications, windows and menus

An application is a widget too. `App` holds its windows and its menu
bar, and you change either by building it differently, the same way you
change anything else.

## The application

```zuri
import zug { * }

run_app(App({
  title: 'Notes',
  windows: [Window('Notes', Center(Text('No notes yet')), { size: Size(640, 420) })],
}))
```

`run_app(app)` starts the application and returns when it quits. The
`App` takes:

| Option | What it does |
| --- | --- |
| `windows` | The windows that are open (required). |
| `title` | The application's name, shown in its menu. |
| `menu_bar` | Its menus; see below. |
| `style` | A `StyleSheet` for everything in it; see [Styling](styling.md). |
| `quit_on_last_window_closed` | Whether closing the last window quits (true). |
| `on_quit_request` | Asked before quitting; return `false` to keep running, say to ask about unsaved changes first. |
| `on_reopen` | Called when the dock icon is clicked with no windows open. |

`quit_app()` quits from code, asking `on_quit_request` first, and
`current_runtime()` returns the running application's runtime, or `nil`.

## Windows

The windows listed in the `App` are the windows that are open. To open
one, add it to the list and rebuild; to close one, take it out:

```zuri
import zug { * }

class Studio < StatefulWidget {
  create_state() {
    return StudioState()
  }
}

class StudioState < State {
  var inspector = false

  build(context) {
    var windows = [Window('Studio', Center(Button('Toggle the inspector', { on_pressed: @{
      self.set_state(@{
        self.inspector = !self.inspector
      })
    } })), { size: Size(480, 300) })]

    if self.inspector {
      windows.append(Window('Inspector', Center(Text('Properties')), {
        size: Size(220, 300),
        floating: true,
        on_closed: @{
          self.set_state(@{
            self.inspector = false
          })
        },
      }))
    }

    return App({ title: 'Studio', windows })
  }
}

run_app(Studio())
```

When the user closes a window, `on_close_request` is asked first (return
`false` to keep it open), then `on_closed` is called. Take the window out
of your list there. A closed window left in the list stays closed, but
its widgets and their state stay alive for nothing.

`Window(title, content, options)` takes:

| Option | What it does |
| --- | --- |
| `size` | The content area's size, 480 by 320 when left out. |
| `min_size`, `max_size` | Limits for resizing. |
| `size_to_content` | Size the window to its content instead. |
| `resizable`, `closable`, `minimizable` | Each true when left out. |
| `title_bar` | `TitleBar.NORMAL`, `TRANSPARENT` with content reaching under it, or `HIDDEN`. |
| `background` | A `Color` for the window behind its content. |
| `position` | Where its top left goes on the screen; centred when left out. |
| `floating` | Keep it above the application's other windows. |
| `visible` | Set false to keep the window and its state but hide it. |
| `on_resize` | Called with the new content `Size`. |
| `on_focus_changed` | Called with whether it became the key window. |

## Menus

On macOS the menu bar belongs to the application. Give the `App` a
`MenuBar`, and zug adds the standard application menu (About, Hide,
Quit and the rest), and Edit and Window menus if you don't make your
own:

```zuri
import zug { * }

var menus = MenuBar([
  Menu('File', [
    MenuItem('New', { shortcut: 'cmd+n', on_selected: @{} }),
    MenuItem('Open…', { shortcut: 'cmd+o', on_selected: @{} }),
    MenuItem.separator(),
    MenuItem('Export', { items: [
      MenuItem('PDF', { on_selected: @{} }),
      MenuItem('Markdown', { on_selected: @{} }),
    ] }),
  ]),
  Menu('View', [
    MenuItem('Show Sidebar', { shortcut: 'cmd+shift+s', checked: true, on_selected: @{} }),
  ]),
], {
  app_items: [MenuItem('Settings…', { shortcut: 'cmd+,', on_selected: @{} })],
})

run_app(App({ title: 'Notes', menu_bar: menus, windows: [Window('Notes', Text('Hello'))] }))
```

A `MenuItem(label, options)` takes `on_selected`, a `shortcut`,
`enabled`, `checked` for a tick mark, `items` for a submenu, or a `role`.
`app_items` go into the application menu, where Settings belongs.

A `role` makes an item do one of the system's standard actions, the way
it would in any Mac application: `about`, `hide`, `hide_others`,
`show_all`, `quit`, `undo`, `redo`, `cut`, `copy`, `paste`, `delete`,
`select_all`, `minimize`, `zoom`, `close`, `fullscreen` and
`bring_all_to_front`. Copy then copies from whatever text field has
focus, with no code of yours.

Menus are rebuilt with the `App`, so an item's `checked` or `enabled`
can come straight from your state.

When you want an item to use the standard action if there is one, and do
something of your own otherwise, call `perform_role()`. It returns
whether anything carried the action out:

```zuri,ignore
MenuItem('Copy', { shortcut: 'cmd+c', on_selected: @{
  if !perform_role('copy') {
    Clipboard.write_text(self.result)
  }
} })
```

## The keyboard

The focused control gets keys first: a text field takes what's typed
into it, a button takes Space. Keys it doesn't use go to the
`Shortcuts` and `KeyListener` widgets in the window, the deepest first,
until one handles the key.

```zuri
import zug { * }

var keypad = Shortcuts({
  'escape': @{ echo 'clear' },
  'cmd+backspace': @{ echo 'clear everything' },
  '+': @{ echo 'plus' },
}, KeyListener(Text('Type a digit'), {
  on_key: @(event) {
    if event.characters.is_number() {
      echo 'digit ${event.characters}'
      return true
    }

    return false
  },
}))
```

`Shortcuts(bindings, child)` maps bindings to functions. A binding is a
key with any modifiers, such as `cmd+shift+k`, `alt+left` or `f5`. A
single character with no modifiers, such as `+`, matches however the
keyboard types it, so you don't have to know that plus is shift and
equals on one layout and something else on another. The named keys are
`enter`, `escape`, `backspace`, `delete`, `tab`, `space`, the arrows
`left`, `right`, `up` and `down`, `home`, `end`, `page_up`, `page_down`,
and `f1` to `f12`, and the modifiers are `cmd`, `shift`, `alt` and
`ctrl`.

`KeyListener(child, { on_key })` sees every key that reaches it. Its
function gets a `KeyEvent` with `key` (a named key, or the character
without modifiers), `characters` (what the key typed), `modifiers`,
`repeat` and `matches(binding)`, and returns `true` for a key it used.

Keys typed into a text field never reach shortcuts. When you want a
window's keys to go to its shortcuts, such as after the user presses
Return in a field, call `clear_focus(context)`.

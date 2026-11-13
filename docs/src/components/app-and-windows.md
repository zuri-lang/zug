# The application

[Components](index.md) › The application

An application is a widget too. `App` holds the windows that are open
and the menu bar, and you change them by building the `App` differently,
the same way you change anything else.
[Applications, windows and menus](../apps-windows-and-menus.md) walks
through how they fit together.

## App

`App(options)` is the root of an application. Pass it to `run_app()`,
which shows its windows and returns when the application quits.

```zuri
import zug { * }

class Notes < StatefulWidget {
  create_state() {
    return NotesState()
  }
}

class NotesState < State {
  var dark = false

  build(context) {
    var content = Center(
      Checkbox(self.dark, {
        label: 'Dark appearance',
        on_changed: @(on) {
          self.set_state(@{
            self.dark = on
          })
        },
      }),
    )

    return App({
      title: 'Notes',
      appearance: self.dark ? Appearance.DARK : Appearance.SYSTEM,
      windows: [
        Window('Notes', content, { size: Size(360, 200) }),
      ],
    })
  }
}

run_app(Notes())
```

The `App` here is built by a `StatefulWidget`, so its options can come
from state. That's how an application opens windows, changes its menus
and switches its appearance.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `windows` | `list` | required | The `Window`s that are open. |
| `title` | `string` | `''` | The application's name, shown in its menu. |
| `menu_bar` | `MenuBar` | the standard menus | The menu bar. See [MenuBar, Menu and MenuItem](#menubar-menu-and-menuitem). |
| `style` | `StyleSheet` | the platform's | The text styles, accent colour and spacing for everything in the application. See [Styling](../styling.md#a-style-sheet-for-the-application). |
| `appearance` | `string` | `Appearance.SYSTEM` | `Appearance.LIGHT` or `DARK` shows the whole application that way, whatever the system's setting. `SYSTEM` follows the system. |
| `quit_on_last_window_closed` | `bool` | `true` | Quits when the user closes the last window. |
| `on_quit_request` | `function` | none | Asked before the application quits. Return `false` to keep it running, for example to ask about unsaved changes first. |
| `on_reopen` | `function` | none | Called when the user clicks the Dock icon while no windows are open. |

`quit_app()` quits from code, asking `on_quit_request` first.

## Window

`Window(title, content, options)` is a window with a title and the
widget to show inside it. The windows in the `App`'s list are the ones
that are open: add one and rebuild to open it, take it out to close it.

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
    var main = Center(
      Button('Show Inspector', {
        on_pressed: @{
          self.set_state(@{
            self.inspector = true
          })
        },
      }),
    )

    var windows = [
      Window('Studio', main, {
        size: Size(480, 300),
        min_size: Size(320, 200),
      }),
    ]

    if self.inspector {
      windows.append(
        Window('Inspector', Center(Text('No selection')), {
          size: Size(220, 300),
          floating: true,
          minimizable: false,
          on_closed: @{
            self.set_state(@{
              self.inspector = false
            })
          },
        }),
      )
    }

    return App({ title: 'Studio', windows })
  }
}

run_app(Studio())
```

When the user closes a window, `on_close_request` is asked first, and
then `on_closed` is called. Take the window out of your list in
`on_closed`, as the inspector does. A closed window left in the list
stays closed, but its widgets and their state stay alive for nothing.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `size` | `Size` | `Size(480, 320)` | The size of the content area. |
| `min_size` | `Size` | none | The smallest the user can make the window. |
| `max_size` | `Size` | none | The largest the user can make the window. |
| `size_to_content` | `bool` | `false` | Sizes the window to fit its content instead of `size`. |
| `position` | `Point` | centred | Where the window's top left goes on the screen. |
| `resizable` | `bool` | `true` | Whether the user can resize the window. |
| `closable` | `bool` | `true` | Whether the window has a close button. |
| `minimizable` | `bool` | `true` | Whether the window can be minimised to the Dock. |
| `title_bar` | `string` | `TitleBar.NORMAL` | `NORMAL`, `TRANSPARENT` for content that reaches up under the title bar, or `HIDDEN` for no title. |
| `background` | `Color` | the system's | The colour behind the window's content. |
| `floating` | `bool` | `false` | Keeps the window above the application's other windows, as a palette or inspector. |
| `visible` | `bool` | `true` | Set to `false` to hide the window but keep it and its state. |
| `on_close_request` | `function` | none | Asked when the user tries to close the window. Return `false` to keep it open. |
| `on_closed` | `function` | none | Called once the window has closed. |
| `on_resize` | `function` | none | Called with the new content `Size` when the window is resized. |
| `on_focus_changed` | `function` | none | Called with `true` when the window becomes the one the user is working in, and `false` when it stops being that window. |

For a question that needs an answer before the user carries on, such as
"Save changes?", use [`show_dialog()`](../services.md#dialogs-of-your-own)
instead of a second window. A dialog appears as a sheet on its window
and gives you a promise of the answer.

## MenuBar, Menu and MenuItem

On macOS the menu bar belongs to the application, not to a window. Give
the `App` a `MenuBar(menus, options)` of `Menu(title, items)`s, and zug
adds the standard application menu, with About, Hide and Quit, and Edit
and Window menus unless you make your own.

```zuri
import zug { * }

class Editor < StatefulWidget {
  create_state() {
    return EditorState()
  }
}

class EditorState < State {
  var sidebar = true
  var saved = 0

  build(context) {
    var menus = MenuBar(
      [
        Menu('File', [
          MenuItem('New', { shortcut: 'cmd+n', on_selected: @{} }),
          MenuItem('Save', {
            shortcut: 'cmd+s',
            on_selected: @{
              self.set_state(@{
                self.saved++
              })
            },
          }),
          MenuItem.separator(),
          MenuItem('Export', {
            items: [
              MenuItem('PDF…', { on_selected: @{} }),
              MenuItem('Markdown…', { on_selected: @{} }),
            ],
          }),
        ]),
        Menu('View', [
          MenuItem('Show Sidebar', {
            shortcut: 'cmd+alt+s',
            checked: self.sidebar,
            on_selected: @{
              self.set_state(@{
                self.sidebar = !self.sidebar
              })
            },
          }),
        ]),
      ],
      {
        app_items: [
          MenuItem('Settings…', { shortcut: 'cmd+,', on_selected: @{} }),
        ],
      },
    )

    var content = Center(Text('Saved ${self.saved} times'))

    return App({
      title: 'Editor',
      menu_bar: menus,
      windows: [Window('Untitled', content)],
    })
  }
}

run_app(Editor())
```

Menus are rebuilt with the `App`, so an item's `checked` and `enabled`
can come straight from your state, as Show Sidebar's tick does.

`MenuBar` takes `app_items`, a list of items for the application menu,
where Settings belongs. `MenuItem.separator()` is a line between groups
of items.

| `MenuItem` option | Type | Default | Description |
| --- | --- | --- | --- |
| `on_selected` | `function` | none | Called with no arguments when the user chooses the item. |
| `shortcut` | `string` | none | The item's key equivalent, such as `cmd+shift+n`. See [key bindings](drawing-and-input.md#shortcuts). |
| `enabled` | `bool` | `true` | Set to `false` to grey the item out. |
| `checked` | `bool` | `false` | Shows a tick beside the item. |
| `items` | `list` | none | Makes the item a submenu of these items. |
| `role` | `string` | none | Makes the item do one of the system's standard actions. See below. |

### Standard roles

An item with a `role` does what the same item does in every Mac
application, with no code of yours. A Copy item with `role: 'copy'`
copies from whichever text field has focus, for example.

The roles are `about`, `hide`, `hide_others`, `show_all`, `quit`,
`undo`, `redo`, `cut`, `copy`, `paste`, `delete`, `select_all`,
`minimize`, `zoom`, `close`, `fullscreen` and `bring_all_to_front`.

```zuri,ignore
Menu('Edit', [
  MenuItem('Undo', { role: 'undo', shortcut: 'cmd+z' }),
  MenuItem('Redo', { role: 'redo', shortcut: 'cmd+shift+z' }),
  MenuItem.separator(),
  MenuItem('Cut', { role: 'cut', shortcut: 'cmd+x' }),
  MenuItem('Copy', { role: 'copy', shortcut: 'cmd+c' }),
  MenuItem('Paste', { role: 'paste', shortcut: 'cmd+v' }),
])
```

To use the standard action when there is one, and do something of your
own when there isn't, give the item an `on_selected` that calls
`perform_role()`. See [Menus](../apps-windows-and-menus.md#menus).

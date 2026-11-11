# Applications, windows and menus

An application is a widget too. `App` holds its windows and its menu
bar, and you change either by building it differently, the same way you
change anything else. This page explains how that works in practice.
Every option of `App`, `Window` and `MenuItem` is listed in
[The application](components/app-and-windows.md).

## The application

```zuri
import zug { * }

run_app(App({
  title: 'Notes',
  windows: [
    Window('Notes', Center(Text('No notes yet')), { size: Size(640, 420) }),
  ],
}))
```

`run_app(app)` starts the application and returns when it quits.
`quit_app()` quits from code, asking the `App`'s `on_quit_request`
first, and `current_runtime()` returns the running application's
runtime, or `nil` when none is running.

When the `App` is built by a `StatefulWidget`, every option of it can
come from state: which windows are open, what the menus say, and
whether it shows itself light or dark.

## Opening and closing windows

The windows listed in the `App` are the windows that are open. To open
one, add it to the list and rebuild. To close one, take it out:

```zuri
import zug { * }

class Studio < StatefulWidget {
  create_state() {
    return StudioState()
  }
}

class StudioState < State {
  var inspector = false

  toggle() {
    self.set_state(@{
      self.inspector = !self.inspector
    })
  }

  build(context) {
    var main = Center(Button('Toggle the Inspector', { on_pressed: @{
      self.toggle()
    } }))

    var windows = [Window('Studio', main, { size: Size(480, 300) })]

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

When the user closes a window, its `on_close_request` is asked first,
and returning `false` keeps it open. That's the place to ask about
unsaved changes. Then `on_closed` is called, and you should take the
window out of your list there. A closed window left in the list stays
closed, but its widgets and their state stay alive for nothing.

Closing the last window quits the application, unless the `App` has
`quit_on_last_window_closed: false`.

## A window or a dialog

A second window suits anything the user keeps open beside their work,
such as an inspector, a settings window or a find panel.

It doesn't suit a question that needs an answer before the user carries
on. A window doesn't stop them going back to the main one, and it can't
hand you an answer. For that, use
[`show_dialog()`](services.md#dialogs-of-your-own), which shows your
widgets as a sheet on the window and gives you a promise of the result.
For a simple question with a few buttons,
[`show_alert()`](services.md#dialogs) is quicker still.

## Menus

On macOS the menu bar belongs to the application. Give the `App` a
`MenuBar`, and zug adds the standard application menu, with About,
Hide, Quit and the rest, and Edit and Window menus if you don't make
your own:

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
    MenuItem('Show Sidebar', {
      shortcut: 'cmd+alt+s',
      checked: true,
      on_selected: @{},
    }),
  ]),
], {
  app_items: [
    MenuItem('Settings…', { shortcut: 'cmd+,', on_selected: @{} }),
  ],
})

run_app(App({
  title: 'Notes',
  menu_bar: menus,
  windows: [Window('Notes', Center(Text('Hello')))],
}))
```

Menus are rebuilt with the `App`, so an item's `checked` and `enabled`
can come straight from your state. `app_items` go in the application
menu, which is where Settings belongs on a Mac.

An item with a `role` does one of the system's standard actions, the
way it would in any Mac application. A Copy item with `role: 'copy'`
copies from whatever text field has focus, with no code of yours. The
[component reference](components/app-and-windows.md#standard-roles)
lists the roles.

When you want an item to do the standard action if there is one, and
something of your own otherwise, call `perform_role()`. It returns
whether anything carried the action out:

```zuri,ignore
MenuItem('Copy', {
  shortcut: 'cmd+c',
  on_selected: @{
    if !perform_role('copy') {
      Clipboard.write_text(self.result)
    }
  },
})
```

A calculator does this: Copy copies the selected text when a field has
some, and the result on the display when nothing does.

## The keyboard

Keys go to the focused control first. A text field takes what's typed
into it, and a button takes Space. Keys the control doesn't use go to
the [`Shortcuts`](components/drawing-and-input.md#shortcuts) and
[`KeyListener`](components/drawing-and-input.md#keylistener) widgets
around it in the window, the nearest first, until one of them handles
the key.

Text the user can select but not edit, such as a `Text` with
`selectable: true`, keeps only the keys that move its selection or the
focus: the arrows, `home`, `end`, `page_up`, `page_down`, `tab`, and
anything with `cmd` or `ctrl`. A digit or Return typed while it's
selected still reaches your shortcuts.

Escape goes to the focused widget first. Every widget that can take
keyboard focus takes `on_escape`, from buttons and checkboxes to tables
and text fields, and while one has focus, its `on_escape` gets Escape
before your shortcuts or a Cancel button do. A focused widget without
one passes Escape on as usual.

```zuri
import zug { * }

var keypad = Shortcuts({
  'escape': @{
    echo 'clear'
  },
  'cmd+backspace': @{
    echo 'clear everything'
  },
  '+': @{
    echo 'plus'
  },
}, KeyListener(Center(Text('Type a digit')), {
  on_key: @(event) {
    if event.characters.is_number() {
      echo 'digit ${event.characters}'
      return true
    }

    return false
  },
}))

run_app(App({
  windows: [Window('Keypad', keypad, { size: Size(240, 120) })],
}))
```

Keys typed into a text field never reach shortcuts. When you want a
window's keys to go to its shortcuts again, such as after the user
presses Return in a field, call `clear_focus(context)` with the context
of a widget inside that window. The context of a widget above the
windows, such as the one that builds your `App`, belongs to no window,
so passing it raises a `ValueError`.

[Shortcuts](components/drawing-and-input.md#shortcuts) explains how to
write a key binding, such as `cmd+shift+k`, and which keys have names.

# Drawing and input

[Components](index.md) › Drawing and input

## Canvas

`Canvas(paint, options)` is a surface you draw on yourself. zug calls
your `paint` function with a `Painter` and the canvas's size whenever
the canvas needs drawing, and you draw shapes, lines, text and images
with it.

```zuri
import zug { * }

var sales = [12, 19, 8, 15, 22, 17]

var chart = Canvas(@(painter, size) {
  var gap = 8
  var width = (size.width - gap * (sales.length() - 1)) / sales.length()
  var highest = 25

  iter var i = 0; i < sales.length(); i++ {
    var height = sales[i] / highest * (size.height - 20)
    var bar = Rect(i * (width + gap), size.height - height, width, height)
    painter.fill_rounded_rect(bar, 3, Color.system('accent'))
  }

  painter.text('Sales by month', Point(0, 0), TextStyle({ size: 11 }))
}, { repaint: sales, height: 160 })

run_app(
  App({
    windows: [
      Window('Sales', Padding(EdgeInsets.all(16), chart), {
        size: Size(360, 200),
      }),
    ],
  }),
)
```

Coordinates start at the canvas's top left, and y grows downwards. A
canvas fills the space it's given unless `width` or `height` fix it.

A canvas is drawn when it first appears, when its size changes, and
when `repaint` changes. Pass the data the drawing comes from as
`repaint`, and the canvas redraws exactly when that data does. Without
`repaint`, it's drawn again after every build that reaches it.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `repaint` | any | none | The data the drawing depends on. The canvas redraws when it changes. |
| `width` | `number` | none | Fixes the canvas's width. |
| `height` | `number` | none | Fixes the canvas's height. |

[Drawing](../drawing-and-pointer.md) covers everything the painter can
draw: rectangles, ovals, lines, paths, text and images, and how to
move, scale, rotate and clip what you draw.

## PointerArea

`PointerArea(child, options)` hears what the mouse and trackpad do over
its child: clicks, double clicks, right clicks, drags, scrolling, and
the pointer coming and going. Together with a `Canvas`, it's all you
need for a control of your own.

```zuri
import zug { * }

class Sketch < StatefulWidget {
  create_state() {
    return SketchState()
  }
}

class SketchState < State {
  var dots = []

  build(context) {
    var canvas = Canvas(@(painter, size) {
      painter.fill_rect(Rect(0, 0, size.width, size.height), Color.WHITE)

      for dot in self.dots {
        painter.fill_oval(Rect(dot.x - 2, dot.y - 2, 4, 4), Color.BLACK)
      }
    }, { repaint: self.dots.length() })

    return PointerArea(canvas, {
      cursor: Cursor.CROSSHAIR,
      on_drag: @(event) {
        self.set_state(@{
          self.dots.append(event.position)
        })
      },
      on_double_tap: @(event) {
        self.set_state(@{
          self.dots = []
        })
      },
    })
  }
}

run_app(
  App({
    windows: [
      Window('Sketch', Sketch(), { size: Size(400, 300) }),
    ],
  }),
)
```

Drag to draw, and double click to start again.

Every handler except `on_enter` and `on_exit` is called with a
`PointerEvent`:

| Field | Holds |
| --- | --- |
| `position` | Where it happened, as a `Point` from the area's top left. |
| `delta` | For a drag or a scroll, how far it moved since the last event. |
| `total` | For a drag, how far it has moved since it started. |
| `clicks` | How many clicks came in quick succession. |
| `modifiers` | The modifier keys held, any of `cmd`, `shift`, `alt` and `ctrl`. `has_modifier('shift')` asks about one. |

| Option | Type | Description |
| --- | --- | --- |
| `on_tap` | `function` | A click. |
| `on_double_tap` | `function` | A double click. |
| `on_secondary_tap` | `function` | A right click, or a click with Control held. |
| `on_drag_start` | `function` | The pointer moved with the button held down. |
| `on_drag` | `function` | Each movement of a drag. |
| `on_drag_end` | `function` | The button was let go after a drag. |
| `on_hover` | `function` | The pointer moved over the area with no button held. |
| `on_scroll` | `function` | A scroll wheel or two finger swipe over the area. |
| `on_enter` | `function` | The pointer came into the area. Called with no arguments. |
| `on_exit` | `function` | The pointer left the area. Called with no arguments. |
| `cursor` | `string` | The pointer's shape over the area. See below. |
| `menu` | `list` | `MenuItem`s to show on a right click. See [ContextMenu](#contextmenu). |

The cursors are `Cursor.ARROW`, `POINTING_HAND`, `TEXT`, `CROSSHAIR`,
`OPEN_HAND`, `CLOSED_HAND`, `RESIZE_LEFT_RIGHT`, `RESIZE_UP_DOWN` and
`NOT_ALLOWED`.

Native controls inside a pointer area handle their own clicks. The area
hears the clicks that land on the rest of it.

## ContextMenu

`ContextMenu(child, items)` opens a menu when the user right clicks its
child. The items are the same [`MenuItem`s](app-and-windows.md#menubar-menu-and-menuitem)
the menu bar uses.

```zuri
import zug { * }

var file = ContextMenu(
  Row(
    [Icon('doc.richtext'), Text('Quarterly report.pdf')],
    { spacing: 6, shrink_wrap: true },
  ),
  [
    MenuItem('Open', {
      on_selected: @{
        echo 'open'
      },
    }),
    MenuItem('Rename…', {
      on_selected: @{
        echo 'rename'
      },
    }),
    MenuItem.separator(),
    MenuItem('Move to Trash', {
      on_selected: @{
        echo 'trash'
      },
    }),
  ],
)

run_app(
  App({
    windows: [
      Window('Files', Center(file), { size: Size(300, 120) }),
    ],
  }),
)
```

A `PointerArea` that already handles the mouse can take the same list
as its `menu` option instead.

## Shortcuts

`Shortcuts(bindings, child)` calls a function when a key is pressed
anywhere inside its child. `bindings` maps each key to its function:

```zuri
import zug { * }

class Viewer < StatefulWidget {
  create_state() {
    return ViewerState()
  }
}

class ViewerState < State {
  var page = 1

  turn(by) {
    self.set_state(@{
      self.page = (self.page + by).max(1)
    })
  }

  build(context) {
    return Shortcuts({
      'right': @{
        self.turn(1)
      },
      'left': @{
        self.turn(-1)
      },
      'cmd+up': @{
        self.set_state(@{
          self.page = 1
        })
      },
    }, Center(Text('Page ${self.page}. Use the arrow keys to turn.')))
  }
}

run_app(
  App({
    windows: [
      Window('Reader', Viewer(), { size: Size(320, 140) }),
    ],
  }),
)
```

A binding is a key with any modifiers before it, joined with `+`, such
as `cmd+shift+k`, `alt+left` or `f5`. The modifiers are `cmd`, `shift`,
`alt` and `ctrl`, and the named keys are `enter`, `escape`,
`backspace`, `delete`, `tab`, `space`, `left`, `right`, `up`, `down`,
`home`, `end`, `page_up`, `page_down` and `f1` to `f12`.

A single character with no modifiers, such as `+` or `*`, matches
however the keyboard types it. You don't need to know that `+` is
Shift and `=` on one layout and a key of its own on another.

Keys typed into a text field stay in the field and never reach
shortcuts. [The keyboard](../apps-windows-and-menus.md#the-keyboard)
explains the order in which keys travel through a window. For a
shortcut that belongs to a menu command, give the `MenuItem` the
shortcut instead, so people can find it in the menu.

## KeyListener

`KeyListener(child, { on_key })` sees every key that reaches it. Use it
when you need the key itself rather than a few fixed bindings, such as
a game or a keypad that takes any digit. `on_key` gets a `KeyEvent` and
returns `true` for a key it used, so the key goes no further:

```zuri
import zug { * }

class Keypad < StatefulWidget {
  create_state() {
    return KeypadState()
  }
}

class KeypadState < State {
  var typed = ''

  build(context) {
    var label = self.typed == '' ? 'Type a number' : self.typed

    return KeyListener(Center(Text(label)), {
      on_key: @(event) {
        if event.characters.is_number() {
          self.set_state(@{
            self.typed += event.characters
          })
          return true
        }

        if event.matches('backspace') and self.typed != '' {
          self.set_state(@{
            self.typed = self.typed[0, self.typed.length() - 1]
          })
          return true
        }

        return false
      },
    })
  }
}

run_app(
  App({
    windows: [
      Window('Keypad', Keypad(), { size: Size(260, 120) }),
    ],
  }),
)
```

| `KeyEvent` field | Holds |
| --- | --- |
| `key` | The key: a named key such as `enter` or `left`, or the character the key types without modifiers. |
| `characters` | What the key types with the modifiers held, such as `+` for Shift and `=` on a US keyboard. |
| `modifiers` | The modifier keys held, any of `cmd`, `shift`, `alt` and `ctrl`. `has_modifier('cmd')` asks about one. |
| `repeat` | Whether the key is repeating because it's held down. |

`event.matches(binding)` says whether the event is the key a binding
names, using the same rules as `Shortcuts`.

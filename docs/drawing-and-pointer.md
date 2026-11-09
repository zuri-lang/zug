# Drawing and pointer input

When none of the controls fits, draw it yourself with `Canvas`, and
handle the mouse with `PointerArea`. Together they're enough for charts,
editors, games and custom controls.

## Canvas

`Canvas(paint, options)` calls your `paint` function with a `Painter` and
the canvas's size whenever it needs drawing:

```zuri
import math
import zug { * }

var values = [3, 7, 4, 9, 6, 11, 8]

var chart = Canvas(@(painter, size) {
  painter.fill_rounded_rect(Rect(0, 0, size.width, size.height), 8, Color.system('control_background'))
  var path = Path()

  iter var i = 0; i < values.length(); i++ {
    var x = 12 + i * (size.width - 24) / (values.length() - 1)
    var y = size.height - 12 - values[i] / 12 * (size.height - 24)
    if i == 0 {
      path.move_to(x, y)
    } else {
      path.line_to(x, y)
    }

    painter.fill_oval(Rect(x - 3, y - 3, 6, 6), Color.system('accent'))
  }

  painter.stroke_path(path, Paint({ color: Color.system('accent'), width: 2, join: LineJoin.ROUND }))
  painter.text('Sales', Point(12, 8), TextStyle({ size: 12, weight: FontWeight.SEMIBOLD }))
}, { repaint: values, height: 160 })
```

Coordinates start at the canvas's top left, with y growing downwards. A
canvas fills the space it's given unless `width` or `height` fix it.

It's drawn again when it first appears, when its size changes, and when
`repaint` changes. Pass the data the drawing comes from as `repaint`, so
the canvas redraws exactly when that data does. Without `repaint` it's
drawn again after every build that reaches it.

### The painter

| Method | Draws |
| --- | --- |
| `fill_rect(rect, color)`, `stroke_rect(rect, paint)` | rectangles |
| `fill_rounded_rect(rect, radius, color)`, `stroke_rounded_rect(rect, radius, paint)` | rounded rectangles |
| `fill_oval(rect, color)`, `stroke_oval(rect, paint)` | ellipses filling `rect` |
| `line(from, to, paint)` | a line between two `Point`s |
| `fill_path(path, color)`, `stroke_path(path, paint)` | any shape |
| `text(text, at, style, max_width)` | text with its top left at `at`, wrapped at `max_width` when you give one |
| `image(source, rect)` | an `ImageSource` scaled into `rect` |

`measure_text(text, style, max_width)` returns the `Size` text would
take, wrapped the same way, which is how you work out where the next
paragraph goes. `size()` is the canvas's size. `translate(dx, dy)`,
`scale(sx, sy)` and `rotate(radians)` move what follows,
`clip_rect(rect)` limits it to a rectangle, and `save()` and `restore()`
bracket those changes so they don't leak into the rest of the drawing.

A `Paint` says how lines are stroked: `color`, `width` (1), `cap`
(`LineCap.BUTT`, `ROUND` or `SQUARE`), `join` (`LineJoin.MITER`, `ROUND`
or `BEVEL`) and `dash`, a list of lengths such as `[4, 2]` that
alternate between drawn and gaps.

A `Path` is built from `move_to(x, y)`, `line_to(x, y)`,
`quad_to(cx, cy, x, y)`, `cubic_to(c1x, c1y, c2x, c2y, x, y)`,
`arc(center, radius, start, end)` with angles in radians, the shapes
`add_rect(rect)`, `add_oval(rect)` and `add_rounded_rect(rect, radius)`,
and `close()`. Each returns the path, so the calls chain.

## Pointer input

`PointerArea(child, options)` hears what the mouse does over its child:

```zuri
import zug { * }

class Sketch < StatefulWidget {
  create_state() {
    return SketchState()
  }
}

class SketchState < State {
  var points = []

  build(context) {
    return PointerArea(Canvas(@(painter, size) {
      painter.fill_rect(Rect(0, 0, size.width, size.height), Color.WHITE)

      for point in self.points {
        painter.fill_oval(Rect(point.x - 2, point.y - 2, 4, 4), Color.BLACK)
      }
    }, { repaint: self.points.length() }), {
      cursor: Cursor.CROSSHAIR,
      on_drag: @(event) {
        self.set_state(@{
          self.points.append(event.position)
        })
      },
      on_double_tap: @(event) {
        self.set_state(@{
          self.points = []
        })
      },
    })
  }
}

run_app(App({ windows: [Window('Sketch', Sketch(), { size: Size(400, 300) })] }))
```

The handlers are `on_tap`, `on_double_tap`, `on_secondary_tap` (a right
or control click), `on_drag_start`, `on_drag`, `on_drag_end`, `on_hover`,
`on_scroll`, and `on_enter` and `on_exit` for the pointer arriving and
leaving. All but the last two receive a `PointerEvent`:

| Field | What it holds |
| --- | --- |
| `position` | where it happened, a `Point` from the area's top left |
| `delta` | for a drag or a scroll, how far it moved since the last event |
| `total` | for a drag, how far it has moved since it started |
| `clicks` | the number of clicks in quick succession |
| `modifiers` | the modifier keys held, any of `cmd`, `shift`, `alt` and `ctrl`; `has_modifier('shift')` asks about one |

Native controls inside a pointer area handle their own clicks; the area
hears the clicks that land on the rest of it.

`cursor` sets the pointer's shape over the area: `Cursor.ARROW`,
`POINTING_HAND`, `TEXT`, `CROSSHAIR`, `OPEN_HAND`, `CLOSED_HAND`,
`RESIZE_LEFT_RIGHT`, `RESIZE_UP_DOWN` or `NOT_ALLOWED`.

## Context menus

A secondary click can open a menu. Give `PointerArea` a `menu` of
`MenuItem`s, or wrap anything in `ContextMenu`:

```zuri
import zug { * }

var row = ContextMenu(Text('report.pdf'), [
  MenuItem('Open', { on_selected: @{} }),
  MenuItem('Rename…', { on_selected: @{} }),
  MenuItem.separator(),
  MenuItem('Move to Trash', { on_selected: @{} }),
])
```

The items are the same `MenuItem`s the menu bar uses; see
[Menus](apps-windows-and-menus.md#menus).

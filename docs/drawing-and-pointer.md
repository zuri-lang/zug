# Drawing and pointer input

When none of the built in controls fits, draw it yourself with a
[`Canvas`](components/drawing-and-input.md#canvas), and handle the mouse
with a [`PointerArea`](components/drawing-and-input.md#pointerarea).
Together they're enough for charts, editors, games and controls of your
own. This page covers the painter you draw with, and walks through
building a small control.

## Drawing on a canvas

`Canvas(paint, options)` calls `paint` with a `Painter` and the
canvas's size whenever it needs drawing:

```zuri
import zug { * }

var values = [3, 7, 4, 9, 6, 11, 8]

var chart = Canvas(@(painter, size) {
  var area = Rect(0, 0, size.width, size.height)
  painter.fill_rounded_rect(area, 8, Color.system('control_background'))

  var path = Path()
  var step = (size.width - 24) / (values.length() - 1)

  iter var i = 0; i < values.length(); i++ {
    var x = 12 + i * step
    var y = size.height - 12 - values[i] / 12 * (size.height - 36)

    if i == 0 {
      path.move_to(x, y)
    } else {
      path.line_to(x, y)
    }

    painter.fill_oval(Rect(x - 3, y - 3, 6, 6), Color.system('accent'))
  }

  painter.stroke_path(path, Paint({
    color: Color.system('accent'),
    width: 2,
    join: LineJoin.ROUND,
  }))
  painter.text('Sales', Point(12, 8), TextStyle({
    size: 12,
    weight: FontWeight.SEMIBOLD,
  }))
}, { repaint: values, height: 160 })

run_app(App({
  windows: [
    Window('Sales', Padding(EdgeInsets.all(16), chart), {
      size: Size(360, 200),
    }),
  ],
}))
```

Coordinates start at the canvas's top left, and y grows downwards.

The canvas is drawn when it first appears, when its size changes, and
when `repaint` changes. Pass the data the drawing comes from as
`repaint`, as above, and it redraws exactly when that data does.

## The painter

| Method | Draws |
| --- | --- |
| `fill_rect(rect, color)`, `stroke_rect(rect, paint)` | Rectangles. |
| `fill_rounded_rect(rect, radius, color)`, `stroke_rounded_rect(rect, radius, paint)` | Rectangles with rounded corners. |
| `fill_oval(rect, color)`, `stroke_oval(rect, paint)` | Ellipses that fill `rect`. A square `rect` gives a circle. |
| `line(from, to, paint)` | A line between two `Point`s. |
| `fill_path(path, color)`, `stroke_path(path, paint)` | Any shape, built as a `Path`. |
| `text(text, at, style, max_width)` | Text with its top left at `at`, wrapped at `max_width` when you give one. |
| `image(source, rect)` | An `ImageSource` scaled into `rect`. |

`measure_text(text, style, max_width)` returns the `Size` the text
would take, wrapped the same way, which is how you work out where the
next line or paragraph goes. `size()` is the size of the canvas.

`translate(dx, dy)`, `scale(sx, sy)` and `rotate(radians)` move,
stretch and turn everything drawn after them, and `clip_rect(rect)`
keeps what follows inside a rectangle. Put `save()` before those
changes and `restore()` after the drawing they're for, so they don't
leak into the rest:

```zuri,ignore
painter.save()
painter.translate(size.width / 2, size.height / 2)
painter.rotate(math.pi / 4)
painter.fill_rect(Rect(-20, -20, 40, 40), Color.system('orange'))
painter.restore()
```

### Paint

A `Paint` says how lines are stroked:

| Option | Default | Description |
| --- | --- | --- |
| `color` | the label colour | The colour of the line. |
| `width` | `1` | How thick the line is, in points. |
| `cap` | `LineCap.BUTT` | How the ends look: `BUTT`, `ROUND` or `SQUARE`. |
| `join` | `LineJoin.MITER` | How corners look: `MITER`, `ROUND` or `BEVEL`. |
| `dash` | none | A list of lengths, such as `[4, 2]`, that alternate between line and gap. |

### Path

A `Path` is a shape built from lines and curves:

| Method | Adds |
| --- | --- |
| `move_to(x, y)` | Starts a new piece of the shape at a point. |
| `line_to(x, y)` | A straight line to a point. |
| `quad_to(cx, cy, x, y)` | A curve to a point, bent towards one control point. |
| `cubic_to(c1x, c1y, c2x, c2y, x, y)` | A curve to a point, shaped by two control points. |
| `arc(center, radius, start, end)` | Part of a circle, with angles in radians. |
| `add_rect(rect)`, `add_oval(rect)`, `add_rounded_rect(rect, radius)` | A whole shape. |
| `close()` | A line back to where the piece started. |

Each method returns the path, so you can chain them:

```zuri,ignore
var triangle = Path().move_to(0, 20).line_to(10, 0).line_to(20, 20).close()
```

## Building a control

A control of your own is a `Canvas` that draws the current state, in a
`PointerArea` that changes it. This one is a row of colour swatches.
Clicking a swatch picks it, and the swatch under the pointer gets a
ring:

```zuri
import zug { * }

var COLORS = ['#ff453a', '#ff9f0a', '#ffd60a', '#32d74b', '#0a84ff', '#bf5af2']
var SIZE = 28
var GAP = 8

class Swatches < StatefulWidget {
  var selected
  var on_changed

  @new(selected: number, on_changed: function) {
    self.selected = selected
    self.on_changed = on_changed
  }

  create_state() {
    return SwatchesState()
  }
}

class SwatchesState < State {
  var hovered

  index_at(point) {
    var index = (point.x / (SIZE + GAP)).floor()
    return index >= 0 and index < COLORS.length() ? index : nil
  }

  build(context) {
    var selected = self.widget.selected
    var hovered = self.hovered

    var canvas = Canvas(@(painter, size) {
      iter var i = 0; i < COLORS.length(); i++ {
        var spot = Rect(i * (SIZE + GAP), 0, SIZE, SIZE)
        painter.fill_oval(spot, Color.hex(COLORS[i]))

        if i == selected or i == hovered {
          painter.stroke_oval(
            Rect(spot.x - 3, spot.y - 3, SIZE + 6, SIZE + 6), Paint({
              color: Color.system(i == selected ? 'label' : 'tertiary_label'),
              width: 2,
            }),
          )
        }
      }
    }, {
      repaint: [selected, hovered],
      width: COLORS.length() * (SIZE + GAP),
      height: SIZE,
    })

    return PointerArea(canvas, {
      cursor: Cursor.POINTING_HAND,
      on_hover: @(event) {
        self.set_state(@{
          self.hovered = self.index_at(event.position)
        })
      },
      on_exit: @{
        self.set_state(@{
          self.hovered = nil
        })
      },
      on_tap: @(event) {
        var index = self.index_at(event.position)

        if index != nil {
          self.widget.on_changed(index)
        }
      },
    })
  }
}

class Picker < StatefulWidget {
  create_state() {
    return PickerState()
  }
}

class PickerState < State {
  var color = 4

  build(context) {
    return Center(Swatches(self.color, @(index) {
      self.set_state(@{
        self.color = index
      })
    }))
  }
}

run_app(App({
  windows: [Window('Label colour', Picker(), { size: Size(300, 100) })],
}))
```

A few things make it behave like a real control:

- **It follows the same rule as the built in controls.** `Swatches`
  shows the `selected` it's given and reports clicks through
  `on_changed`. The `Picker` above it owns the choice.
- **What it only uses for drawing lives in its own state.** The
  swatch under the pointer matters to nobody else, so `SwatchesState`
  keeps it.
- **`repaint` lists everything the drawing depends on,** so the canvas
  redraws when the selection or the hovered swatch changes, and at no
  other time.
- **It has a fixed size,** so it sits in a row or a column like any
  other control instead of filling the space.

Give a control like this an `accessibility_label` through its
`PointerArea` or `Canvas`, so VoiceOver can say what it is.

[Drawing and input](components/drawing-and-input.md#pointerarea) lists
every pointer handler and what a `PointerEvent` holds.

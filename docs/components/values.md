# Numbers, dates and colours

[Components](index.md) › Numbers, dates and colours

## Slider

`Slider(value, options)` picks a number by dragging a knob along a
track. It runs from `min` to `max`, 0 to 1 unless you say otherwise.

```zuri
import zug { * }

class Volume < StatefulWidget {
  create_state() {
    return VolumeState()
  }
}

class VolumeState < State {
  var level = 0.6
  var saved = 0.6

  build(context) {
    return Padding(EdgeInsets.all(20), Column([
      Row([
        Icon('speaker.fill'),
        Expanded(Slider(self.level, {
          accessibility_label: 'Volume',
          on_changed: @(value) {
            self.set_state(@{
              self.level = value
            })
          },
          on_change_end: @(value) {
            self.set_state(@{
              self.saved = value
            })
          },
        })),
        Icon('speaker.wave.3.fill'),
      ], { spacing: 8 }),
      Text('Saved at ${(self.saved * 100).round()}%'),
    ], { spacing: 12 }))
  }
}

run_app(App({
  windows: [Window('Sound', Volume(), { size: Size(320, 110) })],
}))
```

`on_changed` is called again and again while the user drags, so keep it
quick. When the work is slow, such as saving a setting or fetching new
data, do it in `on_change_end`, which is called once when the user lets
go.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `min` | `number` | `0` | The value at the start of the track. |
| `max` | `number` | `1` | The value at the end of the track. |
| `divisions` | `number` | `0` | Snaps the knob to that many equal steps. `0` lets it move freely. |
| `vertical` | `bool` | `false` | Stands the slider up, with `min` at the bottom. |
| `on_changed` | `function` | none | Called with the value while the user drags. |
| `on_change_end` | `function` | none | Called with the final value when the user lets go. |
| `enabled` | `bool` | `true` | Set to `false` to disable the slider. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

In a row, a slider is 160 points wide unless you give it an `Expanded`,
as above, or a `Box` with a width. In a column that stretches its
children, it's as wide as the column.

## Stepper

`Stepper(value, options)` is a pair of arrows that move a number up or
down by a step. It shows no number itself, so put it beside a label
that does:

```zuri
import zug { * }

class Copies < StatefulWidget {
  create_state() {
    return CopiesState()
  }
}

class CopiesState < State {
  var copies = 1

  build(context) {
    return Center(Row([
      Text('Copies:'),
      Text('${self.copies}', {
        style: TextStyle({ monospaced_digits: true }),
      }),
      Stepper(self.copies, {
        min: 1,
        max: 99,
        accessibility_label: 'Copies',
        on_changed: @(value) {
          self.set_state(@{
            self.copies = value
          })
        },
      }),
    ], { spacing: 6, shrink_wrap: true }))
  }
}

run_app(App({
  windows: [Window('Print', Copies(), { size: Size(220, 80) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `min` | `number` | `0` | The lowest value. |
| `max` | `number` | `100` | The highest value. |
| `step` | `number` | `1` | How far each click moves the value. |
| `wraps` | `bool` | `false` | Goes round from `max` to `min`, and back, instead of stopping. |
| `on_changed` | `function` | none | Called with the new value. |
| `enabled` | `bool` | `true` | Set to `false` to disable the stepper. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

## DatePicker

`DatePicker(value, options)` picks a date, a time or both. It takes and
reports a `date.Date` from Zuri's `date` module.

```zuri
import date
import zug { * }

class Booking < StatefulWidget {
  create_state() {
    return BookingState()
  }
}

class BookingState < State {
  var arrival = date(2026, 12, 18)

  build(context) {
    var today = date(2026, 11, 11)

    return Padding(EdgeInsets.all(20), Column([
      Row([
        Text('Arriving'),
        DatePicker(self.arrival, {
          min: today,
          max: today.add_days(365),
          on_changed: @(day) {
            self.set_state(@{
              self.arrival = day
            })
          },
        }),
      ], { spacing: 8 }),
      Text(self.arrival.format('l, j F Y')),
    ], { spacing: 12, align_items: AlignItems.START }))
  }
}

run_app(App({
  windows: [Window('Booking', Booking(), { size: Size(300, 110) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | `string` | `DateMode.DATE` | What the user picks: `DATE`, `TIME` or `DATE_TIME`. |
| `style` | `string` | `DateStyle.FIELD` | `FIELD` is a text field with arrows, `COMPACT` a small field that opens a calendar when clicked, and `GRAPHICAL` a calendar, or a clock for times. |
| `min` | `date.Date` | none | The earliest date the user can pick. |
| `max` | `date.Date` | none | The latest date the user can pick. |
| `on_changed` | `function` | none | Called with the new `date.Date`. |
| `enabled` | `bool` | `true` | Set to `false` to disable the picker. |

## ColorWell

`ColorWell(color, options)` is a swatch of a colour. Clicking it opens
the system's colour panel, and every colour the user picks there is
reported through `on_changed` as a `Color`.

```zuri
import zug { * }

class Highlighter < StatefulWidget {
  create_state() {
    return HighlighterState()
  }
}

class HighlighterState < State {
  var color = Color.hex('#ffd60a')

  build(context) {
    return Padding(EdgeInsets.all(20), Row([
      Text('Highlight colour'),
      ColorWell(self.color, {
        on_changed: @(color) {
          self.set_state(@{
            self.color = color
          })
        },
      }),
      Box({
        padding: EdgeInsets.symmetric({ horizontal: 6, vertical: 2 }),
        color: self.color,
        radius: 4,
        child: Text('Sample', {
          style: TextStyle({ color: Color.BLACK }),
        }),
      }),
    ], { spacing: 10 }))
  }
}

run_app(App({
  windows: [Window('Colours', Highlighter(), { size: Size(320, 80) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `on_changed` | `function` | none | Called with each `Color` the user picks. |
| `enabled` | `bool` | `true` | Set to `false` to disable the well. |

A colour well belongs in a form, next to the setting it changes. To ask
for a colour from a menu command or a toolbar button instead, use
[`pick_color()`](../services.md#fonts-and-colours).

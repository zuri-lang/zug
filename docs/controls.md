# Controls

Every control on this page is the platform's own: on macOS a `Button` is
an `NSButton`, a `Slider` an `NSSlider`, and so on. You get their look,
keyboard handling, accessibility and dark mode for free. Text fields
have [a page of their own](text-input.md).

## How controls take values

Controls follow one pattern, described in
[Getting started](getting-started.md#controls-show-what-you-give-them):
you pass the value to show, and `on_changed` tells you what the user
chose. Store it and rebuild, and the control shows it; ignore it, and the
control goes back. A control with nothing to call is disabled, and
`enabled: false` disables one that has.

Every control also takes a few common options:

| Option | What it does |
| --- | --- |
| `tooltip` | Text shown when the pointer rests on it. |
| `hidden` | Hides the view but keeps its space. |
| `accessibility_label` | What VoiceOver reads, when the visible text isn't enough. |
| `accessibility_identifier` | A name for UI tests and for `WidgetTester` to find it by. |
| `key` | See [Keys](widgets-and-state.md#keys). |
| `on_escape` | Called when Escape is pressed while it has keyboard focus. |

Many controls take `size`, one of `ControlSize.MINI`, `SMALL`,
`REGULAR` (the default) and `LARGE`.

## Text

```zuri
import zug { * }

var heading = Text('Quarterly report', {
  style: TextStyle({ size: 20, weight: FontWeight.SEMIBOLD }),
})
var summary = Text('Revenue grew in every region but one, which we will come back to.', {
  max_lines: 2,
  overflow: TextOverflow.ELLIPSIS,
  selectable: true,
})
```

Text wraps to the width it's given. `max_lines` limits it, `overflow`
says what happens to the rest (`TextOverflow.CLIP` or `ELLIPSIS`),
`align` sets `TextAlign.START`, `CENTER`, `END` or `JUSTIFY`, and
`selectable: true` lets the user select and copy it. A `Text` takes its
default style from the nearest [`Style`](styling.md), and its own
`style` is merged over that.

## Buttons

```zuri
import zug { * }

var actions = Row([
  Button('Cancel', { role: ButtonRole.CANCEL, on_pressed: @{} }),
  Button('Save', { role: ButtonRole.PRIMARY, shortcut: 'cmd+s', on_pressed: @{} }),
], { spacing: 8 })
```

`role` gives a button its meaning as well as its look:
`ButtonRole.PRIMARY` is the default button that Return presses,
`CANCEL` is the one Escape presses, and `DESTRUCTIVE` marks an action
that deletes something. `shortcut` sets a key equivalent such as
`cmd+s`, and `tint` colours it.

`variant` picks the kind of button: `ButtonVariant.PUSH` (the default),
`FLEXIBLE` for a push button that can be taller, `SQUARE`, `TOOLBAR`,
`INLINE`, `CIRCULAR`, `BORDERLESS` and `HELP`.

A button becomes a toggle when you give it `selected`: it stays pressed
in while `selected` is true, and `on_selected_changed` reports clicks.

## Choices

```zuri
import zug { * }

var choices = Column([
  Checkbox(true, { label: 'Show line numbers', on_changed: @(value) {} }),
  Switch(false, { on_changed: @(value) {} }),
  RadioGroup({ deg: 'Degrees', rad: 'Radians' }, 'deg', {
    axis: Axis.HORIZONTAL,
    on_changed: @(unit) {},
  }),
  SegmentedControl(['Day', 'Week', 'Month'], 1, { on_changed: @(index) {} }),
  Dropdown([
    DropdownItem('asc', 'Oldest first'),
    DropdownItem('desc', 'Newest first'),
    DropdownItem.separator(),
    DropdownItem('none', 'Unsorted', { enabled: false }),
  ], 'desc', { on_changed: @(order) {} }),
], { spacing: 10, align_items: AlignItems.START })
```

- **`Checkbox(value, options)`** takes `true` or `false`. With
  `tristate: true` it can also show `nil`, the mixed state you see on a
  "select all" box when only some items are selected.
- **`Switch(value, options)`** is an on and off switch.
- **`RadioGroup(items, value, options)`** is a set of radio buttons.
  `items` is a dictionary of values to labels, or a list of strings that
  are both. `Radio(value, group_value, options)` is a single radio
  button, for laying a group out yourself.
- **`SegmentedControl(segments, selected, options)`** is a row of
  segments, each a string or a `Segment(label, { tooltip, enabled })`.
  Its `mode` decides what `selected` means: in `SegmentMode.ONE` (the
  default) it's an index, in `ANY` a list of indices, and in `MOMENTARY`
  nothing stays selected and `on_pressed` gets the index clicked.
- **`Dropdown(items, value, options)`** is a pop-up menu of choices.
  Items are strings, or `DropdownItem(value, label)` when the value and
  its label differ.

## Numbers and ranges

```zuri
import zug { * }

var volume = Row([
  Icon('speaker.fill'),
  Expanded(Slider(0.6, { on_changed: @(value) {} })),
  Stepper(3, { min: 1, max: 10, on_changed: @(value) {} }),
], { spacing: 8 })
```

- **`Slider(value, options)`** runs from `min` (0) to `max` (1).
  `divisions` makes it stop at that many steps, `vertical: true` stands
  it up, and besides `on_changed`, which comes while the user drags,
  `on_change_end` comes once when they let go.
- **`Stepper(value, options)`** steps by `step` (1) between `min` (0) and
  `max` (100), and `wraps` from one end round to the other.
- **`LevelIndicator(value, options)`** shows a level from `min` (0) to
  `max` (5) in a `style` of `LevelStyle.CONTINUOUS`, `DISCRETE`,
  `RELEVANCY` or `RATING`. `warning` and `critical` colour it past those
  levels, and with `on_changed` a rating can be clicked.

## Dates and colours

```zuri
import date
import zug { * }

var due = DatePicker(date.from_time(1700000000), {
  mode: DateMode.DATE,
  style: DateStyle.FIELD,
  on_changed: @(value) {},
})
var highlight = ColorWell(Color.hex('#ffcc00'), { on_changed: @(color) {} })
```

`DatePicker` takes and reports a `date.Date`. `mode` is
`DateMode.DATE`, `TIME` or `DATE_TIME`; `style` is `DateStyle.FIELD`,
`COMPACT` or `GRAPHICAL` (a calendar); `min` and `max` limit it.
`ColorWell` opens the system colour panel and reports a `Color`.

## Progress

- **`ProgressBar(options)`** shows `value` between `min` (0) and `max`
  (1). Leave `value` out for a bar that shows work without saying how
  much is left.
- **`Spinner(options)`** is the spinning indicator; `animating: false`
  stops it.

## Images, icons and grouping

```zuri
import zug { * }

var header = Row([
  Icon('folder.fill', { size: 16, color: Color.system('blue') }),
  Text('Projects'),
], { spacing: 6 })
var logo = Image(ImageSource.file('assets/logo.png'), { width: 64, height: 64 })
var group = GroupBox(Text('Inside'), { title: 'Display' })
```

- **`Image(source, options)`** shows an `ImageSource`: `file(path)`,
  `memory(bytes)` or, on macOS, `symbol(name)` for an SF Symbol. `fit` is
  `ImageFit.CONTAIN` (the default), `FILL`, `NONE` or `SCALE_DOWN`, and
  `tint` colours a template image. `rotation` turns it clockwise a
  quarter at a time, as 90, 180 or 270 degrees, or -90 the other way,
  and its size turns with it. `flip_horizontal` mirrors it left to right
  and `flip_vertical` top to bottom; flips happen before the turn.
- **`Icon(name, options)`** is an SF Symbol sized like text, with `size`,
  `weight` and `color`.
- **`GroupBox(child, { title })`** draws the standard titled box around
  related controls.
- **`Separator()`** is a thin line, horizontal unless you pass
  `{ axis: Axis.VERTICAL }`. In a `Row`, give a vertical one a height,
  such as `Box({ height: 20, child: Separator({ axis: Axis.VERTICAL }) })`,
  as a row only stretches its children with `AlignItems.STRETCH`.

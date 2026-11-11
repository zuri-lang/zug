# Choices

[Components](index.md) › Choices

These controls let the user pick from options you offer. Each one shows
the value you pass it and reports a new one through `on_changed`, as
described in [How controls take values](index.md#how-controls-take-values).

## Checkbox

`Checkbox(value, options)` is a box with a label that's either ticked
or not. Pass `true` or `false` as the value.

With `tristate: true` the value can also be `nil`, which shows a dash.
That's the mixed state you see on a box that stands for several others
when only some of them are ticked:

```zuri
import zug { * }

class Downloads < StatefulWidget {
  create_state() {
    return DownloadsState()
  }
}

class DownloadsState < State {
  var names = ['Documents', 'Photos', 'Music']
  var chosen = [true, false, true]

  all_value() {
    if self.chosen.every(@(on) => on) {
      return true
    }

    return self.chosen.some(@(on) => on) ? nil : false
  }

  build(context) {
    var boxes = [
      Checkbox(self.all_value(), {
        label: 'Everything',
        tristate: true,
        on_changed: @(value) {
          self.set_state(@{
            self.chosen = self.chosen.map(@(on) => value == true)
          })
        },
      }),
    ]

    iter var i = 0; i < self.names.length(); i++ {
      var index = i
      boxes.append(Padding(
        EdgeInsets.only({ left: 20 }),
        Checkbox(self.chosen[index], {
          label: self.names[index],
          on_changed: @(value) {
            self.set_state(@{
              self.chosen[index] = value
            })
          },
        }),
      ))
    }

    return Padding(EdgeInsets.all(16), Column(boxes, {
      spacing: 6,
      align_items: AlignItems.START,
    }))
  }
}

run_app(App({
  windows: [Window('Sync', Downloads(), { size: Size(240, 140) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `label` | `string` | `''` | The text beside the box. Clicking the text ticks the box too. |
| `on_changed` | `function` | none | Called with the new value when the user clicks. |
| `tristate` | `bool` | `false` | Allows `nil` as the value, shown as a dash. Without it, a `nil` value raises a `ValueError`. |
| `enabled` | `bool` | `true` | Set to `false` to disable the checkbox. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

## Switch

`Switch(value, options)` is an on and off switch. Use it for a setting
that takes effect straight away, such as turning a feature on. For a
choice the user confirms later with a button, a checkbox is the usual
control.

A switch has no label of its own, so put one beside it:

```zuri
import zug { * }

class Sharing < StatefulWidget {
  create_state() {
    return SharingState()
  }
}

class SharingState < State {
  var sharing = false

  build(context) {
    return Padding(EdgeInsets.all(20), Row([
      Expanded(Column([
        Text('Screen Sharing'),
        Text(self.sharing ? 'On' : 'Off', {
          style: TextStyle({ color: Color.system('secondary_label') }),
        }),
      ], { spacing: 2, align_items: AlignItems.START })),
      Switch(self.sharing, {
        accessibility_label: 'Screen Sharing',
        on_changed: @(on) {
          self.set_state(@{
            self.sharing = on
          })
        },
      }),
    ]))
  }
}

run_app(App({
  windows: [Window('Sharing', Sharing(), { size: Size(320, 90) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `on_changed` | `function` | none | Called with `true` or `false` when the user flips the switch. |
| `enabled` | `bool` | `true` | Set to `false` to disable the switch. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

Give a switch an `accessibility_label`, as above, so VoiceOver can say
what it turns on.

## RadioGroup

`RadioGroup(items, value, options)` is a set of radio buttons for one
choice among a few. `items` is a dictionary from each value to its
label, or a list of strings when the value and the label are the same.
`value` is the one that's chosen.

```zuri
import zug { * }

class Units < StatefulWidget {
  create_state() {
    return UnitsState()
  }
}

class UnitsState < State {
  var unit = 'cm'

  build(context) {
    return Padding(EdgeInsets.all(20), Column([
      Text('Show measurements in'),
      RadioGroup(
        { mm: 'Millimetres', cm: 'Centimetres', pt: 'Points' }, self.unit, {
          on_changed: @(unit) {
            self.set_state(@{
              self.unit = unit
            })
          },
        },
      ),
    ], { spacing: 8, align_items: AlignItems.START }))
  }
}

run_app(App({
  windows: [Window('Units', Units(), { size: Size(260, 140) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `on_changed` | `function` | none | Called with the value of the button the user picked. |
| `axis` | `string` | `Axis.VERTICAL` | Lays the buttons out in a column, or in a row with `Axis.HORIZONTAL`. |
| `spacing` | `number` | `6` in a column, `16` in a row | The space between buttons. |
| `enabled` | `bool` | `true` | Set to `false` to disable every button in the group. |
| `on_escape` | `function` | none | Called when Escape is pressed while one of the buttons has focus. |

With more than five or so choices, a [`Dropdown`](#dropdown) takes less
room.

## Radio

`Radio(value, group_value, options)` is a single radio button. It's
selected when `value` equals `group_value`, and choosing it calls
`on_changed` with its `value`. Reach for it when a `RadioGroup` can't
lay the buttons out the way you need, such as with a field beside one
of them:

```zuri
import zug { * }

class Pages < StatefulWidget {
  create_state() {
    return PagesState()
  }
}

class PagesState < State {
  var which = 'all'
  var range = TextController('1-5')

  dispose() {
    self.range.dispose()
  }

  build(context) {
    var choose = @(value) {
      self.set_state(@{
        self.which = value
      })
    }

    return Padding(EdgeInsets.all(20), Column([
      Radio('all', self.which, { label: 'All pages', on_changed: choose }),
      Row([
        Radio('range', self.which, { label: 'Pages:', on_changed: choose }),
        Box({
          width: 80,
          child: TextField({
            controller: self.range,
            enabled: self.which == 'range',
          }),
        }),
      ], { spacing: 8 }),
    ], { spacing: 8, align_items: AlignItems.START }))
  }
}

run_app(App({
  windows: [Window('Print', Pages(), { size: Size(260, 110) })],
}))
```

`Radio` takes `label`, `on_changed`, `enabled` and `size`, which work
as they do for a `Checkbox`.

## SegmentedControl

`SegmentedControl(segments, selected, options)` is a row of segments
joined together. It's the usual control for switching between views of
the same content, and for small sets of options such as text alignment.

Each segment is a string, or a `Segment(label, { tooltip, enabled })`
when it needs a tooltip or has to be disabled on its own.

```zuri
import zug { * }

class Calendar < StatefulWidget {
  create_state() {
    return CalendarState()
  }
}

class CalendarState < State {
  var view = 1

  build(context) {
    var names = ['Day', 'Week', 'Month', 'Year']

    return Padding(EdgeInsets.all(20), Column([
      SegmentedControl(names, self.view, {
        on_changed: @(index) {
          self.set_state(@{
            self.view = index
          })
        },
      }),
      Expanded(Center(Text('Showing a ${names[self.view].lower()}'))),
    ], { spacing: 12 }))
  }
}

run_app(App({
  windows: [Window('Calendar', Calendar(), { size: Size(360, 160) })],
}))
```

`mode` decides what `selected` means:

| Mode | `selected` | `on_changed` gets |
| --- | --- | --- |
| `SegmentMode.ONE` | The index of the selected segment, or `nil` for none. This is the default. | An index. |
| `SegmentMode.ANY` | A list of the indices that are selected. | A list of indices. |
| `SegmentMode.MOMENTARY` | Ignored, as nothing stays selected. Pass `nil`. | Not called. Use `on_pressed`, which gets the index clicked. |

A momentary control works like a row of buttons, such as the Back and
Forward pair in a browser:

```zuri,ignore
SegmentedControl(['‹', '›'], nil, {
  mode: SegmentMode.MOMENTARY,
  on_pressed: @(index) {
    if index == 0 {
      self.go_back()
    } else {
      self.go_forward()
    }
  },
})
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | `string` | `SegmentMode.ONE` | How segments are chosen, as described above. |
| `on_changed` | `function` | none | Called with the new selection in `ONE` and `ANY` mode. |
| `on_pressed` | `function` | none | Called with the index clicked in `MOMENTARY` mode. |
| `enabled` | `bool` | `true` | Set to `false` to disable the whole control. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

## Dropdown

`Dropdown(items, value, options)` is a button that shows the current
choice and opens a menu of the others. It suits longer lists, and lists
whose length you don't know in advance.

Items are strings, or `DropdownItem(value, label, options)` when the
value you store differs from the label people read.
`DropdownItem.separator()` adds a line between groups of items:

```zuri
import zug { * }

class Sorting < StatefulWidget {
  create_state() {
    return SortingState()
  }
}

class SortingState < State {
  var order = 'date'

  build(context) {
    var orders = [
      DropdownItem('name', 'Name'),
      DropdownItem('date', 'Date Modified'),
      DropdownItem('size', 'Size'),
      DropdownItem.separator(),
      DropdownItem('tags', 'Tags', { enabled: false }),
    ]

    return Padding(EdgeInsets.all(20), Row([
      Text('Sort by'),
      Dropdown(orders, self.order, {
        on_changed: @(order) {
          self.set_state(@{
            self.order = order
          })
        },
      }),
    ], { spacing: 8 }))
  }
}

run_app(App({
  windows: [Window('Files', Sorting(), { size: Size(280, 80) })],
}))
```

`on_changed` gets the item's value, not its label or its position, so
you can reorder the items without changing any other code.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `on_changed` | `function` | none | Called with the value of the item the user picked. |
| `enabled` | `bool` | `true` | Set to `false` to disable the dropdown. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

A `DropdownItem` takes `enabled: false` to show an item the user can't
pick. Its label is the value written as text when you leave it out.

# Components

This is the reference for every widget zug provides, grouped by what
it's for. Each page shows a widget in a short program you can run, lists
the options it takes, and calls out the details that are easy to miss.

Every example is a complete program. Save one as `example.zu` in a
project that has zug installed and start it with `zuri run example.zu`.

## Text and images

| Widget | Use it for |
| --- | --- |
| [`Text`](text-and-images.md#text) | Labels, headings and paragraphs. |
| [`Image`](text-and-images.md#image) | A picture from a file, from memory, or from the system's symbols. |
| [`Icon`](text-and-images.md#icon) | A system symbol sized and coloured like text. |

## Buttons and choices

| Widget | Use it for |
| --- | --- |
| [`Button`](buttons.md#button) | An action, or a toggle that stays pressed in. |
| [`Checkbox`](choices.md#checkbox) | A yes or no choice with a label. |
| [`Switch`](choices.md#switch) | A setting that's on or off. |
| [`RadioGroup`](choices.md#radiogroup) | One choice from a short list. |
| [`Radio`](choices.md#radio) | A single radio button, when you lay the group out yourself. |
| [`SegmentedControl`](choices.md#segmentedcontrol) | A row of segments for switching views or picking options. |
| [`Dropdown`](choices.md#dropdown) | One choice from a longer list, shown in a menu. |

## Numbers, dates and colours

| Widget | Use it for |
| --- | --- |
| [`Slider`](values.md#slider) | A number in a range, chosen by dragging. |
| [`Stepper`](values.md#stepper) | A number changed one step at a time. |
| [`DatePicker`](values.md#datepicker) | A date, a time, or both. |
| [`ColorWell`](values.md#colorwell) | A colour, picked from the system's colour panel. |

## Progress and levels

| Widget | Use it for |
| --- | --- |
| [`ProgressBar`](indicators.md#progressbar) | How far a task has got, or that it's still going. |
| [`Spinner`](indicators.md#spinner) | Work in progress with no measure of how much is left. |
| [`LevelIndicator`](indicators.md#levelindicator) | A capacity, a relevance, or a star rating. |

## Text input

| Widget | Use it for |
| --- | --- |
| [`TextField`](text-input.md#textfield) | A single line of text, including passwords. |
| [`TextArea`](text-input.md#textarea) | Several lines of text that scroll. |
| [`SearchField`](text-input.md#searchfield) | A search box with a clear button. |
| [`ComboBox`](text-input.md#combobox) | A text field with a list of suggestions. |
| [`TextController`](text-input.md#textcontroller) | Holding the text of a field and changing it from code. |

## Layout

| Widget | Use it for |
| --- | --- |
| [`Row`, `Column` and `Flex`](rows-and-columns.md#row-and-column) | Children side by side or one above another. |
| [`Expanded` and `Flexible`](rows-and-columns.md#expanded-and-flexible) | A child that takes the space left over. |
| [`Spacer` and `Gap`](rows-and-columns.md#spacer-and-gap) | Empty space, stretchy or fixed. |
| [`Box`](boxes.md#box) | Size, padding, margins, colour, border, corners and shadow in one widget. |
| [`Padding`](boxes.md#padding) | Space around a child. |
| [`Align` and `Center`](boxes.md#align-and-center) | Placing a smaller child inside the space it's given. |
| [`AspectRatio`](boxes.md#aspectratio) | A child with a fixed shape. |
| [`FitContent`](boxes.md#fitcontent) | A child exactly as wide, or as tall, as its content. |
| [`Visibility`](boxes.md#visibility) | Hiding a child, with or without keeping its state. |
| [`Responsive`](boxes.md#responsive) | A layout that changes with the space available. |
| [`Stack` and `Positioned`](stacks-grids-and-tables.md#stack-and-positioned) | Children layered on top of each other. |
| [`Wrap`](stacks-grids-and-tables.md#wrap) | Children that flow onto new lines when they run out of room. |
| [`Grid` and `GridCell`](stacks-grids-and-tables.md#grid-and-gridcell) | Children in equal columns, such as a keypad. |
| [`Table`](stacks-grids-and-tables.md#table) | Rows of widgets lined up in columns, such as a form. |

## Scrolling and lists

| Widget | Use it for |
| --- | --- |
| [`ScrollView`](scrolling-and-lists.md#scrollview) | Content larger than the space it's in. |
| [`ListView`](scrolling-and-lists.md#listview) | A short scrolling list. |
| [`ListView.builder`](scrolling-and-lists.md#listviewbuilder) | A list of any length, built as it scrolls. |
| [`DataTable`](scrolling-and-lists.md#datatable) | The native table, with columns, sorting and selection. |

## Containers

| Widget | Use it for |
| --- | --- |
| [`TabView`](containers.md#tabview) | Pages of content behind tabs. |
| [`SplitView`](containers.md#splitview) | Panes the user can resize, such as a sidebar. |
| [`GroupBox`](containers.md#groupbox) | A titled box around related controls. |
| [`Separator`](containers.md#separator) | A thin line between groups of content. |
| [`VisualEffect`](containers.md#visualeffect) | A translucent system material, such as a sidebar's. |

## Drawing and input

| Widget | Use it for |
| --- | --- |
| [`Canvas`](drawing-and-input.md#canvas) | Anything you draw yourself, such as a chart. |
| [`PointerArea`](drawing-and-input.md#pointerarea) | Clicks, drags, hovering and scrolling over any widget. |
| [`ContextMenu`](drawing-and-input.md#contextmenu) | A menu that opens on a right click. |
| [`Shortcuts`](drawing-and-input.md#shortcuts) | Keyboard shortcuts for part of a window. |
| [`KeyListener`](drawing-and-input.md#keylistener) | Every key that reaches part of a window. |

## Animation

| Widget | Use it for |
| --- | --- |
| [`AnimatedBox`](animation.md#animatedbox) | A `Box` that moves smoothly to its new size, colour or position. |
| [`AnimatedValue`](animation.md#animatedvalue) | Any value that glides to its new setting. |

## Builders

| Widget | Use it for |
| --- | --- |
| [`Builder`](builders.md#builder) | A context in the middle of a larger build. |
| [`StatefulBuilder`](builders.md#statefulbuilder) | A little state without writing a class. |
| [`Observer`](builders.md#observer) | Rebuilding when an observable value changes. |
| [`PromiseBuilder`](builders.md#promisebuilder) | Showing a promise as it settles. |
| [`Style`](builders.md#style) | A style sheet for part of the tree. |

## The application

| Widget | Use it for |
| --- | --- |
| [`App`](app-and-windows.md#app) | The application: its windows, menus, style and appearance. |
| [`Window`](app-and-windows.md#window) | A window and its content. |
| [`MenuBar`, `Menu` and `MenuItem`](app-and-windows.md#menubar-menu-and-menuitem) | The menu bar and the menus in it. |

## How controls take values

Every control works the same way. You give it the value to show, and it
tells you, through `on_changed`, when the user picks a different one.
Store the new value and rebuild, and the control shows it:

```zuri
import zug { * }

class Preferences < StatefulWidget {
  create_state() {
    return PreferencesState()
  }
}

class PreferencesState < State {
  var spell_check = true

  build(context) {
    return Center(
      Checkbox(self.spell_check, {
        label: 'Check spelling while typing',
        on_changed: @(value) {
          self.set_state(@{
            self.spell_check = value
          })
        },
      }),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Preferences', Preferences(), { size: Size(320, 120) }),
    ],
  }),
)
```

If `on_changed` doesn't store the value, the checkbox goes back to
showing `self.spell_check` at the next frame. That keeps your state the
single source of truth: what's on screen always matches it.

A control with no `on_changed` can't change, so zug shows it disabled.
To disable one that has a handler, pass `enabled: false`.

## Options every widget takes

Every widget backed by a native view takes these options as well as its
own:

| Option | Type | Description |
| --- | --- | --- |
| `key` | `Key` | Identifies the widget among its siblings. See [Keys](../widgets-and-state.md#keys). |
| `tooltip` | `string` | Text shown when the pointer rests on the view. |
| `accessibility_label` | `string` | What VoiceOver reads, when the visible text isn't enough. |
| `accessibility_identifier` | `string` | A name for UI tests. `WidgetTester` finds views by it too. |

Widgets that can take keyboard focus, which includes every control on
these pages, also take `on_escape`. It's called when the user presses
Escape while the widget has focus, before any shortcut or Cancel button
sees the key.

Many controls take a `size`, one of `ControlSize.MINI`, `SMALL`,
`REGULAR` (the default) and `LARGE`. Use the smaller sizes for
inspectors and toolbars where space is tight.

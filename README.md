# zug

zug is a toolkit for desktop applications in Zuri. An interface is a tree
of widgets built from the application's state, in the manner of Flutter;
zug works out what changed and updates the native views to match. Every
control is the platform's own: on macOS a zug button is an `NSButton`, a
text field an `NSTextField`, a table an `NSTableView`, with the input
methods, accessibility, dark mode and behaviour that come with them.

```zuri
import zug { * }

class Counter < StatefulWidget {
  create_state() {
    return CounterState()
  }
}

class CounterState < State {
  var count = 0

  build(context) {
    return Padding(EdgeInsets.all(20), Column([
      Text('Pressed ${self.count} times', { style: Style.of(context).headline }),
      Button('Press', { role: ButtonRole.PRIMARY, on_pressed: @{ self.set_state(@{ self.count++ }) } }),
    ], { spacing: 12, align_items: AlignItems.START }))
  }
}

run_app(App({ title: 'Counter', windows: [Window('Counter', Counter())] }))
```

## Requirements

- Zuri 0.1 or later.
- macOS 11 or later. It is developed and tested on Apple silicon; the
  Intel code paths are written but have not been run.

The macOS backend drives AppKit directly through `ffi` and the
Objective-C runtime. Nothing is compiled when zug is installed, and an
application bundles with `zuri bundle --format app` as it is.

## Installing

Until zug is published, depend on it by path:

```toml
[dependencies]
zug = { path = "../zug" }
```

then `zuri install`.

## The pieces

**Widgets and state.** `StatelessWidget`, `StatefulWidget` with its
`State` and `set_state()`, `Inherited` for data shared down the tree,
read through the `Context` every `build()` receives. Keys (`ValueKey`,
`ObjectKey`, `UniqueKey`, `GlobalKey`) keep state with a child as it
moves. `Observable`, `ObservableValue` and the `Observer` widget hold
state several widgets show.

**Layout**, done by zug the same way on every platform: `Row`, `Column`
and `Flex` with `justify`, `align_items`, `spacing` and `shrink_wrap`;
`Expanded`, `Flexible`, `Spacer`, `Gap`; `Padding`, `Align`, `Center`;
`Box` for size, spacing, alignment and decoration; `Stack` and
`Positioned`; `Wrap`; `Grid` and `GridCell`; `AspectRatio`, `FitContent`,
`Visibility` and `Responsive`.

**Native controls.** `Text`, `Button`, `Checkbox`, `Radio` and
`RadioGroup`, `Switch`, `TextField`, `TextArea`, `SearchField`,
`ComboBox`, `Dropdown`, `Slider`, `Stepper`, `ProgressBar`, `Spinner`,
`SegmentedControl`, `DatePicker`, `ColorWell`, `LevelIndicator`,
`Separator`, `GroupBox`, `Image`, `Icon`. Controls are controlled: each
shows the value it is given and reports the user's choice through
`on_changed`; a choice the application does not accept is undone. A
control with nothing to call is disabled.

**Containers.** `ScrollView`, `ListView` and the virtualized
`ListView.builder`, `TabView`, `SplitView`, `DataTable`, `VisualEffect`.

**Drawing and pointer input.** `Canvas` paints with a `Painter`: shapes,
paths, text and images. `PointerArea` reports clicks, drags, scrolling
and hovering, sets the cursor, and opens a `ContextMenu`.

**The application.** `App` lists the open `Window`s and the `MenuBar`.
`Shortcuts` and `KeyListener` handle keys no control used. `Style` and
`StyleSheet` set text styles and the accent colour; `Appearance.of()`
says whether the system is light or dark. `Timer`, `Clipboard`,
`show_alert()`, `pick_files()`, `save_file()`, `perform_role()` and
`clear_focus()` reach the system. `run_in_background()` and `Worker`
move work to isolates, and their results arrive as `Promise`s, which
`PromiseBuilder` shows.

## Testing

`zug.testing` runs widgets on a headless backend with no display, where
every native view is a record that can be found and acted on:

```zuri
import test { * }
import zug { * }
import zug.testing { WidgetTester }

describe('the counter', @{
  it('counts presses', @{
    var tester = WidgetTester()
    tester.pump_widget(Counter())
    tester.tap('Press')
    expect(tester.find_text('Pressed 1 times')).to_have_length(1)
    tester.dispose()
  })
})
```

`zuri test` runs zug's own tests the same way. The tests in
`tests/macos.test.zu` drive real AppKit views; they open windows, so they
run only with `ZUG_MACOS_TESTS=1`.

## Examples

Single scripts, each run with `zuri run`:

- `examples/hello.zu`: a counter.
- `examples/controls.zu`: every control that chooses a value.
- `examples/containers.zu`: scrolling, tabs, split panes, a table,
  drawing and pointer input.
- `examples/services.zu`: keys, an alert, a timer, the clipboard and
  background work.

And a whole application, `examples/calculator`, its own project with its
own tests: keyed and typed calculations, a scientific keypad, memory, a
tape, and settings that persist. It bundles into a macOS application.

## Other platforms

Everything platform-specific is a backend. The core never refers to a
platform, so a backend for Linux, Windows, the web or a phone is a new
directory under `app/backends` and nothing else. `docs/backends.md`
describes the contract and how to write one; the headless backend is the
reference.

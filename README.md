# zug

Zug is a toolkit for desktop applications in Zuri. An interface is a tree
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

Zug installs from its git repository:

```console
$ zuri install --git https://github.com/zuri-lang/zug.git
```

That adds it to `project.toml`:

```toml
[dependencies]
zug = { git = "https://github.com/zuri-lang/zug.git", branch = "main" }
```

The lock file holds the commit that was installed, so `zuri restore`
installs the same one; `zuri update` moves to the newest commit on the
branch. `--rev` with a commit pins one instead of following the branch.

## Documentation

The [documentation](docs/index.md) covers every part of zug an
application uses. [Getting started](docs/getting-started.md) is the
place to begin; after that, the pages most people reach for are
[Layout](docs/layout.md), [Controls](docs/controls.md),
[Widgets and state](docs/widgets-and-state.md) and
[Testing](docs/testing.md).

## Examples

Single scripts, each run with `zuri run`:

- `examples/hello.zu`: a counter.
- `examples/controls.zu`: every control that chooses a value.
- `examples/containers.zu`: scrolling, tabs, split panes, a table,
  drawing and pointer input.
- `examples/animation.zu`: a panel that eases open, a value that glides
  to new settings, and a pulse.
- `examples/services.zu`: keys, an alert, a timer, the clipboard and
  background work.
- `examples/conformance.zu`: every widget zug has in one working window,
  the gallery the backend conformance check builds, with a switch that
  swaps every widget to a second set of options.

And a whole application, `examples/calculator`, its own project with its
own tests: keyed and typed calculations, a scientific keypad, memory, a
tape, and settings that persist. It bundles into a macOS application.

## Testing

`zuri test` runs zug's tests on the headless backend, with no display.
The tests in `tests/macos.test.zu` drive real AppKit views and open
windows, so they run only with `ZUG_MACOS_TESTS=1`. Your own
application's tests use `zug.testing` the same way; see
[Testing](docs/testing.md).

## Other platforms

Everything platform-specific is a backend, so bringing zug to Linux,
Windows or anywhere else means writing a backend and nothing more.
[Writing a backend](docs/backends.md) describes the contract, and
`tools/check_backend.zu` checks a new backend against it.

# When something goes wrong

A desktop application shouldn't vanish because one button handler had a
bug. zug keeps running through errors in your code, tells you about each
one, and makes the common mistakes easy to spot.

## Mistakes in options

Widgets check their options when you make them. A misspelt or wrongly
typed option raises an `OptionError` straight away, at the line that
made the widget, with a suggestion when one is close:

```console
OptionError: unknown option 'colour' for Box; did you mean 'color'? Known options: alignment, border, child, …
```

Other mistakes raise as they're found: a `ValueError` for a value out of
range, such as a `Grid` with no columns, and a `TypeError` for something
of the wrong kind, such as a `Table` row that isn't a list.

## Errors in your callbacks

Code that the platform calls, such as a button's `on_pressed`, a timer,
a menu item or a promise callback, can't let an error escape into
AppKit's event loop. zug catches it and passes it to `report_error()`,
which by default prints the error, what was running and the stack trace
to standard error, and lets the application carry on:

```console
zug: error while running the 'pressed' handler of a button view
  ValueError: the expression ends too soon
    at /Users/ada/calc/app/ui.zu:212 -> submit()
```

`set_error_handler(handler)` replaces that, which is how you'd log
errors to a file or send them to a crash reporter. The handler gets the
error and a short description of what was running:

```zuri
import zug { * }

set_error_handler(@(error, context) {
  echo 'while ${context}: ${typeof(error)}: ${error.message}'
})
```

`set_error_handler(nil)` puts the default back. In tests, a handler that
collects errors lets you assert that nothing went wrong behind the
scenes.

`report_error(error, context)` is there for your own code too, when you
catch something you can't handle but want it on the record.

## Errors in build

When a widget's `build()` raises, zug reports the error and shows a red
`ErrorBox` with the message in place of that widget. The rest of the
window keeps working, so you can see exactly which part broke while you
fix it.

## Layout warnings

A `Row` or `Column` whose children don't fit prints one warning, saying
by how much it overflowed. A `Column` with `Expanded` children inside a
`ScrollView` raises, because there's no spare space to share.
[Layout](layout.md#when-layout-goes-wrong) explains both.

## The errors zug raises

All of them are `ZugError`s, so one `catch` can take any of them:

| Error | Means |
| --- | --- |
| `OptionError` | an option that doesn't exist or has the wrong type; `owner` and `option` say which |
| `LayoutError` | a layout that can't be satisfied |
| `LifecycleError` | something used at the wrong time, such as a timer with no application running, or `set_state()` after `dispose()` |
| `BackendError` | the platform failed to do something |
| `UnsupportedError` | the running backend has no support for something, such as a view kind |

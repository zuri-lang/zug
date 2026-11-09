# zug documentation

zug is a toolkit for desktop applications in Zuri. You describe your
interface as widgets built from your application's state, and zug keeps
real native controls in step with it. These pages cover everything an
application uses, roughly in the order you'll need it.

## Start here

- [Getting started](getting-started.md): a project, a first window, and
  the ideas everything else builds on.
- [Widgets and state](widgets-and-state.md): stateless and stateful
  widgets, the life of a `State`, keys, the `Context`, sharing data with
  `Inherited` and observable values.
- [Layout](layout.md): how sizes are decided, rows and columns, boxes,
  stacks, wrapping, grids and tables, and what to do when layout goes
  wrong.

## Building the interface

- [Controls](controls.md): text, buttons, checkboxes and switches, radio
  buttons, segmented controls, drop-downs, sliders, steppers, date and
  colour pickers, progress, images and icons.
- [Text input](text-input.md): text fields, text areas, search fields and
  combo boxes, and the `TextController` behind them.
- [Scrolling, lists and tables](lists-and-containers.md): scroll views,
  lists of any length, native tables, tabs, split views and materials.
- [Drawing and pointer input](drawing-and-pointer.md): `Canvas` and its
  painter, paths, mouse and trackpad input, cursors and context menus.
- [Animation](animation.md): widgets that animate their own changes,
  `Animation`, tweens and curves.
- [Styling](styling.md): text styles, colours, borders and shadows, a
  style sheet for the whole application, and light and dark.

## The application

- [Applications, windows and menus](apps-windows-and-menus.md): the
  `App`, opening and closing windows, the menu bar, and keyboard
  shortcuts.
- [Printing](printing.md): printing pages you draw, the page setup
  and print dialogs, and PDF files.
- [Timers, promises and the system](services.md): promises, timers, the
  clipboard, alerts, file dialogs, dialogs of your own, font and colour
  choosers, and work on other isolates.
- [When something goes wrong](errors.md): how zug reports mistakes and
  errors, and how to handle them yourself.

## Testing and performance

- [Testing](testing.md): testing widgets without a display, with
  `WidgetTester`.
- [Performance](performance.md): what things cost, measured, and how to
  keep an application fast.

## Other platforms

- [Writing a backend](backends.md): the contract a platform implements,
  and how to check a new backend against it.

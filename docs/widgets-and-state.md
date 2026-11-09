# Widgets and state

A zug interface is a tree of widgets that you rebuild whenever your data
changes. This page covers the two kinds of widget you write yourself,
the life of a `State`, keys, and the ways to share data between widgets
that aren't next to each other.

## Stateless widgets

A `StatelessWidget` turns its fields into other widgets and nothing
more. Classes in Zuri are sealed, so declare every field with `var`:

```zuri
import zug { * }

class Badge < StatelessWidget {
  var label
  var count = 0

  @new(label: string, count: number) {
    self.label = label
    self.count = count
  }

  build(context) {
    return Row([
      Text(self.label),
      Box({
        padding: EdgeInsets.symmetric({ horizontal: 6, vertical: 1 }),
        color: Color.system('red'),
        radius: 8,
        child: Text('${self.count}', {
          style: TextStyle({ size: 11, weight: FontWeight.BOLD, color: Color.WHITE }),
        }),
      }),
    ], { spacing: 6 })
  }
}

run_app(App({ windows: [Window('Inbox', Center(Badge('Inbox', 3)))] }))
```

Widgets are cheap. Making a new `Badge` on every build is normal, and
it's how a change reaches the screen: the parent builds a `Badge` with a
new `count`, and zug updates only the label whose text changed.

## Stateful widgets

When a widget has to remember something between builds, such as what
the user typed or which tab is open, make it a `StatefulWidget`. The
widget itself still only carries configuration. The memory lives in its
`State`:

```zuri,ignore
class Stopwatch < StatefulWidget {
  create_state() {
    return StopwatchState()
  }
}

class StopwatchState < State {
  var seconds = 0
  var ticker

  init_state() {
    self.ticker = Timer.periodic(Duration.seconds(1), @(timer) {
      self.set_state(@{
        self.seconds++
      })
    })
  }

  dispose() {
    self.ticker.cancel()
  }

  build(context) {
    return Text('${self.seconds} s')
  }
}
```

Change fields inside `set_state()`. zug then rebuilds this widget at the
next frame, together with anything else that changed, so ten
`set_state()` calls in one click cost one rebuild.

### The life of a State

| Method | Called |
| --- | --- |
| `init_state()` | once, when the widget first appears. Start timers and subscriptions here. |
| `did_change_dependencies()` | after `init_state()`, and again whenever an `Inherited` this state reads changes. |
| `did_update_widget(previous)` | when the parent rebuilds with a new widget of the same type. `self.widget` is the new one; compare it with `previous` to react to changed options. |
| `build(context)` | whenever the interface needs describing again. |
| `deactivate()` | when the widget is taken out of the tree, possibly to be put back elsewhere in the same frame. |
| `dispose()` | once, when it's gone for good. Cancel timers, dispose controllers and animations. |

`self.widget` is always the current widget, `self.mounted()` tells you
whether the state is still in the tree, and `self.context()` is its
`Context`. Calling `set_state()` after `dispose()` raises a
`LifecycleError`, which usually means a timer or a promise outlived the
widget that started it.

## Keys

zug matches each new widget with the old one in the same place, by
type. That's right almost always, but it goes wrong when stateful
children are reordered: their states stay where they were, attached to
the wrong data. Give such children keys and zug matches them by key
instead:

```zuri,ignore
Column(self.todos.map(@(todo) => TodoRow(todo, { key: ValueKey(todo.id) })))
```

| Key | Matches |
| --- | --- |
| `ValueKey(value)` | widgets whose values are equal, such as an id. |
| `ObjectKey(object)` | widgets for the same object. |
| `UniqueKey()` | only itself; use one to force a fresh state. |
| `GlobalKey()` | one widget anywhere in the tree, and lets you reach it. |

A `GlobalKey` can find its widget from outside: `current_state()`,
`current_widget()` and `current_context()` return what it's attached
to, or `nil` when nothing is. Use them sparingly; passing data down is
usually clearer.

## The Context

Every `build()` receives a `Context`, which is the widget's place in the
tree. It's how a widget reads things provided further up:

- `context.depend_on(Type)` finds the nearest `Inherited` of that type,
  and rebuilds this widget when it changes.
- `context.get_inherited(Type)` finds it without depending on it.
- `context.find_ancestor_widget(Type)` and
  `context.find_ancestor_state(Type)` look upwards for a widget or state.
- `context.size()` is the size the widget was laid out at, once it has
  been, which is handy in event handlers.

## Sharing data down the tree

An `Inherited` widget makes a value available to everything below it,
however deep. Widgets that read it with `depend_on()` are rebuilt when a
new one replaces it, and `update_should_notify()` says whether the
change matters:

```zuri
import zug { * }

class Units < Inherited {
  var metric = true

  @new(metric: bool, child) {
    parent(child)
    self.metric = metric
  }

  update_should_notify(previous) {
    return previous.metric != self.metric
  }

  static of(context) {
    return context.depend_on(Units)
  }
}

class Distance < StatelessWidget {
  var meters = 0

  @new(meters: number) {
    self.meters = meters
  }

  build(context) {
    return Units.of(context).metric
      ? Text('${self.meters} m')
      : Text('${(self.meters * 3.281).round()} ft')
  }
}

run_app(App({ windows: [Window('Trail', Units(false, Center(Distance(1200))))] }))
```

`Style` and `Appearance` are built this way, which is why
`Style.of(context)` works anywhere.

## Observable values

Some state belongs to no single widget: the open document, the signed-in
user, a setting several windows show. Keep it in an `ObservableValue`
and let widgets watch it with `Observer`:

```zuri
import zug { * }

var theme_name = ObservableValue('Light')

run_app(App({ windows: [Window('Theme', Padding(EdgeInsets.all(16), Column([
  Observer(theme_name, @(context) => Text('Theme: ${theme_name.value()}')),
  Button('Switch', { on_pressed: @{
    theme_name.update(@(name) => name == 'Light' ? 'Dark' : 'Light')
  } }),
], { spacing: 8 })))] }))
```

`value()` reads it, `set(value)` replaces it, and `update(fn)` replaces
it with what `fn` returns. Only the `Observer` rebuilds when it changes,
not the window around it, which is the cheapest rebuild zug has (see
[Performance](performance.md)).

For something richer than one value, subclass `Observable` and call
`self.notify()` whenever it changes. `add_listener()` and
`remove_listener()` let ordinary code watch one too, and
`Observable.merge([a, b])` notifies whenever any of several does.
`dispose()` an observable you made once nothing needs it.

## Building in place

Sometimes a whole class is more ceremony than a piece of interface
deserves:

- `Builder(@(context) => ...)` builds from a context, for when you need
  `Style.of(context)` in the middle of a larger build.
- `StatefulBuilder(@(context, set_state) => ...)` gives a small piece of
  interface its own `set_state`.
- `PromiseBuilder(promise, @(context, snapshot) => ...)` builds from a
  `Promise` as it settles. The snapshot has `is_pending()`,
  `is_resolved()` and `is_rejected()`, and `value` or `error` once it's
  settled.

```zuri,ignore
PromiseBuilder(self.loading, @(context, snapshot) {
  if snapshot.is_pending() {
    return Spinner()
  }

  return snapshot.is_rejected()
    ? Text('Could not load: ${snapshot.error.message}')
    : Text('${snapshot.value.length()} notes')
})
```

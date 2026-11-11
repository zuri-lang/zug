# Builders

[Components](index.md) › Builders

These widgets build their child with a function you give them. They
save you writing a class for a small piece of interface, and some of
them rebuild that piece on their own when something changes.

## Builder

`Builder(builder)` calls `builder` with a context and shows the widget
it returns. The context is the one at that point in the tree, which is
what you need to read something provided just above, such as a `Style`
that's part of the same build:

```zuri
import zug { * }

var sheet = StyleSheet({
  headline: TextStyle({ size: 22, weight: FontWeight.BOLD }),
})

var page = Style(sheet, Center(Builder(@(context) => Text('Welcome back', {
  style: Style.of(context).headline,
}))))

run_app(App({
  windows: [Window('Welcome', page, { size: Size(300, 120) })],
}))
```

Called with the outer context, `Style.of()` would find the sheet above
the window instead of this one.

## StatefulBuilder

`StatefulBuilder(builder)` calls `builder` with a context and a
`set_state` function, which gives a small piece of interface a rebuild
of its own without a `State` class. The state lives in variables the
function closes over:

```zuri
import zug { * }

var liked = false

var heart = Center(StatefulBuilder(
  @(context, set_state) => Button(liked ? '♥ Liked' : '♡ Like', {
    on_pressed: @{
      set_state(@{
        liked = !liked
      })
    },
  }),
))

run_app(App({
  windows: [Window('Post', heart, { size: Size(200, 100) })],
}))
```

Only the builder's own widgets rebuild. Once the state grows past a
value or two, a `StatefulWidget` is easier to follow.

## Observer

`Observer(observable, builder, options)` rebuilds whenever an
`Observable` notifies. The builder gets the context, and it's the only
part that rebuilds, which makes an observer the cheapest way to keep a
small part of a large window up to date.

An `ObservableValue` is the simplest observable: one value that you can
read, replace and watch from anywhere.

```zuri
import zug { * }

var unread = ObservableValue(3)

var toolbar = Padding(EdgeInsets.all(16), Row([
  Observer(unread, @(context) => Text('${unread.value()} unread')),
  Spacer(),
  Button('Read One', { on_pressed: @{
    unread.update(@(count) => (count - 1).max(0))
  } }),
  Button('New Mail', { on_pressed: @{
    unread.update(@(count) => count + 1)
  } }),
], { spacing: 8 }))

run_app(App({
  windows: [Window('Mail', toolbar, { size: Size(360, 70) })],
}))
```

`TextController`, `ScrollController` and `Animation` are observables
too, so an observer of a text field's controller rebuilds as the user
types.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `child` | `Widget` | none | A part that doesn't depend on the observable. It's built once, and passed to the builder as its second argument so it can place it. |

[Observable values](../widgets-and-state.md#observable-values) explains
`ObservableValue` and how to write observables of your own.

## PromiseBuilder

`PromiseBuilder(promise, builder)` builds from where a `Promise` has
got to, and again when it settles. The builder gets the context and a
snapshot of the promise:

| Snapshot | Holds |
| --- | --- |
| `is_pending()` | Whether the promise is still waiting. |
| `is_resolved()` | Whether it resolved. `value` then holds what it resolved with. |
| `is_rejected()` | Whether it failed. `error` then holds the error. |

```zuri
import zug { * }

def count_words(text) {
  return text.split(' ').length()
}

class WordCount < StatefulWidget {
  create_state() {
    return WordCountState()
  }
}

class WordCountState < State {
  var counting

  init_state() {
    self.counting = run_in_background(
      count_words, 'the quick brown fox ' * 5000,
    )
  }

  build(context) {
    return Center(PromiseBuilder(self.counting, @(context, snapshot) {
      if snapshot.is_pending() {
        return Spinner()
      }

      if snapshot.is_rejected() {
        return Text('Could not count: ${snapshot.error.message}')
      }

      return Text('${snapshot.value} words')
    }))
  }
}

run_app(App({
  windows: [Window('Words', WordCount(), { size: Size(240, 100) })],
}))
```

Start the work once, in `init_state()` or in an event handler, and keep
the promise in your state, as the example does. A promise made inside
`build()` would start the work again on every build.

When you build a `PromiseBuilder` with a different promise, it follows
the new one and forgets the old.

## Style

`Style(sheet, child)` gives everything below it a
[`StyleSheet`](../styling.md#a-style-sheet-for-the-application), the
text styles, accent colour and spacing that `Style.of(context)` reads
and every `Text` starts from.

Most applications set one sheet for everything through `App`'s `style`
option. Use a `Style` when one part of the window should look different,
such as an inspector with smaller text:

```zuri
import zug { * }

var compact = StyleSheet({ text: TextStyle({ size: 11 }), spacing: 4 })

class Inspector < StatelessWidget {
  build(context) {
    var style = Style.of(context)

    return Column([
      Text('Width: 640'),
      Text('Height: 480'),
    ], { spacing: style.spacing, align_items: AlignItems.START })
  }
}

var layout = Padding(EdgeInsets.all(16), Row([
  Expanded(Center(Text('Canvas'))),
  Style(compact, Inspector()),
]))

run_app(App({
  windows: [Window('Layout', layout, { size: Size(400, 160) })],
}))
```

`Style.of(context)` returns the nearest sheet above `context`, or the
platform's usual one when there's none, and rebuilds the widget when
that sheet changes.

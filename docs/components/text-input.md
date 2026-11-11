# Text input

[Components](index.md) › Text input

zug has four widgets for typing. `TextField` takes a line of text,
`TextArea` takes several, `SearchField` is a search box, and `ComboBox`
is a field with a list of suggestions. They share most of their
options, and they all hold their text in one of two ways.

## Holding the text

**With a [`TextController`](#textcontroller).** The controller holds
the text, and the field always shows what's in it. This is the usual
choice. The field needs nothing from you while the user types, and you
read the text whenever you need it:

```zuri
import zug { * }

class SignIn < StatefulWidget {
  create_state() {
    return SignInState()
  }
}

class SignInState < State {
  var name = TextController()
  var password = TextController()
  var message = ''

  dispose() {
    self.name.dispose()
    self.password.dispose()
  }

  sign_in() {
    self.set_state(@{
      self.message = 'Signing in as ${self.name.text()}…'
    })
  }

  build(context) {
    return Padding(
      EdgeInsets.all(20),
      Column(
        [
          TextField({
            controller: self.name,
            placeholder: 'Name',
            autofocus: true,
          }),
          TextField({
            controller: self.password,
            placeholder: 'Password',
            obscure: true,
            on_submitted: @(text) {
              self.sign_in()
            },
          }),
          Row([
            Text(self.message),
            Spacer(),
            Button('Sign In', {
              role: ButtonRole.PRIMARY,
              on_pressed: @{
                self.sign_in()
              },
            }),
          ]),
        ],
        { spacing: 10, align_items: AlignItems.STRETCH },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Sign In', SignIn(), { size: Size(320, 150) }),
    ],
  }),
)
```

Make controllers in the `State`, as fields or in `init_state()`, so
they live as long as the field does, and dispose them in `dispose()`.

**With `text` and `on_changed`.** The field shows exactly the `text`
you give it and reports each edit, the same way a checkbox reports a
click. You decide what to keep:

```zuri,ignore
TextField({
  text: self.code,
  placeholder: 'Discount code',
  on_changed: @(text) {
    self.set_state(@{
      self.code = text.upper()
    })
  },
})
```

That suits text that's part of your state anyway, or text you want to
reshape as it's typed, like the capitals above. A field with `text` but
no `on_changed` is read only, since nothing could ever change it. A
field can't take both `text` and a `controller`.

## Limiting what's typed

`max_length` caps how many characters a field holds. `input_filter`
looks at every edit before it's made: it's called with the new text and
the old, and returns `true` to accept the edit, `false` to refuse it, or
a string to put in its place.

```zuri
import zug { * }

var code = TextField({
  controller: TextController(),
  placeholder: '6 digit code',
  max_length: 6,
  input_filter: @(text, previous) => text.match('/^[0-9]*$/') ? true : false,
})

run_app(
  App({
    windows: [
      Window('Verify', Padding(EdgeInsets.all(20), code), {
        size: Size(240, 70),
      }),
    ],
  }),
)
```

A refused edit leaves the text as it was.

## Options all text inputs take

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `controller` | `TextController` | a controller of its own | Holds the text. See [Holding the text](#holding-the-text). |
| `text` | `string` | none | The text to show, for a field you control yourself. |
| `on_changed` | `function` | none | Called with the text after each edit. |
| `on_submitted` | `function` | none | Called with the text when the user presses Return. Not used by `TextArea`, where Return starts a new line. |
| `enabled` | `bool` | `true` | Set to `false` to disable the field. |
| `autofocus` | `bool` | `false` | Puts the cursor in the field when it first appears. |
| `max_length` | `number` | none | The most characters the field holds. |
| `input_filter` | `function` | none | Accepts, refuses or rewrites each edit. See [Limiting what's typed](#limiting-whats-typed). |
| `on_focus_changed` | `function` | none | Called with `true` when the field gets keyboard focus and `false` when it loses it. |
| `on_escape` | `function` | none | Called when the user presses Escape in the field. |

They also take the [options every widget takes](index.md#options-every-widget-takes).

## TextField

`TextField(options)` is a single line of editable text. The sign in
form above shows the common case. Besides the options all text inputs
take, it has these:

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `placeholder` | `string` | none | Grey text shown while the field is empty. |
| `obscure` | `bool` | `false` | Hides what's typed, for passwords. |
| `read_only` | `bool` | `false` | Lets the user select and copy the text, but not change it. |
| `bezel` | `string` | `FieldBezel.ROUNDED` | The field's border. `ROUNDED` is the standard field, `SQUARE` has square corners, and `NONE` has no border or background, so the text looks like a label until it's edited. |
| `align` | `string` | `TextAlign.START` | `START`, `CENTER` or `END`. Numbers usually read best at the `END`. |
| `style` | `TextStyle` | the system's | How the text looks. |

In a row, a text field is 160 points wide unless you give it an
`Expanded`, to take the width that's left, or a `Box` with a width. In
a column that stretches its children, it's as wide as the column.

## TextArea

`TextArea(options)` is several lines of editable text that scroll when
they don't fit. It fills all the space it's given, so put it in an
`Expanded`, or a `Box` with a height.

```zuri
import zug { * }

class Notes < StatefulWidget {
  create_state() {
    return NotesState()
  }
}

class NotesState < State {
  var notes = TextController('Things to pack:\n')

  dispose() {
    self.notes.dispose()
  }

  build(context) {
    var counter = Observer(self.notes, @(context) {
      var words = self.notes.text().split(' ').filter(@(word) => word != '')
      return Text('${words.length()} words', {
        style: TextStyle({ color: Color.system('secondary_label') }),
      })
    })

    return Padding(
      EdgeInsets.all(16),
      Column(
        [
          Expanded(
            TextArea({
              controller: self.notes,
              style: TextStyle({ size: 14 }),
              autofocus: true,
            }),
          ),
          counter,
        ],
        { spacing: 8, align_items: AlignItems.END },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Notes', Notes(), { size: Size(400, 300) }),
    ],
  }),
)
```

A controller is an `Observable`, so an [`Observer`](builders.md#observer)
of it rebuilds as the user types. Here only the word count rebuilds,
not the text area.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `read_only` | `bool` | `false` | Lets the user select and copy the text, but not change it. |
| `style` | `TextStyle` | the system's | How the text looks. |
| `wrap` | `bool` | `true` | Wraps long lines at the edge. With `false`, lines run on and the area scrolls sideways. |
| `bordered` | `bool` | `true` | Draws a border around the area. |

## SearchField

`SearchField(options)` is the rounded field with a magnifying glass and
a button that clears it. It works well with `text` and `on_changed`,
filtering as the user types:

```zuri
import zug { * }

var fruit = ['Apple', 'Apricot', 'Banana', 'Cherry', 'Grape', 'Mango']

class Finder < StatefulWidget {
  create_state() {
    return FinderState()
  }
}

class FinderState < State {
  var query = ''

  build(context) {
    var needle = self.query.lower()
    var found = fruit.filter(@(name) => name.lower().contains(needle))

    return Padding(
      EdgeInsets.all(16),
      Column(
        [
          SearchField({
            text: self.query,
            placeholder: 'Search fruit',
            on_changed: @(text) {
              self.set_state(@{
                self.query = text
              })
            },
          }),
          Expanded(ListView(found.map(@(name) => Text(name)), { spacing: 4 })),
        ],
        { spacing: 10, align_items: AlignItems.STRETCH },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Fruit', Finder(), { size: Size(260, 240) }),
    ],
  }),
)
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `placeholder` | `string` | none | Grey text shown while the field is empty. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

Escape empties a search field that has text in it, as search fields do
everywhere on the Mac. Give it an `on_escape` to do something else.

## ComboBox

`ComboBox(items, options)` is a text field with a button that opens a
list of suggestions. The user can type anything, or pick a suggestion,
which sets the text and calls `on_selected` with its index and its
text.

```zuri
import zug { * }

class Calculator < StatefulWidget {
  create_state() {
    return CalculatorState()
  }
}

class CalculatorState < State {
  var input = TextController()
  var recent = ['2 ^ 10', 'sqrt(2)', '22 / 7']

  dispose() {
    self.input.dispose()
  }

  build(context) {
    return Padding(
      EdgeInsets.all(20),
      ComboBox(self.recent, {
        controller: self.input,
        placeholder: 'Expression',
        completes: true,
        on_submitted: @(text) {
          self.set_state(@{
            self.recent = [text] + self.recent.filter(@(r) => r != text)
          })
          self.input.clear()
        },
      }),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Calculate', Calculator(), { size: Size(280, 70) }),
    ],
  }),
)
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `placeholder` | `string` | none | Grey text shown while the field is empty. |
| `completes` | `bool` | `false` | Finishes what the user types from the suggestions. |
| `visible_items` | `number` | `8` | How many suggestions the list shows before it scrolls. |
| `on_selected` | `function` | none | Called with the index and the text of a suggestion the user picks. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

## TextController

`TextController(text)` holds the text of a field, starting from `text`
or from an empty string. Give it to a field as `controller` and use it
to read and change the field from code:

| Method | What it does |
| --- | --- |
| `text()` | Returns the text. |
| `set_text(text)` | Replaces the text and puts the cursor at the end. |
| `clear()` | Empties the field. |
| `select(start, end)` | Selects the characters from `start` up to `end`. Equal positions put the cursor there. |
| `select_all()` | Selects all the text. |
| `selection()` | Returns the selection as `[start, end]`, or `nil` before the field has reported one. |
| `request_focus()` | Puts the cursor in the field. |
| `dispose()` | Lets the controller go once nothing uses it. |

`set_text()`, `clear()`, `select()` and the other changes return the
controller, so they chain:

```zuri,ignore
self.search.set_text('report').select_all().request_focus()
```

A controller is an `Observable`. `add_listener(fn)` calls `fn` after
every change, and an [`Observer`](builders.md#observer) rebuilds with
it, as the notes example shows.

## Focus

The focused field in a window gets the keys the user types.
`autofocus: true` focuses a field when it first appears, and
`controller.request_focus()` focuses it later.

To take focus away, so that keys go to the window's
[shortcuts](../apps-windows-and-menus.md#the-keyboard) again, call
`clear_focus(context)`. A calculator does this after the user presses
Return in its expression field, so the next digits go back to the
keypad.

Escape in a field goes to its `on_escape` first. Without one, Escape
empties a search field that has text in it, and otherwise presses the
window's [`CANCEL` button](buttons.md#roles), if there is one, as it
would in any dialog.

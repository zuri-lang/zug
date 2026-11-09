# Text input

zug has four widgets for typing: `TextField` for a line of text,
`TextArea` for several, `SearchField` for search boxes, and `ComboBox`
for a field with suggestions. They share their options and their two
ways of holding text.

## Two ways to hold the text

**With a `TextController`**, the controller holds the text and the field
always shows it. This is the usual choice: the field needs nothing from
you while the user types, and you read the text when you need it:

```zuri
import zug { * }

class Signup < StatefulWidget {
  create_state() {
    return SignupState()
  }
}

class SignupState < State {
  var name = TextController()
  var notes = TextController('Anything else?')

  dispose() {
    self.name.dispose()
    self.notes.dispose()
  }

  build(context) {
    return Padding(EdgeInsets.all(16), Column([
      TextField({ controller: self.name, placeholder: 'Your name', autofocus: true }),
      Box({ height: 80, child: TextArea({ controller: self.notes }) }),
      Button('Send', { role: ButtonRole.PRIMARY, on_pressed: @{
        echo 'sending ${self.name.text()}'
      } }),
    ], { spacing: 8, align_items: AlignItems.STRETCH }))
  }
}

run_app(App({ windows: [Window('Sign up', Signup(), { size: Size(320, 220) })] }))
```

The controller is also how you change the field from code:
`set_text()` replaces the text, `clear()` empties it, `select(start,
end)` and `select_all()` select part or all of it, `selection()` says
what's selected, and `request_focus()` puts the cursor in it. It's an
`Observable`, so an `Observer` of it rebuilds as the user types.

**With `text` and `on_changed`**, the field is controlled like a
checkbox: it shows exactly the `text` you give it, reports each edit,
and you decide what to keep:

```zuri,ignore
TextField({
  text: self.query,
  on_changed: @(text) {
    self.set_state(@{
      self.query = text.lower()
    })
  },
})
```

That's handy when the text is part of your state anyway, or when you
want to reshape what's typed. Give `text` without `on_changed` and the
field is read-only, since nothing could ever change it. You can't give
both `text` and a `controller`.

## Limiting what's typed

`max_length` caps the length, and `input_filter` looks at each edit. It
is called with the new text and the old, and returns `true` to accept
it, `false` to refuse it, or a string to put in its place:

```zuri
import zug { * }

var digits_only = TextField({
  controller: TextController(),
  max_length: 6,
  placeholder: 'Code',
  input_filter: @(text, previous) => text.match('/^[0-9]*$/') ? true : false,
})
```

## TextField

A single line. Return calls `on_submitted` with the text. Besides the
options above it takes `placeholder`, `obscure: true` for passwords,
`read_only`, `enabled`, `bezel` (`FieldBezel.ROUNDED`, `SQUARE`, or
`NONE` for text that looks like a label until edited), `align`, `style`,
`autofocus` and `on_focus_changed`, called with whether it has focus.

## TextArea

Several lines that scroll when they don't fit. It fills the space it's
given, so put it in an `Expanded`, or a `Box` with a height. It takes
`wrap` (true) to wrap long lines, `bordered` (true), `read_only`,
`style`, `max_length`, `input_filter`, `autofocus` and
`on_focus_changed`.

## SearchField

The rounded field with a magnifying glass and a clear button. It takes
`placeholder`, `size`, `enabled`, `on_changed` and `on_submitted`.

## ComboBox

A field with a list of suggestions. The text can be anything typed;
picking a suggestion sets the text and calls `on_selected` with its
index and text:

```zuri
import zug { * }

var expression = ComboBox(['1 + 1', 'sqrt(2)', '2 ^ 10'], {
  controller: TextController(),
  placeholder: 'Expression',
  completes: true,
  on_selected: @(index, text) {},
  on_submitted: @(text) {},
})
```

`completes: true` finishes what the user types from the suggestions, and
`visible_items` sets how many show at once.

## Focus

A window's focused field gets the keys. `autofocus: true` focuses a
field when it first appears, and `controller.request_focus()` focuses it
later. To take focus away, so that keys go to the window's
[shortcuts](apps-windows-and-menus.md#the-keyboard) instead, call
`clear_focus(context)`.

Every text input takes `on_escape`, called when the user presses Escape
in it. Without one, Escape does what the platform does there: a search
field, for one, empties itself.

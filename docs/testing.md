# Testing

`zug.testing` runs your widgets on a headless backend: no window
appears, nothing takes focus, and time moves only when the test says
so. Every native view becomes a record you can find, inspect and act on
the way a user would. Tests written this way are fast, and they run on a
build server with no display.

## A first test

```zuri
import test { * }
import zug { * }
import zug.testing { WidgetTester }

class Counter < StatefulWidget {
  create_state() {
    return CounterState()
  }
}

class CounterState < State {
  var count = 0

  build(context) {
    return Column([
      Text('Pressed ${self.count} times'),
      Button('Press', { on_pressed: @{
        self.set_state(@{
          self.count++
        })
      } }),
    ])
  }
}

describe('the counter', @{
  it('counts presses', @{
    var t = WidgetTester()
    t.pump_widget(Counter())
    t.tap('Press')
    t.tap('Press')
    expect(t.find_text('Pressed 2 times')).to_have_length(1)
    t.dispose()
  })
})
```

`pump_widget(widget)` shows a
widget in a window, 800 by 600 unless you pass `{ size }`, and draws it.
`pump_app(app)` takes a whole `App` instead, for testing menus and
several windows. Call `dispose()` at the end, so the next test starts
clean.

## Finding views

| Method | Returns |
| --- | --- |
| `find_text(text)` | every view showing `text`, such as a label or a button title |
| `find_kind(kind)` | every view of a kind, such as `'checkbox'` or `'table'` |
| `find_identifier(id)` | every view with that `accessibility_identifier` |
| `one(target)` | the only view whose text or identifier is `target`, and fails if there are none or several |
| `views()` | every view |
| `state_of(Type)` | the first `State` of that type, to call its methods or read its fields |

Give the controls you test an `accessibility_identifier`. Tests that
find things by identifier survive changes to wording, and the same
identifiers serve UI tests and VoiceOver.

A view has `props`, the values zug last gave it, `text()`, `kind()` and
`window_frame()`, its position in the window as a `Rect`. That's how you
test layout:

```zuri,ignore
expect(t.one('sidebar').window_frame().width).to_be(200)
```

## Acting like a user

`tap(target)` clicks a button, or toggles a toggle button. For anything
else, `fire(target, event, payload)` reports an event from a view, just
as the native control would, and draws what changed:

```zuri,ignore
t.fire('remember', 'changed', { value: true })
t.fire('name', 'changed', { text: 'Ada' })
t.fire('name', 'submitted', nil)
t.fire('people', 'changed', { selected: [2] })
t.fire('people', 'activated', { row: 2 })
```

The common events are:

| View | Event and payload |
| --- | --- |
| Checkbox, switch, slider, stepper, date picker, colour well, level | `changed` with `{ value }` |
| Radio | `changed` with `{ selected: true }` |
| Dropdown, segmented control, tabs | `changed` with `{ selected }`, an index |
| Text field, text area, search field, combo box | `changed` with `{ text }`; `submitted`; `escape` for a field |
| Data table | `changed` with `{ selected }`, a list of rows; `activated` with `{ row }`; `sorted` with `{ column, ascending }` |
| Scroll view | `scrolled` with `{ x, y }` |
| Pointer area | `tap`, `drag` and the rest, with `{ x, y }`, and `dx`, `dy`, `clicks` and `modifiers` where they matter |

Like the real controls, a view first shows the value it reported. If
your code refuses it, the next frame puts the old value back, and
`props` shows that.

Windows and the system can be acted on too:

- `window().press_key('cmd+k')` presses a key in the first window, and
  `press_key('shift+8', '*')` gives the characters it types. It returns
  whether anything handled the key.
- `window().resize(Size(400, 300))` resizes it, and `click_close()`
  clicks its close button.
- `choose_menu('File', 'New')` chooses a menu item.
- `backend.set_appearance('dark')` switches the appearance.

## Time, timers and animations

Headless time stands still until you move it. `advance(seconds)`, or
`advance(Duration.ms(250))`, moves the clock on, fires each timer as it
falls due, and draws a frame for every display refresh while something
animates:

```zuri,ignore
t.tap('Save')
t.advance(Duration.seconds(2))
expect(t.find_text('Saved')).to_be_empty()
```

`pump()` draws any frames that are waiting without moving the clock.

## Dialogs and the clipboard

Dialogs don't open headless; they wait in `backend.dialogs` for the test
to answer them. `backend.answer_dialog(answer)` answers the oldest, with
what the real dialog would give: `{ button: 1, suppressed: false }` for
an alert, a list of paths or `nil` for an open panel, a path or `nil`
for a save panel.

```zuri,ignore
t.tap('Delete')
expect(t.backend.dialogs[0].spec.title).to_be('Delete “Q3 report”?')
t.backend.answer_dialog({ button: 0, suppressed: false })
```

The clipboard is in memory, so `Clipboard.write_text()` and
`read_text()` work as usual without touching yours. `backend.beeps`
counts calls to `beep()`, `backend.opened_urls` lists what `open_url()`
was given, and `backend.performed_roles` lists the menu roles performed.

# Buttons

[Components](index.md) › Buttons

## Button

`Button(label, options)` is the platform's push button. Give it an
`on_pressed` function, and it's called each time the user clicks the
button or presses it from the keyboard.

```zuri
import zug { * }

class Export < StatefulWidget {
  create_state() {
    return ExportState()
  }
}

class ExportState < State {
  var status = 'Ready to export.'

  build(context) {
    var actions = Row(
      [
        Button('', {
          variant: ButtonVariant.HELP,
          accessibility_label: 'Help with exporting',
          on_pressed: @{
            open_url('https://example.com/help/export')
          },
        }),
        Spacer(),
        Button('Cancel', {
          role: ButtonRole.CANCEL,
          on_pressed: @{
            self.set_state(@{
              self.status = 'Cancelled.'
            })
          },
        }),
        Button('Export', {
          role: ButtonRole.PRIMARY,
          on_pressed: @{
            self.set_state(@{
              self.status = 'Exported.'
            })
          },
        }),
      ],
      { spacing: 8 },
    )

    return Padding(
      EdgeInsets.all(20),
      Column([Expanded(Center(Text(self.status))), actions]),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Export', Export(), { size: Size(380, 140) }),
    ],
  }),
)
```

A button without `on_pressed` can't do anything, so it's shown disabled.
That's a convenient way to disable an action until it makes sense, for
example by passing `nil` until a form is filled in.

### Roles

A button's `role` says what part it plays, which decides both how it
looks and which keys press it:

| Role | Meaning |
| --- | --- |
| `ButtonRole.NORMAL` | An ordinary button. This is the default. |
| `ButtonRole.PRIMARY` | The default action. It's drawn in the accent colour, and Return presses it. |
| `ButtonRole.CANCEL` | The way out. Escape presses it. |
| `ButtonRole.DESTRUCTIVE` | An action that deletes or discards something. It's drawn as a warning. |

Give a window one `PRIMARY` button at most, and put it last in the row,
as the example does. That's where people on macOS expect to find it.

### Variants

`variant` picks the kind of button:

| Variant | Looks like |
| --- | --- |
| `ButtonVariant.PUSH` | The standard rounded button, of a fixed height. This is the default. |
| `ButtonVariant.FLEXIBLE` | A rounded button that grows to any height, such as a calculator key. |
| `ButtonVariant.SQUARE` | A button with square corners that grows to any size. |
| `ButtonVariant.TOOLBAR` | A button for a toolbar. |
| `ButtonVariant.INLINE` | A small rounded button for use inside other controls. |
| `ButtonVariant.CIRCULAR` | A round button, usually holding a single character or symbol. |
| `ButtonVariant.BORDERLESS` | Only the title, with no bezel. |
| `ButtonVariant.HELP` | The round help button with a question mark. Give it an empty label and an `accessibility_label` for VoiceOver. |

### Toggle buttons

Give a button `selected` and it becomes a toggle: it stays pressed in
while `selected` is `true`, and each click calls `on_selected_changed`
with the state the user wants. As with any control, it shows whatever
the next build says:

```zuri
import zug { * }

class Formatting < StatefulWidget {
  create_state() {
    return FormattingState()
  }
}

class FormattingState < State {
  var bold = false
  var italic = false

  build(context) {
    return Center(
      Row(
        [
          Button('Bold', {
            variant: ButtonVariant.TOOLBAR,
            selected: self.bold,
            on_selected_changed: @(on) {
              self.set_state(@{
                self.bold = on
              })
            },
          }),
          Button('Italic', {
            variant: ButtonVariant.TOOLBAR,
            selected: self.italic,
            on_selected_changed: @(on) {
              self.set_state(@{
                self.italic = on
              })
            },
          }),
        ],
        { spacing: 4, shrink_wrap: true },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Format', Formatting(), { size: Size(240, 100) }),
    ],
  }),
)
```

For a group where only one can be chosen, such as text alignment, a
[`SegmentedControl`](choices.md#segmentedcontrol) is usually clearer.

### Options

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `on_pressed` | `function` | none | Called with no arguments when the button is pressed. |
| `role` | `string` | `ButtonRole.NORMAL` | The part the button plays. See [Roles](#roles). |
| `variant` | `string` | `ButtonVariant.PUSH` | The kind of button. See [Variants](#variants). |
| `shortcut` | `string` | none | A key that presses the button, such as `cmd+s` or `escape`. See [key bindings](../apps-windows-and-menus.md#the-keyboard). |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |
| `enabled` | `bool` | `true` | Set to `false` to disable the button. |
| `style` | `TextStyle` | the system's | The style of the title. |
| `tint` | `Color` | none | A colour for the bezel, for buttons that need to stand out, such as the operator keys of a calculator. |
| `selected` | `bool` | none | Makes the button a toggle, pressed in while `true`. |
| `on_selected_changed` | `function` | none | Called with `true` or `false` when the user clicks a toggle. |

Buttons also take the [options every widget takes](index.md#options-every-widget-takes),
including `on_escape`.

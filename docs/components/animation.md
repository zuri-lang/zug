# Animation

[Components](index.md) › Animation

These two widgets animate their own changes. Build them with a new
value, and they move to it smoothly instead of jumping. For animations
you start and stop yourself, such as a pulse, see the
[Animation guide](../animation.md).

## AnimatedBox

`AnimatedBox(options)` is a [`Box`](boxes.md#box) that moves to its new
size, spacing, alignment, colour, border, corner radius, shadow and
opacity whenever you build it with new ones.

```zuri
import zug { * }

class Details < StatefulWidget {
  create_state() {
    return DetailsState()
  }
}

class DetailsState < State {
  var open = false

  build(context) {
    var toggle = Button(self.open ? 'Hide Details' : 'Show Details', {
      on_pressed: @{
        self.set_state(@{
          self.open = !self.open
        })
      },
    })

    var details = AnimatedBox({
      duration: Duration.ms(250),
      curve: Curve.EASE_OUT,
      height: self.open ? 90 : 0,
      opacity: self.open ? 1 : 0,
      clip: true,
      padding: EdgeInsets.all(10),
      color: Color.system('control_background'),
      radius: 6,
      child: Text('Created 3 March. Last opened yesterday by Ada.'),
    })

    return Padding(EdgeInsets.all(16), Column([
      toggle,
      details,
    ], { spacing: 10, align_items: AlignItems.STRETCH }))
  }
}

run_app(App({
  windows: [Window('Report', Details(), { size: Size(340, 180) })],
}))
```

A change in the middle of a move starts a new move from wherever the
box has got to, so clicking quickly never makes it jump.

Some changes happen at once, because there's nothing in between to
move through. The child and `clip` switch straight away, and so does a
size that appears or disappears, such as `width` going from unset to
`100`. To animate a box opening from nothing, as the example does, give
it a size of `0` rather than leaving the size out.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `duration` | `Duration` | required | How long each move takes. |
| `curve` | `Curve` | `Curve.EASE` | How the move speeds up and slows down. See [Curves](../animation.md#tweens-and-curves). |
| `on_end` | `function` | none | Called with no arguments when a move finishes. |

It also takes every [`Box` option](boxes.md#box). These move:
`width`, `height`, `min_width`, `max_width`, `min_height`,
`max_height`, `width_factor`, `height_factor`, `padding`, `margin`,
`alignment`, `color`, `border`, `radius`, `shadow` and `opacity`.

## AnimatedValue

`AnimatedValue(value, builder, options)` does the same for any value.
When you build it with a new `value`, it moves there over the
`duration`, and calls `builder` with the context and the value at each
step on the way:

```zuri
import zug { * }

class Storage < StatefulWidget {
  create_state() {
    return StorageState()
  }
}

class StorageState < State {
  var used = 0.25

  build(context) {
    var meter = AnimatedValue(self.used, @(context, value) => Column([
      ProgressBar({ value }),
      Text('${(value * 100).round()}% used'),
    ], { spacing: 6, align_items: AlignItems.STRETCH }), {
      duration: Duration.ms(600),
      curve: Curve.EASE_IN_OUT,
    })

    return Padding(EdgeInsets.all(20), Column([
      meter,
      Button('Add Photos', { on_pressed: @{
        self.set_state(@{
          self.used = (self.used + 0.2).min(1)
        })
      } }),
    ], { spacing: 12, align_items: AlignItems.START }))
  }
}

run_app(App({
  windows: [Window('Storage', Storage(), { size: Size(300, 130) })],
}))
```

The value can be a number, a `Color`, a `Point`, a `Size`, a `Rect`,
`EdgeInsets`, an `Alignment`, a `Border` or a `Shadow`. Only what
`builder` returns is rebuilt at each step, so keep the builder to the
part that changes.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `duration` | `Duration` | required | How long each move takes. |
| `curve` | `Curve` | `Curve.EASE` | How the move speeds up and slows down. |
| `on_end` | `function` | none | Called with no arguments when a move finishes. |

System colours such as `Color.system('accent')` have no fixed value to
blend from, so they switch halfway through instead of fading.

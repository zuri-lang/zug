# Animation

Most animation in an application is a change that shouldn't happen
abruptly: a panel opening, a value settling, a colour fading. zug has
two ways to do it. Widgets that animate their own changes cover most
cases with no setup at all, and `Animation` gives you full control when
you need it.

While anything animates, zug draws a frame for every refresh of the
display, which is 120 a second on a ProMotion screen. When nothing
animates, it draws nothing.

## Animating a change

[`AnimatedBox`](components/animation.md#animatedbox) is a `Box` that
moves to its new size, spacing, colour, border, corner radius, shadow
and opacity whenever you build it with new ones.
[`AnimatedValue`](components/animation.md#animatedvalue) does the same
for any value, such as a number or a colour, and builds from it at each
step.

Neither needs any setup. Build them with a new value, as you would any
widget, and they take it from there:

```zuri
import zug { * }

class Status < StatefulWidget {
  create_state() {
    return StatusState()
  }
}

class StatusState < State {
  var online = false

  build(context) {
    var light = AnimatedBox({
      duration: Duration.ms(300),
      width: 12,
      height: 12,
      radius: 6,
      color: Color.system(self.online ? 'green' : 'gray'),
    })

    return Center(
      Row(
        [
          light,
          Switch(self.online, {
            accessibility_label: 'Online',
            on_changed: @(on) {
              self.set_state(@{
                self.online = on
              })
            },
          }),
        ],
        { spacing: 10, shrink_wrap: true },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Status', Status(), { size: Size(200, 100) }),
    ],
  }),
)
```

A change that arrives in the middle of a move starts a new move from
wherever it got to, so a user clicking quickly never sees a jump.

## Animation

For animations you start and stop yourself, such as a pulse, a loading
shimmer or items arriving one after another, use an `Animation`. It
runs a number from 0 to 1 over a `duration`, changing once a frame
while it runs. It's an `Observable`, so an
[`Observer`](components/builders.md#observer) of it rebuilds with every
step:

```zuri
import zug { * }

class Pulse < StatefulWidget {
  create_state() {
    return PulseState()
  }
}

class PulseState < State {
  var beat = Animation({ duration: Duration.ms(900) })
  var glow = Tween(0.3, 1, Curve.EASE_IN_OUT)

  init_state() {
    self.beat.repeat({ reverse: true })
  }

  dispose() {
    self.beat.dispose()
  }

  build(context) {
    return Observer(
      self.beat,
      @(context) => Box({
        width: 16,
        height: 16,
        radius: 8,
        color: Color.system('green'),
        opacity: self.glow.at(self.beat.value()),
      }),
    )
  }
}

run_app(
  App({
    windows: [Window('Pulse', Center(Pulse()))],
  }),
)
```

| Method | What it does |
| --- | --- |
| `forward(from)` | Runs to `upper`, from where it is or from `from`. |
| `reverse(from)` | Runs back to `lower`. |
| `animate_to(target, { duration, curve })` | Runs to any value, along a curve. |
| `repeat({ reverse, count })` | Runs to `upper` and starts again, from `lower` each time, or back and forth with `reverse`. It repeats `count` times, or until it's stopped. |
| `stop()` | Stops where it is. |
| `set(value)` | Jumps straight to a value. |
| `value()` | Where it is now. |
| `is_animating()` | Whether it's moving. |

Each run returns a `Promise` of whether it finished: `true` when it got
to the end, and `false` when something stopped it or started another
run. That makes sequences easy to write:

```zuri,ignore
self.slide.forward().then(@(finished) {
  if finished {
    self.fade.forward()
  }
})
```

An `Animation` takes these options:

| Option | Default | Description |
| --- | --- | --- |
| `duration` | required | How long a run from `lower` to `upper` takes. |
| `reverse_duration` | `duration` | How long a run back takes, when it should differ. |
| `lower` | `0` | The value at the start. |
| `upper` | `1` | The value at the end. |
| `value` | `lower` | Where it starts. |

A run over part of the range takes the same part of the duration, so
reversing halfway through takes half as long.

An `Animation` needs a running application, so create it as a field of
a `State` or in `init_state()`, and dispose it in `dispose()`.

## Tweens and curves

An `Animation` gives you progress from 0 to 1. A `Tween` turns that
progress into the values you actually want, eased by a `Curve`:

```zuri
import zug { * }

var width = Tween(80, 240, Curve.EASE_OUT)
var tint = Tween(Color.hex('#d1d1d6'), Color.hex('#0a84ff'))

echo width.at(0.5)
echo tint.at(0.25).to_hex()
```

`Tween(begin, end, curve)` takes the same kinds of value as
`AnimatedValue`, and `at(t)` gives the value at progress `t`.
`lerp(a, b, t)` does a single step without a tween. System colours such
as `Color.system('accent')` have no fixed value, so they switch halfway
instead of blending.

The curves are:

| Curve | Feels like |
| --- | --- |
| `Curve.LINEAR` | An even speed throughout. |
| `Curve.EASE` | The web's default: a quick start and a gentle finish. |
| `Curve.EASE_IN` | A slow start. Suits things leaving. |
| `Curve.EASE_OUT` | Slowing into the end. Suits things arriving. |
| `Curve.EASE_IN_OUT` | Slow at both ends. Suits things moving across. |
| `Curve.BACK_IN`, `Curve.BACK_OUT` | Pulling back before going, or overshooting and settling. |
| `Curve.BOUNCE_OUT` | Bouncing against the end. |

`Curve.cubic(x1, y1, x2, y2)` makes any curve that CSS's
`cubic-bezier()` can, `Curve.steps(n)` jumps in `n` steps, and
`flipped()` turns an ease in into an ease out. `Curve(@(t) => t * t)`
makes one from a function.

`Curve.interval(start, end, curve)` holds still until `start`, moves
until `end`, and then holds at the end. Give several tweens different
intervals of one animation, and they move one after another:

```zuri,ignore
var title = Tween(0, 1, Curve.interval(0, 0.6, Curve.EASE_OUT))
var body = Tween(0, 1, Curve.interval(0.3, 1, Curve.EASE_OUT))
```

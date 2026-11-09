# Animation

Most animation in an application is a change that shouldn't happen
abruptly: a panel opening, a value settling, a colour fading. zug has two
ways to do it. Widgets that animate their own changes cover most cases
with no setup, and `Animation` gives you full control when you need it.

While anything animates, zug draws a frame for every refresh of the
display, 120 a second on a ProMotion screen. When nothing does, it draws
nothing.

## Animating a change

`AnimatedBox` is a `Box` that moves to its new size, spacing, colour,
border, corner radius, shadow and opacity whenever you build it with new
ones:

```zuri
import zug { * }

class Drawer < StatefulWidget {
  create_state() {
    return DrawerState()
  }
}

class DrawerState < State {
  var open = false

  build(context) {
    return Column([
      Button(self.open ? 'Hide details' : 'Show details', { on_pressed: @{
        self.set_state(@{
          self.open = !self.open
        })
      } }),
      AnimatedBox({
        duration: Duration.ms(250),
        curve: Curve.EASE_OUT,
        height: self.open ? 120 : 0,
        opacity: self.open ? 1 : 0,
        clip: true,
        child: Text('The details.'),
      }),
    ], { spacing: 8, align_items: AlignItems.STRETCH })
  }
}

run_app(App({ windows: [Window('Drawer', Drawer())] }))
```

It takes every `Box` option, plus `duration` (required), `curve`
(`Curve.EASE` when left out) and `on_end`, called when a move finishes.
A change in the middle of a move starts a new one from wherever it got
to, so clicking quickly never makes it jump. The child and the options
that can't move, such as `clip`, change at once, and so does a size that
appears or disappears, like `width` going from unset to 100, because
there's nothing to move from.

`AnimatedValue` does the same for any value, and builds from it on the
way:

```zuri
import zug { * }

var progress = AnimatedValue(0.75, @(context, value) => Column([
  ProgressBar({ value }),
  Text('${(value * 100).round()}%'),
], { spacing: 4, align_items: AlignItems.STRETCH }), { duration: Duration.ms(400) })
```

The value can be a number, a `Color`, a `Point`, a `Size`, a `Rect`,
`EdgeInsets`, an `Alignment`, a `Border` or a `Shadow`.

## Animation

For animations you start and stop yourself, such as a pulse, a loading
shimmer or a staggered entrance, use an `Animation`. It runs a number
from 0 to 1 over a `duration`, changes once a frame while it runs, and
is an `Observable`, so an `Observer` of it rebuilds with every step:

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
    return Observer(self.beat, @(context) => Box({
      width: 16,
      height: 16,
      radius: 8,
      color: Color.system('green'),
      opacity: self.glow.at(self.beat.value()),
    }))
  }
}

run_app(App({ windows: [Window('Pulse', Center(Pulse()))] }))
```

| Method | What it does |
| --- | --- |
| `forward(from)` | runs to `upper`, from the current value or from `from` |
| `reverse(from)` | runs back to `lower` |
| `animate_to(target, { duration, curve })` | runs to any value, along a curve |
| `repeat({ reverse, count })` | runs to `upper` and starts again, from `lower` each time or back and forth with `reverse`, `count` times or until stopped |
| `stop()` | stops where it is |
| `set(value)` | jumps straight to a value |
| `value()`, `is_animating()` | where it is, and whether it's moving |

Each run returns a `Promise` of whether it finished: `true` when it got
to its end, `false` when something stopped or replaced it. That makes
sequences easy to write:

```zuri,ignore
self.slide.forward().then(@(finished) {
  if finished {
    self.fade.forward()
  }
})
```

An `Animation` takes `duration`, `reverse_duration` when going back
should take a different time, `lower` and `upper` (0 and 1), and
`value`, where it starts. A run over part of the range takes the same
part of the duration, so reversing halfway through takes half as long.

It needs a running application, so create it as a field of a `State` or
in `init_state()`, and dispose it in `dispose()`.

## Tweens and curves

An `Animation` gives you progress from 0 to 1; a `Tween` turns that into
the values you actually want, eased by a `Curve`:

```zuri
import zug { * }

var width = Tween(80, 240, Curve.EASE_OUT)
var tint = Tween(Color.hex('#d1d1d6'), Color.hex('#0a84ff'))
echo width.at(0.5)
echo tint.at(0.25).to_hex()
```

`Tween(begin, end, curve)` takes the same kinds of value as
`AnimatedValue`, and `at(t)` gives the value at progress `t`. `lerp(a,
b, t)` does a single step without a tween. System colours such as
`Color.system('accent')` have no fixed value, so they switch halfway
rather than blend.

The curves are:

| Curve | Feels like |
| --- | --- |
| `Curve.LINEAR` | even speed throughout |
| `Curve.EASE` | the web's default: quick start, gentle finish |
| `Curve.EASE_IN` | starts slowly; for things leaving |
| `Curve.EASE_OUT` | slows into the end; for things arriving |
| `Curve.EASE_IN_OUT` | slow at both ends; for things moving across |
| `Curve.BACK_IN`, `Curve.BACK_OUT` | pulls back before going, or overshoots and settles |
| `Curve.BOUNCE_OUT` | bounces against the end |

`Curve.cubic(x1, y1, x2, y2)` makes any curve CSS's `cubic-bezier()`
can, `Curve.steps(n)` jumps in `n` steps, and `flipped()` turns an
ease-in into an ease-out. `Curve(@(t) => t * t)` makes one from a
function.

`Curve.interval(start, end, curve)` holds still until `start`, moves
until `end`, and then holds at the end. Give several tweens different
intervals of one animation and they move one after another:

```zuri,ignore
var title = Tween(0, 1, Curve.interval(0, 0.6, Curve.EASE_OUT))
var body = Tween(0, 1, Curve.interval(0.3, 1, Curve.EASE_OUT))
```

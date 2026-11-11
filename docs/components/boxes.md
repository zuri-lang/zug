# Boxes, spacing and sizing

[Components](index.md) › Boxes, spacing and sizing

## Box

`Box(options)` does sizing, spacing and decoration in one widget. It
can fix a child's size, put space inside and outside it, and give it a
background, a border, rounded corners and a shadow.

```zuri
import zug { * }

class Card < StatelessWidget {
  var title
  var body

  @new(title: string, body: string) {
    self.title = title
    self.body = body
  }

  build(context) {
    return Box({
      width: 220,
      padding: EdgeInsets.all(14),
      color: Color.system('control_background'),
      border: Border.all(1, Color.system('separator')),
      radius: 10,
      shadow: Shadow({ blur: 8, y: 3 }),
      child: Column(
        [
          Text(self.title, {
            style: TextStyle({ weight: FontWeight.SEMIBOLD }),
          }),
          Text(self.body, {
            style: TextStyle({ color: Color.system('secondary_label') }),
          }),
        ],
        { spacing: 4, align_items: AlignItems.START },
      ),
    })
  }
}

var cards = Center(
  Row(
    [
      Card('Drafts', '3 documents waiting for review.'),
      Card('Shared', 'Ada edited the budget an hour ago.'),
    ],
    { spacing: 16, shrink_wrap: true },
  ),
)

run_app(
  App({
    windows: [
      Window('Overview', cards, { size: Size(520, 200) }),
    ],
  }),
)
```

`padding` goes inside the decoration, between the border and the child,
and `margin` goes outside it. A box with no child and no size fills the
space it's given, which makes an empty coloured box a handy background
or placeholder.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `child` | `Widget` | none | What's inside the box. |
| `width`, `height` | `number` | none | Fix the box's size, in points. |
| `min_width`, `max_width` | `number` | none | Limit how narrow or wide the box can be. |
| `min_height`, `max_height` | `number` | none | Limit how short or tall the box can be. |
| `width_factor`, `height_factor` | `number` | none | Make the box that fraction of the space it's given, such as `0.5` for half. |
| `padding` | `EdgeInsets` | none | Space between the edge of the box and its child. |
| `margin` | `EdgeInsets` | none | Space around the outside of the box. |
| `alignment` | `Alignment` | none | Where a smaller child sits inside the box. See [Align and Center](#align-and-center). |
| `color` | `Color` | none | The background colour. |
| `border` | `Border` | none | A border on every side, made with `Border.all(width, color)`. |
| `radius` | `number` | `0` | The corner radius, in points. |
| `shadow` | `Shadow` | none | A shadow behind the box. |
| `clip` | `bool` | `false` | Clips the child to the box, rounded corners included. |
| `opacity` | `number` | `1` | How opaque the box and its child are, from `0` (invisible) to `1`. |

`Shadow(options)` takes a `color` (a translucent black), `blur` (`4`),
and `x` (`0`) and `y` (`2`) for how far it falls to the right and down.

Decoration costs a native view. A box that only sizes and spaces has
none, so use `Box` freely for layout.

## Padding

`Padding(insets, child)` puts space around a child. It's the plainest
way to keep content off the edges of a window:

```zuri
import zug { * }

var content = Padding(
  EdgeInsets.symmetric({ horizontal: 24, vertical: 16 }),
  Text('24 points at the sides, and 16 at the top and bottom.'),
)

run_app(
  App({
    windows: [
      Window('Padding', content, { size: Size(360, 100) }),
    ],
  }),
)
```

`EdgeInsets` says how much space goes on each side:

| Constructor | Gives |
| --- | --- |
| `EdgeInsets.all(16)` | The same on every side. |
| `EdgeInsets.symmetric({ horizontal: 12, vertical: 4 })` | One amount at the left and right, another at the top and bottom. |
| `EdgeInsets.only({ left: 8, top: 4 })` | Only the sides you name. The rest are 0. |
| `EdgeInsets(left, top, right, bottom)` | Each side in turn. |

## Align and Center

`Align(child, options)` places a smaller child inside the space it's
given. `Center(child)` is the common case, the child in the middle.

```zuri
import zug { * }

var corner = Stack(
  [
    Center(Text('Drop files here')),
    Align(Padding(EdgeInsets.all(12), Button('Browse…', { on_pressed: @{} })), {
      alignment: Alignment.BOTTOM_RIGHT,
    }),
  ],
  { fit: StackFit.EXPAND },
)

run_app(
  App({
    windows: [
      Window('Import', corner, { size: Size(360, 220) }),
    ],
  }),
)
```

There are nine named alignments: `TOP_LEFT`, `TOP_CENTER`, `TOP_RIGHT`,
`CENTER_LEFT`, `CENTER`, `CENTER_RIGHT`, `BOTTOM_LEFT`, `BOTTOM_CENTER`
and `BOTTOM_RIGHT`. `Alignment(x, y)` takes any point, where `-1` is the
left or top edge, `0` the middle and `1` the right or bottom edge.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `alignment` | `Alignment` | `Alignment.CENTER` | Where the child goes. Only `Align` takes it, as `Center` always centres. |
| `width_factor` | `number` | none | Makes the widget that many times as wide as its child, instead of filling the space. |
| `height_factor` | `number` | none | Makes the widget that many times as tall as its child. |

`Align` and `Center` let their child be any size up to theirs. That's
the usual way to stop a parent from stretching a child: wrap the child
in a `Center`.

## AspectRatio

`AspectRatio(ratio, child)` sizes its child to a shape, as its width
divided by its height. A video preview, for example, stays 16 by 9 at
any window width:

```zuri
import zug { * }

var preview = Padding(
  EdgeInsets.all(16),
  Column(
    [
      AspectRatio(
        16 / 9,
        Box({
          color: Color.BLACK,
          radius: 6,
          child: Center(Icon('play.fill', { size: 32, color: Color.WHITE })),
        }),
      ),
      Text('Keynote, 32 minutes'),
    ],
    { spacing: 8, align_items: AlignItems.STRETCH },
  ),
)

run_app(
  App({
    windows: [
      Window('Preview', preview, { size: Size(400, 300) }),
    ],
  }),
)
```

## FitContent

`FitContent(child, options)` makes its child as wide as its content
wants to be, even where the parent would stretch it. It's the tool for
a column of buttons that should all be as wide as the widest one:

```zuri
import zug { * }

var actions = Center(
  FitContent(
    Column(
      [
        Button('New Document', { on_pressed: @{} }),
        Button('Open…', { on_pressed: @{} }),
        Button('Open Recent', { on_pressed: @{} }),
      ],
      { spacing: 8, shrink_wrap: true, align_items: AlignItems.STRETCH },
    ),
  ),
)

run_app(
  App({
    windows: [
      Window('Welcome', actions, { size: Size(320, 200) }),
    ],
  }),
)
```

The column stretches its buttons, and `FitContent` makes the column
exactly as wide as the widest button needs.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `axis` | `string` | `Axis.HORIZONTAL` | Which dimension to fit. With `Axis.VERTICAL` it fits the height instead. |

## Visibility

`Visibility(child, options)` shows or hides a child. A hidden child
leaves the tree, so its state is lost, just as if you'd left it out of
the build.

With `keep_state: true`, a hidden child stays in the tree with its
state and its native views, takes no space and isn't shown. Showing it
again is instant, and anything typed into it is still there:

```zuri
import zug { * }

class Search < StatefulWidget {
  create_state() {
    return SearchState()
  }
}

class SearchState < State {
  var advanced = false

  build(context) {
    return Padding(
      EdgeInsets.all(16),
      Column(
        [
          Checkbox(self.advanced, {
            label: 'Advanced options',
            on_changed: @(on) {
              self.set_state(@{
                self.advanced = on
              })
            },
          }),
          Visibility(TextField({ placeholder: 'Only in folder…' }), {
            visible: self.advanced,
            keep_state: true,
          }),
        ],
        { spacing: 10, align_items: AlignItems.STRETCH },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Search', Search(), { size: Size(300, 110) }),
    ],
  }),
)
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `visible` | `bool` | `true` | Whether the child is shown. |
| `keep_state` | `bool` | `false` | Keeps a hidden child's state and views, ready to show again. |

## Responsive

`Responsive(builder)` builds its child from the space it's given, so a
layout can change shape with the window. The builder gets a context and
the `Constraints` the widget was given, whose `max_width` and
`max_height` are the room available:

```zuri
import zug { * }

var mail = Responsive(@(context, constraints) {
  var list = Box({
    color: Color.system('control_background'),
    child: Center(Text('Messages')),
  })
  var reader = Center(Text('Message'))

  if constraints.max_width >= 560 {
    return Row([
      Box({ width: 220, child: list }),
      Expanded(reader),
    ])
  }

  return list
})

run_app(
  App({
    windows: [
      Window('Mail', mail, { size: Size(720, 400) }),
    ],
  }),
)
```

In a wide window this shows the list and the message side by side, and
in a narrow one only the list. The builder runs again whenever the space
changes, so keep it to choosing a layout.

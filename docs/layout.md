# Layout

zug lays your widgets out itself, the same way on every platform, and
then puts each native view exactly where it decided. This page explains
the one rule behind it and then goes through the layout widgets.

## The rule

**Constraints go down, sizes come up, and the parent decides where each
child goes.**

A parent tells each child how small and how large it may be: a minimum
and maximum width and height. The child picks a size within those
limits, and the parent then positions it. That's the whole system, and
nearly every layout puzzle is answered by asking what a widget was
allowed to be.

A few consequences are worth knowing up front:

- A window gives its content exactly the window's size. That's why a
  `Column` placed directly in a window fills it.
- A widget can't be bigger than its parent allows. A `Box` with
  `width: 400` inside a 300-point window is 300 points wide.
- A scroll view lets its content be as long as it likes, so content
  inside one that wants to "fill the space" has nothing to fill. That's
  the cause of the most common layout error, covered below.

## Rows and columns

`Row` lays children out left to right, `Column` top to bottom. Both take
the same options:

```zuri
import zug { * }

var toolbar = Row([
  Button('Back', { on_pressed: @{} }),
  Button('Forward', { on_pressed: @{} }),
  Spacer(),
  SearchField({ placeholder: 'Search' }),
], { spacing: 8 })
```

| Option | What it does |
| --- | --- |
| `spacing` | Points between neighbouring children. |
| `justify` | How children spread along the axis when there's room left over: `Justify.START` (the default), `END`, `CENTER`, `SPACE_BETWEEN`, `SPACE_AROUND` or `SPACE_EVENLY`. |
| `align_items` | Where children sit across the axis: `AlignItems.CENTER` (the default), `START`, `END`, `STRETCH` to make them all as tall as the row (or as wide as the column), or `BASELINE` to line up the first line of text in a row. |
| `shrink_wrap` | Take only the space the children need along the axis, rather than all of it. |

`Flex(children, { axis: Axis.VERTICAL })` is the same widget with the
direction as an option, for when it's chosen at run time.

### Sharing the spare space

Children of a row or column are their natural size unless you say
otherwise. To make one take the space left over, wrap it:

- `Expanded(child)` takes a share of the spare space and fills it.
  `Expanded(child, { flex: 2 })` takes twice the share of a plain one.
- `Flexible(child)` may take its share, but can stay smaller if it
  doesn't need it.
- `Spacer()` is empty space that takes a share, which is how you push
  things to opposite ends.
- `Gap(12)` is a fixed amount of empty space.

```zuri
import zug { * }

var form_row = Row([
  Text('Name'),
  Expanded(TextField({ placeholder: 'Ada Lovelace' })),
], { spacing: 8, align_items: AlignItems.BASELINE })
```

## Spacing, alignment and boxes

- `Padding(EdgeInsets.all(16), child)` puts space around a child.
  `EdgeInsets.symmetric({ horizontal: 12, vertical: 4 })` and
  `EdgeInsets.only({ top: 8 })` give different sides different insets.
- `Align(child, { alignment: Alignment.BOTTOM_RIGHT })` places a smaller
  child within the space it's given. `Center(child)` is the common case.
  There are nine named alignments, from `TOP_LEFT` to `BOTTOM_RIGHT`, and
  `Alignment(x, y)` takes any point from -1 to 1 on each axis.
- `Box` does sizes and decoration in one widget:

```zuri
import zug { * }

var card = Box({
  width: 240,
  padding: EdgeInsets.all(12),
  margin: EdgeInsets.symmetric({ vertical: 6 }),
  color: Color.system('control_background'),
  border: Border.all(1, Color.system('separator')),
  radius: 8,
  shadow: Shadow({ blur: 6, y: 2 }),
  child: Text('A card'),
})
```

`Box` takes `width` and `height`, `min_width`, `max_width`, `min_height`
and `max_height`, `width_factor` and `height_factor` (a fraction of the
space it's given), `padding`, `margin`, `alignment` for its child, and
the decoration options `color`, `border`, `radius`, `shadow`, `clip` and
`opacity`. A box with no decoration is pure layout and has no native
view at all.

## Layers

`Stack` puts its children on top of each other, the first at the
bottom. Children that aren't positioned are placed by `alignment`;
`Positioned` pins a child to the stack's edges:

```zuri
import zug { * }

var thumbnail = Stack([
  Image(ImageSource.symbol('photo', { size: 48 }), { width: 120, height: 90 }),
  Positioned(
    Box({ padding: EdgeInsets.all(4), color: Color.BLACK.with_alpha(0.6), child: Text('4:12') }),
    { right: 4, bottom: 4 },
  ),
])
```

`Positioned` takes any of `left`, `top`, `right`, `bottom`, `width` and
`height`, and `Positioned.fill(child)` covers the whole stack. The
stack's `fit` decides how big unpositioned children may be:
`StackFit.LOOSE` (the default) lets them be their natural size,
`EXPAND` makes them fill it.

## Flowing, grids and tables

`Wrap` lays children along a line and starts a new line when it runs out
of room, like words in a paragraph. It's the right thing for tags,
chips and toolbars that have to fit narrow windows. It takes `axis`,
`spacing` between children, `run_spacing` between lines, and `justify`,
`run_justify` and `align_items`.

`Grid` puts children in equal columns, row by row:

```zuri
import zug { * }

var keypad = Grid([
  Button('7', { on_pressed: @{} }), Button('8', { on_pressed: @{} }), Button('9', { on_pressed: @{} }),
  GridCell(Button('0', { on_pressed: @{} }), { column_span: 2 }), Button('.', { on_pressed: @{} }),
], { columns: 3, spacing: 6, row_height: 36 })
```

Rows are as tall as their tallest child unless you fix them with
`row_height`, or with `aspect_ratio` (cell width over height).
`GridCell` makes a child span several columns or rows.

`Table` lines rows of cells up in columns, and sizes each column by its
own rule. It's the widget for forms:

```zuri
import zug { * }

var form = Table([
  [Text('Name'), TextField({ placeholder: 'Ada Lovelace' })],
  [Text('Email'), TextField({ placeholder: 'ada@example.com' })],
  [Text('Role'), Dropdown(['Analyst', 'Engineer'], 'Analyst', { on_changed: @(role) {} })],
], {
  columns: [ColumnWidth.fit(), ColumnWidth.flex()],
  align_items: AlignItems.BASELINE,
  column_spacing: 8,
  row_spacing: 6,
})
```

Each row is a plain list of widgets, and a row that's shorter than the
others leaves its last cells empty. The column rules are:

| Rule | Width |
| --- | --- |
| `ColumnWidth.fixed(120)` | exactly that many points |
| `ColumnWidth.flex()` | a share of what's left, `flex(2)` twice the share |
| `ColumnWidth.fit()` | as wide as its widest cell |
| `ColumnWidth.fraction(0.3)` | that fraction of the table's width |

Columns past the end of `columns` are `flex()`. When a table is too
narrow for its columns, the `fit()` columns give up width first.

## Sizes from shapes and content

- `AspectRatio(16 / 9, child)` keeps a width-to-height ratio.
- `FitContent(child)` makes a child exactly as wide as its content wants
  to be, which helps when a parent would otherwise stretch it. Pass
  `{ axis: Axis.VERTICAL }` for height.
- `Visibility(child, { visible: false })` takes a child off screen. With
  `keep_state: true` it keeps the child's state and native views while
  hidden, so showing it again is instant.
- `Responsive(@(context, constraints) => ...)` builds from the space it's
  given, for interfaces that change shape with the window:

```zuri
import zug { * }

var adaptive = Responsive(@(context, constraints) {
  if constraints.max_width > 600 {
    return Row([Text('Sidebar'), Expanded(Text('Content'))])
  }

  return Column([Text('Content')])
})
```

## When layout goes wrong

**"a Column with Expanded or Flexible children was given an unbounded
height"** means a `Column` inside a `ScrollView` (or anything else that
allows unlimited height) has an `Expanded` child. In a scroll view there
is no spare space to share. Give the column's content natural sizes, or
put the `Expanded` part outside the scroll view.

**"a Row overflowed by 40 points"**, printed once, means the children
need more room than the row has. Wrap the one that can shrink in
`Expanded` or `Flexible`, switch to a `Wrap`, or put the row in a
horizontal `ScrollView`.

**A widget won't get smaller or bigger.** Look at what its parent
allowed. A `Center` or `Align` loosens constraints so the child can be
smaller; `Expanded` and `align_items: AlignItems.STRETCH` force a size.

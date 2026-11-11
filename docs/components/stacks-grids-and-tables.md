# Stacks, grids and tables

[Components](index.md) › Stacks, grids and tables

## Stack and Positioned

`Stack(children, options)` lays its children on top of each other, the
first at the back. It's how you put a badge on an icon, a caption over a
picture, or a button in the corner of a panel.

The stack is as big as its largest child that isn't positioned, and
places those children by its `alignment`. Wrap a child in
`Positioned(child, options)` to pin it to the stack's edges instead:

```zuri
import zug { * }

var thumbnail = Center(
  Stack([
    Box({
      width: 160,
      height: 100,
      radius: 6,
      color: Color.system('control_background'),
      child: Center(Icon('film', { size: 32, color: Color.system('gray') })),
    }),
    Positioned(
      Box({
        padding: EdgeInsets.symmetric({ horizontal: 5, vertical: 1 }),
        radius: 4,
        color: Color.BLACK.with_alpha(0.6),
        child: Text('4:12', {
          style: TextStyle({ size: 11, color: Color.WHITE }),
        }),
      }),
      { right: 6, bottom: 6 },
    ),
  ]),
)

run_app(
  App({
    windows: [
      Window('Videos', thumbnail, { size: Size(300, 180) }),
    ],
  }),
)
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `alignment` | `Alignment` | `Alignment.TOP_LEFT` | Where children that aren't positioned sit. |
| `fit` | `string` | `StackFit.LOOSE` | How big children that aren't positioned may be. `LOOSE` lets them be their natural size, `EXPAND` makes them fill the stack, and `PASSTHROUGH` gives them the same limits the stack was given. |

`Positioned` takes any of `left`, `top`, `right` and `bottom`, the
distance from that edge of the stack, and `width` and `height`. Give it
both `left` and `right` and the child stretches between them.
`Positioned.fill(child)` covers the whole stack, which suits a
translucent overlay.

## Wrap

`Wrap(children, options)` lays children along a line and starts a new
line when it runs out of room, the way words fill a paragraph. Use it
for tags, filters and toolbars that have to fit a narrow window.

```zuri
import zug { * }

class Chip < StatelessWidget {
  var label

  @new(label: string) {
    self.label = label
  }

  build(context) {
    return Box({
      padding: EdgeInsets.symmetric({ horizontal: 10, vertical: 3 }),
      radius: 10,
      color: Color.system('accent').with_alpha(0.15),
      child: Text(self.label),
    })
  }
}

var tags = ['design', 'research', 'urgent', 'q4', 'customers', 'mobile']

var cloud = Padding(
  EdgeInsets.all(16),
  Wrap(tags.map(@(tag) => Chip(tag)), { spacing: 6, run_spacing: 6 }),
)

run_app(
  App({
    windows: [
      Window('Tags', cloud, { size: Size(260, 120) }),
    ],
  }),
)
```

Resize the window and the chips move onto as many lines as they need.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `axis` | `string` | `Axis.HORIZONTAL` | The direction of each line. With `Axis.VERTICAL`, children fill columns from top to bottom instead. |
| `spacing` | `number` | `0` | The space between children on a line. |
| `run_spacing` | `number` | `0` | The space between lines. |
| `justify` | `string` | `Justify.START` | How children spread along each line. Takes the same values as a row's `justify`. |
| `run_justify` | `string` | `Justify.START` | How the lines spread across the wrap, when it has room to spare. |
| `align_items` | `string` | `AlignItems.START` | Where each child sits across its line. |

## Grid and GridCell

`Grid(children, options)` puts children in equal columns, filling each
row from the left before it starts the next. Every child is stretched
to fill its cell, which makes a grid the natural fit for keypads,
palettes and photo walls.

```zuri
import zug { * }

def key(label) {
  return Button(label, {
    variant: ButtonVariant.FLEXIBLE,
    on_pressed: @{
      echo label
    },
  })
}

var keypad = Padding(
  EdgeInsets.all(12),
  Grid(
    [
      key('7'),
      key('8'),
      key('9'),
      key('4'),
      key('5'),
      key('6'),
      key('1'),
      key('2'),
      key('3'),
      GridCell(key('0'), { column_span: 2 }),
      key('.'),
    ],
    { columns: 3, spacing: 6, row_height: 40 },
  ),
)

run_app(
  App({
    windows: [
      Window('Keypad', keypad, { size: Size(200, 200) }),
    ],
  }),
)
```

`GridCell(child, options)` makes a child span several cells. It takes
`column_span` and `row_span`, both `1` when left out. A child that spans
cells takes the first place where it fits.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `columns` | `number` | required | How many columns the grid has. |
| `spacing` | `number` | `0` | The space between cells, across and down. |
| `column_spacing` | `number` | `spacing` | The space between columns, when it differs. |
| `row_spacing` | `number` | `spacing` | The space between rows, when it differs. |
| `row_height` | `number` | none | Makes every row this tall. |
| `aspect_ratio` | `number` | none | Makes every row as tall as a cell's width divided by this, so `1` gives square cells. |

Without `row_height` or `aspect_ratio`, each row is as tall as its
tallest child.

## Table

`Table(rows, options)` lines up rows of widgets in columns, and sizes
each column by its own rule. It's the widget for forms, where labels
sit in one column and fields in the next.

```zuri
import zug { * }

var form = Padding(
  EdgeInsets.all(20),
  Table(
    [
      [
        Text('Name'),
        TextField({ placeholder: 'Ada Lovelace' }),
      ],
      [
        Text('Email'),
        TextField({ placeholder: 'ada@example.com' }),
      ],
      [
        Text('Team'),
        Dropdown(
          ['Analytics', 'Engineering', 'Design'],
          'Analytics',
          { on_changed: @(team) {} },
        ),
      ],
    ],
    {
      columns: [ColumnWidth.fit(), ColumnWidth.flex()],
      align_items: AlignItems.BASELINE,
      column_spacing: 8,
      row_spacing: 8,
    },
  ),
)

run_app(
  App({
    windows: [
      Window('New member', form, { size: Size(380, 150) }),
    ],
  }),
)
```

Each row is a plain list of widgets. A row shorter than the others
leaves its last cells empty. Every cell is as wide as its column, and
each row is as tall as its tallest cell.

The column rules are:

| Rule | Width |
| --- | --- |
| `ColumnWidth.fixed(120)` | Exactly that many points. |
| `ColumnWidth.flex()` | A share of the width left after the other columns. `flex(2)` takes twice the share. |
| `ColumnWidth.fit()` | As wide as the widest cell in the column. |
| `ColumnWidth.fraction(0.3)` | That fraction of the table's width. |

Columns past the end of `columns` are `flex()`. When the table is too
narrow for its columns, the `fit()` columns give up width first.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `columns` | `list` | every column `flex()` | The rule for each column, from the left. |
| `align_items` | `string` | `AlignItems.START` | Where cells sit within their row. `BASELINE` lines up the text of labels and fields. |
| `spacing` | `number` | `0` | The space between cells, across and down. |
| `column_spacing` | `number` | `spacing` | The space between columns, when it differs. |
| `row_spacing` | `number` | `spacing` | The space between rows, when it differs. |

A `Table` holds widgets and builds all of them. For rows of data, and
especially many rows, use a [`DataTable`](scrolling-and-lists.md#datatable)
instead.

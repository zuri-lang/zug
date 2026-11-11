# Rows and columns

[Components](index.md) › Rows and columns

Most layouts are rows and columns inside each other. This page covers
the widgets that build them. [Layout](../layout.md) explains how they
decide sizes, which is worth reading once you start nesting them.

## Row and Column

`Row(children, options)` lays its children out from left to right, and
`Column(children, options)` from top to bottom. They take the same
options.

```zuri
import zug { * }

class Message < StatelessWidget {
  var sender
  var preview
  var time

  @new(sender: string, preview: string, time: string) {
    self.sender = sender
    self.preview = preview
    self.time = time
  }

  build(context) {
    var muted = TextStyle({ color: Color.system('secondary_label') })

    return Row(
      [
        Icon('person.crop.circle.fill', {
          size: 28,
          color: Color.system('gray'),
        }),
        Expanded(
          Column(
            [
              Text(self.sender, {
                style: TextStyle({ weight: FontWeight.SEMIBOLD }),
              }),
              Text(self.preview, {
                style: muted,
                max_lines: 1,
                overflow: TextOverflow.ELLIPSIS,
              }),
            ],
            { spacing: 2, align_items: AlignItems.START },
          ),
        ),
        Text(self.time, { style: muted }),
      ],
      { spacing: 10, align_items: AlignItems.START },
    )
  }
}

var inbox = Padding(
  EdgeInsets.all(16),
  Column(
    [
      Message('Ada', 'The figures for the third quarter are in.', '09:41'),
      Message('Charles', 'Can we move Thursday’s review to Friday?', '08:15'),
      Message('Grace', 'Shipped. Thanks, everyone!', 'Yesterday'),
    ],
    { spacing: 14, align_items: AlignItems.STRETCH },
  ),
)

run_app(
  App({
    windows: [
      Window('Inbox', inbox, { size: Size(380, 200) }),
    ],
  }),
)
```

The direction a row or column lays children out in is its main axis,
and the other direction is its cross axis. `justify` spreads the
children along the main axis, and `align_items` places them across it.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `spacing` | `number` | `0` | The space between neighbouring children, in points. |
| `justify` | `string` | `Justify.START` | Where children go along the main axis when there's room to spare. See below. |
| `align_items` | `string` | `AlignItems.CENTER` | Where children sit across the main axis. See below. |
| `shrink_wrap` | `bool` | `false` | Takes only the room the children need along the main axis, instead of all the room it's given. |

| `justify` | Children go |
| --- | --- |
| `Justify.START` | Together at the start. |
| `Justify.END` | Together at the end. |
| `Justify.CENTER` | Together in the middle. |
| `Justify.SPACE_BETWEEN` | With the spare room shared out between them, and none at the ends. |
| `Justify.SPACE_AROUND` | With equal room around each, so the ends get half as much as the gaps. |
| `Justify.SPACE_EVENLY` | With equal room in every gap and at both ends. |

| `align_items` | Children sit |
| --- | --- |
| `AlignItems.START` | At the top of a row, or the left of a column. |
| `AlignItems.CENTER` | In the middle. |
| `AlignItems.END` | At the bottom of a row, or the right of a column. |
| `AlignItems.STRETCH` | Stretched to fill the row's height, or the column's width. |
| `AlignItems.BASELINE` | In a row, with the first lines of their text lined up. Use it for a label beside a field. |

A row or column takes all the room it's given along its main axis,
unless it has `shrink_wrap`. A column straight in a window is as tall as
the window, so `justify` can place its children. In a `Center`, a
column usually wants `shrink_wrap: true`, so that what's centred is the
children and not an invisible column the size of the window.

Children that need more room than there is overflow. zug prints one
warning that says by how much. See
[When layout goes wrong](../layout.md#when-layout-goes-wrong).

### Flex

`Flex(children, { axis })` is a row or a column, with the direction as
an option. Use it when the direction is chosen while the application
runs, such as a toolbar that can sit at the side or the top:

```zuri,ignore
Flex(tools, {
  axis: self.toolbar_at_side ? Axis.VERTICAL : Axis.HORIZONTAL,
  spacing: 4,
})
```

It takes every option `Row` takes, plus `axis`, which is
`Axis.HORIZONTAL` when left out.

## Expanded and Flexible

Children of a row or column are their natural size. To make one take
the room that's left over, wrap it:

- `Expanded(child, { flex })` takes a share of the spare room and fills
  it. A child with `flex: 2` gets twice the share of one with the
  default `flex: 1`.
- `Flexible(child, { flex })` can take its share, but stays smaller when
  it doesn't need it all.

```zuri
import zug { * }

var editor = Row([
  Expanded(
    Box({
      color: Color.system('control_background'),
      child: Center(Text('Sidebar')),
    }),
    { flex: 1 },
  ),
  Expanded(Center(Text('Document')), { flex: 3 }),
])

run_app(
  App({
    windows: [
      Window('Editor', editor, { size: Size(480, 240) }),
    ],
  }),
)
```

The sidebar gets a quarter of the width and the document the other
three quarters, however wide the window is.

`Flexible` is useful for text that should shrink when space runs out
but not stretch when there's plenty, such as a file name in a status
bar:

```zuri,ignore
Row(
  [
    Icon('doc'),
    Flexible(
      Text(self.path, { max_lines: 1, overflow: TextOverflow.ELLIPSIS }),
    ),
    Text('Saved'),
  ],
  { spacing: 6 },
)
```

An `Expanded` or `Flexible` must be a direct child of a `Row`, `Column`
or `Flex`. Inside a `ScrollView` there's no spare room to share, so an
`Expanded` there raises an error.

## Spacer and Gap

`Spacer({ flex })` is empty space that takes a share of the spare room,
like an empty `Expanded`. It pushes the children on either side of it
apart, which is how you put buttons at opposite ends of a row.

`Gap(extent)` is a fixed amount of empty space along the main axis. Use
it where one gap should differ from the row's `spacing`.

```zuri
import zug { * }

var footer = Padding(
  EdgeInsets.all(16),
  Row([
    Button('Reset', { on_pressed: @{} }),
    Spacer(),
    Button('Cancel', { role: ButtonRole.CANCEL, on_pressed: @{} }),
    Gap(4),
    Button('Apply', { role: ButtonRole.PRIMARY, on_pressed: @{} }),
  ]),
)

run_app(
  App({
    windows: [
      Window('Settings', footer, { size: Size(400, 70) }),
    ],
  }),
)
```

| Widget | Options |
| --- | --- |
| `Expanded(child, options)` | `flex` (`1`), the share of the spare room it takes. |
| `Flexible(child, options)` | `flex` (`1`). |
| `Spacer(options)` | `flex` (`1`). |
| `Gap(extent)` | None besides `key`. `extent` is the size in points. |

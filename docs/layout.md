# Layout

zug lays your widgets out itself, the same way on every platform, and
then puts each native view exactly where it decided. This page explains
the one rule behind it, walks through building a typical window, and
covers what to do when a layout doesn't come out the way you expected.

The widgets themselves are described in the component reference, under
[Rows and columns](components/rows-and-columns.md),
[Boxes, spacing and sizing](components/boxes.md) and
[Stacks, grids and tables](components/stacks-grids-and-tables.md).

## The rule

**Constraints go down, sizes come up, and the parent decides where each
child goes.**

A parent tells each child how small and how large it may be: a minimum
and a maximum width and height. The child picks a size within those
limits, and the parent then positions it. That's the whole system, and
nearly every layout puzzle is answered by asking what a widget was
allowed to be.

A few things follow from it:

- A window gives its content exactly the window's size. That's why a
  `Column` placed straight in a window fills it.
- A widget can't be bigger than its parent allows. A `Box` with
  `width: 400` inside a window 300 points wide is 300 points wide.
- A scroll view lets its content be as long as it likes, so content
  inside one that wants to fill the space has nothing to fill. That's
  the cause of the most common layout error, covered below.
- `Center` and `Align` let their child be any size up to their own,
  while `Expanded` and `AlignItems.STRETCH` make a child a particular
  size. When a widget won't grow or shrink, one of these is usually
  the reason.

## Building a window

Most windows are a column of rows. Here's a mail client's main window,
with a toolbar at the top, a sidebar and a message list in the middle,
and a status line at the bottom:

```zuri
import zug { * }

var muted = TextStyle({ size: 11, color: Color.system('secondary_label') })

var toolbar = Padding(
  EdgeInsets.all(8),
  Row(
    [
      Button('New Message', { on_pressed: @{} }),
      Button('Reply', { on_pressed: @{} }),
      Spacer(),
      Box({
        width: 180,
        child: SearchField({ placeholder: 'Search' }),
      }),
    ],
    { spacing: 8 },
  ),
)

var sidebar = Box({
  width: 160,
  color: Color.system('control_background'),
  padding: EdgeInsets.all(10),
  child: Column(
    [Text('Inbox'), Text('Drafts'), Text('Sent')],
    { spacing: 6, align_items: AlignItems.START },
  ),
})

var messages = ListView.builder(
  50,
  @(context, index) => Padding(
    EdgeInsets.symmetric({ horizontal: 10 }),
    Row([
      Text('Message ${index + 1}'),
      Spacer(),
      Text('09:41', { style: muted }),
    ]),
  ),
  { item_extent: 26 },
)

var status = Padding(
  EdgeInsets.symmetric({ horizontal: 10, vertical: 4 }),
  Text('50 messages', { style: muted }),
)

var window = Column(
  [
    toolbar,
    Separator(),
    Expanded(
      Row([sidebar, Expanded(messages)], { align_items: AlignItems.STRETCH }),
    ),
    Separator(),
    status,
  ],
  { align_items: AlignItems.STRETCH },
)

run_app(
  App({
    windows: [
      Window('Mail', window, { size: Size(640, 400) }),
    ],
  }),
)
```

Reading it from the outside in:

- The window's `Column` is as big as the window. `AlignItems.STRETCH`
  makes every row in it as wide as the window too.
- The toolbar, the separators and the status line are their natural
  height. The middle row is in an `Expanded`, so it takes all the
  height that's left, and grows and shrinks with the window.
- In the toolbar, `Spacer` takes the spare width, which pushes the
  search field to the right. A search field is 160 points wide on its
  own, and the `Box` makes it 180.
- In the middle row, the sidebar is a fixed 160 points wide, and the
  message list is in an `Expanded`, so it takes the rest.
  `AlignItems.STRETCH` makes both as tall as the row.
- The message list is a `ListView.builder`, which builds only the rows
  in view. A plain `Column` of messages would build all of them.

When you build a window of your own, start the same way: decide which
parts are a fixed size, which parts are their natural size, and which
parts take what's left. The fixed parts get a `Box`, and the parts that
take what's left get an `Expanded`.

## Choosing a widget

| To | Use |
| --- | --- |
| Put things side by side, or one above another | [`Row` and `Column`](components/rows-and-columns.md#row-and-column) |
| Let one child take the space that's left | [`Expanded`](components/rows-and-columns.md#expanded-and-flexible) |
| Push children to opposite ends | [`Spacer`](components/rows-and-columns.md#spacer-and-gap) |
| Fix a size, or add a background, border or corners | [`Box`](components/boxes.md#box) |
| Keep content off the edges | [`Padding`](components/boxes.md#padding) |
| Centre something, or put it in a corner | [`Center` and `Align`](components/boxes.md#align-and-center) |
| Lay something over something else | [`Stack` and `Positioned`](components/stacks-grids-and-tables.md#stack-and-positioned) |
| Flow children onto new lines as the window narrows | [`Wrap`](components/stacks-grids-and-tables.md#wrap) |
| Make a keypad or a grid of equal cells | [`Grid`](components/stacks-grids-and-tables.md#grid-and-gridcell) |
| Line up labels and fields in a form | [`Table`](components/stacks-grids-and-tables.md#table) |
| Keep a shape, such as 16 by 9 | [`AspectRatio`](components/boxes.md#aspectratio) |
| Give a column of buttons the width of the widest | [`FitContent`](components/boxes.md#fitcontent) |
| Change the layout when the window is narrow | [`Responsive`](components/boxes.md#responsive) |
| Scroll content that doesn't fit | [`ScrollView`](components/scrolling-and-lists.md#scrollview) |

## When layout goes wrong

**"a Column with Expanded or Flexible children was given an unbounded
height".** A `Column` inside a `ScrollView`, or anything else that
allows unlimited height, has an `Expanded` child. In a scroll view
there's no spare space to share. Give the column's children their
natural sizes, or move the `Expanded` part outside the scroll view.

**"a Row overflowed by 40 points".** The children need more room than
the row has. zug prints this once, rather than at every frame. Wrap the
child that can shrink, usually some text, in `Expanded` or `Flexible`,
switch to a `Wrap`, or put the row in a horizontal `ScrollView`.

**Text runs off the edge instead of wrapping.** A `Row` lets each child
be as wide as it likes, so a long `Text` in a row never needs to wrap.
Wrap the text in `Expanded`, and it takes the width that's left and
wraps within it.

**A field, slider or progress bar is the wrong width.** These are 160
points wide unless something sizes them. In a column with
`AlignItems.STRETCH` they're as wide as the column. In a row, give them
an `Expanded` to take the width that's left, or a `Box` with a width.

**A widget won't get smaller or bigger.** Look at what its parent
allowed. A `Center` or `Align` loosens the limits so the child can be
smaller, and `Expanded` or `AlignItems.STRETCH` force a size.

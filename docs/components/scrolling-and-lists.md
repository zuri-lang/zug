# Scrolling and lists

[Components](index.md) › Scrolling and lists

## ScrollView

`ScrollView(child, options)` scrolls its child when it doesn't fit. A
scroll view fills the space it's given, so in a column give it an
`Expanded`:

```zuri
import zug { * }

var terms = 'These terms explain what you can expect from us. ' * 40

var agreement = Padding(EdgeInsets.all(16), Column([
  Text('Terms of Service', {
    style: TextStyle({ size: 17, weight: FontWeight.SEMIBOLD }),
  }),
  Expanded(ScrollView(Padding(EdgeInsets.all(10), Text(terms)), {
    bordered: true,
  })),
  Row([
    Spacer(),
    Button('Decline', { role: ButtonRole.CANCEL, on_pressed: @{} }),
    Button('Agree', { role: ButtonRole.PRIMARY, on_pressed: @{} }),
  ], { spacing: 8 }),
], { spacing: 10, align_items: AlignItems.STRETCH }))

run_app(App({
  windows: [Window('Terms', agreement, { size: Size(420, 320) })],
}))
```

Content in a scroll view can be as long as it likes in the direction it
scrolls. That's what makes scrolling work, and it's also why an
`Expanded` inside a scroll view has nothing to fill. See
[When layout goes wrong](../layout.md#when-layout-goes-wrong).

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `axis` | `string` | `'vertical'` | Which way it scrolls: `'vertical'`, `'horizontal'` or `'both'`. |
| `controller` | `ScrollController` | none | Follows and moves the scroll position. See below. |
| `on_scroll` | `function` | none | Called with the offset, a `Point`, while the view scrolls. |
| `bordered` | `bool` | `false` | Draws a border around the view. |
| `background` | `Color` | none | Fills the view behind its content. |

### Scrolling from code

A `ScrollController` follows a scroll view's position and moves it.
`offset()` returns the point of the content at the view's top left, and
`scroll_to(x, y)` scrolls that point into the top left:

```zuri,ignore
class LogState < State {
  var scroller = ScrollController()

  dispose() {
    self.scroller.dispose()
  }

  build(context) {
    return Column([
      Button('Back to Top', { on_pressed: @{
        self.scroller.scroll_to(0, 0)
      } }),
      Expanded(ScrollView(LogLines(), { controller: self.scroller })),
    ])
  }
}
```

A controller is an `Observable`, so an [`Observer`](builders.md#observer)
of it rebuilds as the view scrolls, which suits a "Back to Top" button
that only appears once you've scrolled down.

## ListView

`ListView(children, options)` is a scrolling column of all its children.
It builds every one of them, so it's best for lists of up to a few
dozen rows, such as a settings page or a short list of accounts.

```zuri
import zug { * }

var accounts = ['Personal', 'Work', 'Family', 'Newsletters']

var list = ListView(accounts.map(@(name) => Row([
  Icon('tray.full'),
  Text(name),
], { spacing: 8 })), {
  spacing: 8,
  padding: EdgeInsets.all(12),
})

run_app(App({
  windows: [Window('Mailboxes', list, { size: Size(220, 160) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `spacing` | `number` | `0` | The space between rows. |
| `padding` | `EdgeInsets` | none | Space around the rows, inside the scrolling area. |
| `axis` | `string` | `'vertical'` | `'horizontal'` makes a row that scrolls sideways. |
| `controller` | `ScrollController` | none | Follows and moves the scroll position. |
| `on_scroll` | `function` | none | Called with the offset, a `Point`, while the list scrolls. |

Rows are stretched to the width of the list.

## ListView.builder

`ListView.builder(count, builder, options)` is a list of any length. It
builds only the rows in view, plus a few either side, and builds the
others as they scroll in. A list of a hundred thousand rows costs about
what a screenful does.

The `builder` is called with a context and a row index, and returns the
row. Every row is `item_extent` points tall, which is how the list
knows which rows are in view without building the others.

```zuri
import zug { * }

var muted = TextStyle({ color: Color.system('secondary_label') })

var log = ListView.builder(100000, @(context, index) => Padding(
  EdgeInsets.symmetric({ horizontal: 12 }),
  Row([
    Text('Request ${index + 1}'),
    Spacer(),
    Text('${(index * 37) % 900 + 20} ms', { style: muted }),
  ]),
), { item_extent: 24 })

run_app(App({
  windows: [Window('Requests', log, { size: Size(300, 320) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `item_extent` | `number` | required | The height of every row, or its width in a horizontal list. |
| `overscan` | `number` | `4` | How many rows to build beyond each end of the view, so they're ready when the user scrolls. |
| `axis` | `string` | `'vertical'` | `'horizontal'` makes a row that scrolls sideways. |
| `controller` | `ScrollController` | none | Follows and moves the scroll position. |
| `on_scroll` | `function` | none | Called with the offset, a `Point`, while the list scrolls. |

Rows that scroll out of view are thrown away, state and all. Keep
anything a row needs to remember, such as whether it's expanded, in the
data the rows are built from. [Performance](../performance.md#long-lists-and-tables)
has measurements of long lists.

## DataTable

`DataTable(columns, row_count, cell, options)` is the platform's native
table: rows and columns of text with a header, selection, sorting and
double click. It never builds widgets for its rows. Instead, it calls
your `cell` function with a row and a column id for each cell it's
about to draw, and shows the text it returns. That's why it handles any
number of rows.

```zuri
import zug { * }

var people = [
  { name: 'Ada Lovelace', age: 36, role: 'Analyst' },
  { name: 'Charles Babbage', age: 79, role: 'Engineer' },
  { name: 'Grace Hopper', age: 85, role: 'Admiral' },
  { name: 'Alan Turing', age: 41, role: 'Mathematician' },
]

class People < StatefulWidget {
  create_state() {
    return PeopleState()
  }
}

class PeopleState < State {
  var selected = []
  var sort = ['name', true]

  sorted() {
    var column = self.sort[0]
    var ascending = self.sort[1]
    var rows = people.clone()
    rows.sort(@(a, b) {
      var x = a[column]
      var y = b[column]
      var order = is_number(x) ? x - y : x.compare(y)
      return ascending ? order : -order
    })
    return rows
  }

  build(context) {
    var rows = self.sorted()
    var columns = [
      DataColumn('name', 'Name', { width: 160, sortable: true }),
      DataColumn('age', 'Age', {
        width: 50,
        align: TextAlign.END,
        sortable: true,
      }),
      DataColumn('role', 'Role'),
    ]

    var cell = @(row, column) => '${rows[row][column]}'

    return DataTable(columns, rows.length(), cell, {
      selection: Selection.MULTIPLE,
      selected: self.selected,
      sort: self.sort,
      on_selection_changed: @(chosen) {
        self.set_state(@{
          self.selected = chosen
        })
      },
      on_sort: @(column, ascending) {
        self.set_state(@{
          self.sort = [column, ascending]
          self.selected = []
        })
      },
      on_activate: @(row) {
        echo 'Opening ${rows[row].name}'
      },
    })
  }
}

run_app(App({
  windows: [Window('People', People(), { size: Size(420, 200) })],
}))
```

The table doesn't sort for you. When the user clicks a sortable header,
`on_sort` tells you which column and which direction. Sort your rows,
store the new `sort`, and rebuild, and the header shows the arrow where
you put it. Selection works the same way: the table shows the
`selected` rows you pass it.

Each `DataColumn(id, title, options)` has an `id`, which your `cell`
function receives, and a `title` for the header:

| Column option | Type | Default | Description |
| --- | --- | --- | --- |
| `width` | `number` | the table's choice | The column's starting width. The user can resize it. |
| `min_width`, `max_width` | `number` | none | Limits for resizing. |
| `align` | `string` | `TextAlign.START` | How the cells' text lines up. Numbers usually read best at the `END`. |
| `sortable` | `bool` | `false` | Lets the user click the header to sort by this column. |

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `selection` | `string` | `Selection.SINGLE` | How many rows can be selected: `NONE`, `SINGLE` or `MULTIPLE`. |
| `selected` | `list` | `[]` | The indices of the selected rows. |
| `on_selection_changed` | `function` | none | Called with the list of rows the user selected. |
| `sort` | `list` | none | `[column id, ascending]`, shown as the arrow in that column's header. |
| `on_sort` | `function` | none | Called with a column id and whether to sort ascending, when the user clicks a sortable header. |
| `on_activate` | `function` | none | Called with a row when the user double clicks it, or presses Return on it. |
| `alternating` | `bool` | `true` | Shades every other row. |
| `grid` | `bool` | `false` | Draws lines between the cells. |
| `header` | `bool` | `true` | Shows the header row. |

Every time the widget is built, the table reloads, so it always shows
what your `cell` function returns now. Keep `cell` quick: it runs for
every cell that comes into view.

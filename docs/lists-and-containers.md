# Scrolling, lists and tables

This page covers the widgets that hold more than fits: scroll views,
lists, tables, tabs and split views.

## Scroll views

`ScrollView(child)` scrolls its child when it doesn't fit. The scroll
view fills the space it's given, so inside a column give it an
`Expanded`:

```zuri
import zug { * }

var page = Column([
  Text('Release notes', { style: TextStyle({ size: 17, weight: FontWeight.SEMIBOLD }) }),
  Expanded(ScrollView(Padding(EdgeInsets.all(12), Text('A long text that needs room.')))),
], { spacing: 8, align_items: AlignItems.STRETCH })
```

It scrolls vertically unless you give it `axis: 'horizontal'` or
`'both'`. `bordered` draws a border round it and `background` fills it
with a colour. `on_scroll` is called with the offset as a `Point` while
it scrolls.

To scroll from code, pass a `ScrollController`. `offset()` says where it
is, and `scroll_to(x, y)` puts that point of the content at the top
left:

```zuri,ignore
var scroller = ScrollController()
ScrollView(content, { controller: scroller })
scroller.scroll_to(0, 0)
```

Content inside a scroll view can be as long as it likes along the
scrolling axis, which is also why an `Expanded` inside it has nothing to
fill. See [When layout goes wrong](layout.md#when-layout-goes-wrong).

## Lists

`ListView(children)` is a scrolling column of every child, with
`spacing` and `padding`. It's fine for a few dozen rows.

For anything longer, or anything whose length you don't control, use
`ListView.builder`. It builds only the rows in view, plus a few either
side, and builds the rest as they scroll in:

```zuri
import zug { * }

var log = ListView.builder(100000, @(context, index) => Row([
  Text('Entry ${index}'),
  Spacer(),
  Text('${index * 7 % 60} s', { style: TextStyle({ color: Color.system('secondary_label') }) }),
], { spacing: 8 }), { item_extent: 22 })
```

Every row is `item_extent` points tall, which is how the list knows
which rows are in view without building the others. `overscan` sets how
many extra rows it builds either side (4), and it also takes `axis`,
`controller` and `on_scroll`. A list of a hundred thousand rows costs
about what a screenful does; [Performance](performance.md) has the
numbers and the reasons.

## Tables

`DataTable` is the native table: rows and columns of text with a header,
selection, sorting and double-click. It never builds widgets for its
rows. It asks your `cell` function for the text of the cells it's about
to draw, so it handles any number of rows.

```zuri
import zug { * }

var people = [['Ada Lovelace', 36, 'Analyst'], ['Charles Babbage', 79, 'Engineer'], ['Grace Hopper', 85, 'Admiral']]

class People < StatefulWidget {
  create_state() {
    return PeopleState()
  }
}

class PeopleState < State {
  var selected = []
  var ascending = true

  build(context) {
    var rows = people.clone()
    rows.sort(@(a, b) => self.ascending ? a[0].compare(b[0]) : b[0].compare(a[0]))

    return DataTable([
      DataColumn('name', 'Name', { width: 160, sortable: true }),
      DataColumn('age', 'Age', { width: 50, align: TextAlign.END }),
      DataColumn('role', 'Role'),
    ], rows.length(), @(row, column) {
      var person = rows[row]
      return column == 'name' ? person[0] : column == 'age' ? person[1] : person[2]
    }, {
      selection: Selection.MULTIPLE,
      selected: self.selected,
      sort: ['name', self.ascending],
      on_selection_changed: @(chosen) {
        self.set_state(@{
          self.selected = chosen
        })
      },
      on_sort: @(column, ascending) {
        self.set_state(@{
          self.ascending = ascending
        })
      },
      on_activate: @(row) {
        echo 'open ${rows[row][0]}'
      },
    })
  }
}

run_app(App({ windows: [Window('People', People(), { size: Size(420, 200) })] }))
```

Each `DataColumn(id, title, options)` has an `id` your `cell` function
receives, a header `title`, and optionally `width`, `min_width`,
`max_width`, `align` and `sortable`. The table's options are:

| Option | What it does |
| --- | --- |
| `selection` | `Selection.SINGLE` (the default), `MULTIPLE` or `NONE`. |
| `selected` | The selected row indices, and `on_selection_changed` reports new ones, as with any control. |
| `sort` | `[column id, ascending]`, shown as the arrow in that column's header. |
| `on_sort` | Called with a column id and a direction when the user clicks a sortable header. The table doesn't sort for you; sort your rows and rebuild. |
| `on_activate` | Called with a row when it's double-clicked or Return is pressed on it. |
| `alternating`, `grid`, `header` | Alternate row colours (on), grid lines (off), the header (on). |

Whenever the widget is rebuilt the table reloads, so it always shows
what your `cell` function returns now.

## Tabs

`TabView(tabs, selected, options)` shows one of several `Tab(label,
content)`s, and `on_changed` reports the tab the user picked. The
content of every tab stays in the tree, so a tab keeps its state, its
scroll position and its half-typed text while you're on another:

```zuri
import zug { * }

var settings = TabView([
  Tab('General', Center(Text('General settings'))),
  Tab('Advanced', Center(Text('Advanced settings'))),
], 0, { on_changed: @(index) {} })
```

## Split views

`SplitView(panes, options)` puts panes side by side, or one above
another with `axis: Axis.VERTICAL`, with dividers the user can drag.
`initial` gives the starting sizes of the panes along the axis,
`divider` is `SplitDivider.THIN` (the default), `THICK` or `PANE`, and
`on_resize` is called with the pane sizes after a drag.

```zuri
import zug { * }

var browser = SplitView([
  VisualEffect(ListView.builder(200, @(context, index) => Text('Folder ${index}'), { item_extent: 22 })),
  Center(Text('Pick a folder')),
], { initial: [200] })
```

## Materials

`VisualEffect(child, options)` puts a translucent system material behind
its child, such as the frosted look of a sidebar. `material` is
`Material.SIDEBAR` (the default), `HEADER`, `CONTENT`, `WINDOW`, `MENU`,
`POPOVER`, `HUD` or `UNDER_WINDOW`, and `blending` is `behind` the window
or `within` it. On a platform without materials it's a plain container.

# Containers

[Components](index.md) › Containers

## TabView

`TabView(tabs, selected, options)` shows one of several pages, with a
row of tabs to switch between them. Each page is a `Tab(label, child)`,
and `selected` is the index of the one showing.

```zuri
import zug { * }

class Settings < StatefulWidget {
  create_state() {
    return SettingsState()
  }
}

class SettingsState < State {
  var tab = 0
  var autosave = true

  build(context) {
    var general = Padding(EdgeInsets.all(16), Column([
      Checkbox(self.autosave, {
        label: 'Save documents automatically',
        on_changed: @(on) {
          self.set_state(@{
            self.autosave = on
          })
        },
      }),
    ], { align_items: AlignItems.START }))

    var accounts = Center(Text('No accounts yet'))

    return Padding(EdgeInsets.all(12), TabView([
      Tab('General', general),
      Tab('Accounts', accounts),
    ], self.tab, {
      on_changed: @(index) {
        self.set_state(@{
          self.tab = index
        })
      },
    }))
  }
}

run_app(App({
  windows: [Window('Settings', Settings(), { size: Size(420, 240) })],
}))
```

Every tab's content stays in the tree while another tab is showing, so
each page keeps its state, its scroll position and anything half typed
into it.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `on_changed` | `function` | none | Called with the index of the tab the user picked. |

## SplitView

`SplitView(panes, options)` puts panes side by side, with dividers the
user can drag to resize them. It's how you build a sidebar, or an
editor with a preview beside it.

```zuri
import zug { * }

var folders = ListView.builder(40, @(context, index) => Padding(
  EdgeInsets.symmetric({ horizontal: 12 }),
  Row([Icon('folder'), Text('Folder ${index + 1}')], { spacing: 6 }),
), { item_extent: 24 })

var browser = SplitView([
  VisualEffect(folders),
  Center(Text('Choose a folder')),
], {
  initial: [200],
  on_resize: @(sizes) {
    echo 'sidebar is now ${sizes[0]} points wide'
  },
})

run_app(App({
  windows: [Window('Files', browser, { size: Size(600, 360) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `axis` | `string` | `Axis.HORIZONTAL` | Panes side by side, or one above another with `Axis.VERTICAL`. |
| `initial` | `list` | the platform's choice | The starting sizes of the panes along the axis. The last pane takes what's left, so `[200]` gives a two pane split a sidebar 200 points wide. |
| `divider` | `string` | `SplitDivider.THIN` | The divider's style: `THIN`, `THICK`, or `PANE` for the style with a grip. |
| `on_resize` | `function` | none | Called with the list of pane sizes after the user drags a divider. |

`initial` applies only when the split view first appears, and after
that the user decides. To remember their sizes between launches, save
what `on_resize` reports and pass it back as `initial` next time.

## GroupBox

`GroupBox(child, options)` draws the standard box around a set of
related controls, with an optional title above it.

```zuri
import zug { * }

class Export < StatefulWidget {
  create_state() {
    return ExportState()
  }
}

class ExportState < State {
  var format = 'PNG'
  var transparent = false

  build(context) {
    return Padding(EdgeInsets.all(16), GroupBox(Column([
      RadioGroup(['PNG', 'JPEG', 'PDF'], self.format, {
        axis: Axis.HORIZONTAL,
        on_changed: @(format) {
          self.set_state(@{
            self.format = format
          })
        },
      }),
      Checkbox(self.transparent, {
        label: 'Transparent background',
        enabled: self.format == 'PNG',
        on_changed: @(on) {
          self.set_state(@{
            self.transparent = on
          })
        },
      }),
    ], { spacing: 10, align_items: AlignItems.START }), { title: 'Format' }))
  }
}

run_app(App({
  windows: [Window('Export', Export(), { size: Size(340, 140) })],
}))
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `title` | `string` | none | The title shown above the box. |

The child sits inside the border and below the title, with the
platform's usual margin around it.

## Separator

`Separator(options)` is a thin line between groups of content. In a
column it runs across, and in a row it runs down.

```zuri
import zug { * }

var toolbar = Padding(EdgeInsets.all(12), Row([
  Button('Cut', { on_pressed: @{} }),
  Button('Copy', { on_pressed: @{} }),
  Button('Paste', { on_pressed: @{} }),
  Box({ height: 20, child: Separator() }),
  Button('Undo', { on_pressed: @{} }),
], { spacing: 8 }))

var page = Column([
  toolbar,
  Separator(),
  Expanded(Center(Text('Document'))),
], { align_items: AlignItems.STRETCH })

run_app(App({
  windows: [Window('Editor', page, { size: Size(420, 200) })],
}))
```

A row only stretches its children with `AlignItems.STRETCH`, so a
separator in a row is as short as it can be unless you give it a
height, as the toolbar does with a `Box`.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `axis` | `string` | decided by the parent | Fixes the direction: `Axis.HORIZONTAL` for a line across, `Axis.VERTICAL` for a line down. Use it outside a row or column. |

## VisualEffect

`VisualEffect(child, options)` puts a translucent system material
behind its child, such as the frosted background of a sidebar. Content
behind the window shows through, softly blurred, the way it does in
Finder and Mail. The `SplitView` example above uses one for its sidebar.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `material` | `string` | `Material.SIDEBAR` | Which material: `SIDEBAR`, `HEADER`, `CONTENT`, `WINDOW`, `MENU`, `POPOVER`, `HUD` or `UNDER_WINDOW`. |
| `blending` | `string` | `'behind'` | `'behind'` blurs what's behind the window. `'within'` blurs what's behind the view inside the window, for a bar over scrolling content. |

On a platform without materials, a `VisualEffect` is a plain container.

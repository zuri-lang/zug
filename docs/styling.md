# Styling

zug leans on the platform for its looks: controls are drawn by AppKit,
colours follow the system's light and dark appearance, and text uses the
system font. This page covers what you can change on top of that: text
styles, colours, a style sheet for the whole application, and the
appearance.

## Text styles

A `TextStyle` describes how text looks:

```zuri
import zug { * }

var price = TextStyle({
  size: 15,
  weight: FontWeight.SEMIBOLD,
  monospaced_digits: true,
  color: Color.system('label'),
})
var note = price.copy_with({ size: 11, weight: FontWeight.REGULAR, italic: true })
```

| Option | What it does |
| --- | --- |
| `size` | in points; 13 is the system's usual size |
| `weight` | `FontWeight.ULTRA_LIGHT` (100) through `REGULAR` (400), `SEMIBOLD` (600), `BOLD` (700) to `BLACK` (900) |
| `design` | `FontDesign.DEFAULT`, `MONOSPACED`, `ROUNDED` or `SERIF`, the system font's variants |
| `family` | a font family by name, such as `Menlo` |
| `italic`, `underline`, `strikethrough` | as named |
| `monospaced_digits` | digits of equal width, so changing numbers don't jiggle |
| `color` | a `Color`; the label colour when left out |

`copy_with(options)` makes a changed copy, and `a.merge(b)` lays `b`
over `a`, keeping what `b` leaves out.

## Colours

```zuri
import zug { * }

var brand = Color.hex('#0a84ff')
var shade = Color.rgba(0, 0, 0, 0.25)
var muted = brand.with_alpha(0.5)
var text = Color.system('secondary_label')
```

- `Color.rgb(r, g, b)` and `Color.rgba(r, g, b, alpha)` take channels
  from 0 to 255 and alpha from 0 to 1; `Color.gray(level, alpha)` makes a
  grey from 0 for black to 1 for white; `Color.hex()` reads `#rgb`,
  `#rgba`, `#rrggbb` or `#rrggbbaa`, and `to_hex()` writes one back.
- `Color.system(name)` is one of the platform's semantic colours, which
  change with the appearance and the user's accent colour. Prefer these
  for anything that should look right in dark mode. The names include
  `label`, `secondary_label`, `tertiary_label`, `text`,
  `placeholder_text`, `window_background`, `control_background`,
  `control`, `control_text`, `selected_content_background`,
  `separator`, `grid`, `link`, `accent`, and the system's own `red`,
  `orange`, `yellow`, `green`, `mint`, `teal`, `cyan`, `blue`,
  `indigo`, `purple`, `pink`, `brown` and `gray`.
- `Color.BLACK`, `WHITE`, `TRANSPARENT`, `RED`, `GREEN`, `BLUE`,
  `YELLOW`, `ORANGE` and `GRAY` are fixed colours for when you want one
  that doesn't change.

## Borders and shadows

`Border.all(width, color)` and `Shadow({ color, blur, x, y })` decorate a
`Box`, together with its `color`, `radius` and `opacity`; see
[Layout](layout.md#spacing-alignment-and-boxes). A shadow is a
translucent black, 4 points of blur and 2 points down unless you say
otherwise.

## A style sheet for the application

Most applications want the same few text styles everywhere. Put them in
a `StyleSheet` and give it to the `App`, or to a `Style` around any part
of the tree:

```zuri
import zug { * }

var sheet = StyleSheet({
  accent: Color.hex('#ff9500'),
  text: TextStyle({ size: 14 }),
  headline: TextStyle({ size: 26, weight: FontWeight.BOLD }),
})

class Welcome < StatelessWidget {
  build(context) {
    var style = Style.of(context)
    return Padding(style.padding, Column([
      Text('Welcome', { style: style.headline }),
      Text('Every Text below the sheet starts from its text style.'),
    ], { spacing: style.spacing, align_items: AlignItems.START }))
  }
}

run_app(App({ style: sheet, windows: [Window('Welcome', Welcome())] }))
```

A sheet holds `accent`, the text styles `text`, `title`, `headline`,
`caption` and `mono`, and the usual `spacing` (8) and `padding` (16 on
every side). Whatever you leave out takes the platform's usual look.
`Style.of(context)` reads the nearest sheet, and every `Text` merges its
own style over the sheet's `text` style, so changing the sheet changes
the whole application. `sheet.copy_with(options)` makes a variant for one
part of the tree.

## Light and dark

`Appearance.of(context)` is `'light'` or `'dark'`, and the widgets that
ask are rebuilt when the user switches. You rarely need it, because
system colours already follow the appearance, but it's there for
anything you draw yourself:

```zuri,ignore
Canvas(@(painter, size) {
  var ink = Appearance.of(context) == 'dark' ? Color.WHITE : Color.BLACK
  painter.line(Point(0, 0), Point(size.width, size.height), Paint({ color: ink }))
}, { repaint: Appearance.of(context) })
```

An application follows the system's appearance unless it chooses one.
`App`'s `appearance` option is `Appearance.LIGHT`, `Appearance.DARK` or
`Appearance.SYSTEM`, the default. It's an option like any other, so to
let people pick, keep their choice in state and build the `App` with it:

```zuri,ignore
var LOOKS = [Appearance.SYSTEM, Appearance.LIGHT, Appearance.DARK]

class Editor < StatefulWidget {
  create_state() {
    return EditorState()
  }
}

class EditorState < State {
  var look = Appearance.SYSTEM

  build(context) {
    var picker = SegmentedControl(['System', 'Light', 'Dark'], LOOKS.index_of(self.look), {
      on_changed: @(index) {
        self.set_state(@{
          self.look = LOOKS[index]
        })
      },
    })

    return App({
      title: 'Editor',
      appearance: self.look,
      windows: [Window('Editor', Center(picker))],
    })
  }
}
```

The whole application switches at once, open windows, sheets and panels
included, and `Appearance.of(context)` reports the appearance shown.

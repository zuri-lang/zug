# Text and images

[Components](index.md) › Text and images

## Text

`Text(text, options)` shows a run of text in one style. It's the label
next to a field, the heading of a window and the paragraph in a sheet.

```zuri
import zug { * }

class ReleaseNotes < StatelessWidget {
  build(context) {
    var style = Style.of(context)

    return Padding(EdgeInsets.all(20), Column([
      Text('What’s new in 2.4', { style: style.headline }),
      Text('Released on 11 November', {
        style: TextStyle({ color: Color.system('secondary_label') }),
      }),
      Text(
        'Large files open twice as fast, and printing on A5 paper is fixed.',
        { max_lines: 2, overflow: TextOverflow.ELLIPSIS, selectable: true },
      ),
    ], { spacing: 6, align_items: AlignItems.START }))
  }
}

run_app(App({
  windows: [Window('Release notes', ReleaseNotes(), { size: Size(360, 150) })],
}))
```

Text wraps at the width it's given, so in a window or a column it
breaks into lines on its own. A `Row` lets each child be as wide as it
likes, so a long `Text` in a row doesn't wrap. Wrap it in
[`Expanded`](rows-and-columns.md#expanded-and-flexible) and it takes the
width that's left and wraps within it.

Each `Text` starts from the `text` style of the nearest
[style sheet](../styling.md#a-style-sheet-for-the-application) and lays
its own `style` over it. Setting only a colour, as the date above does,
keeps the sheet's font and size.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `style` | `TextStyle` | the sheet's text style | How the text looks. See [Text styles](../styling.md#text-styles). |
| `align` | `string` | `TextAlign.START` | How lines line up: `START`, `CENTER`, `END` or `JUSTIFY`. |
| `max_lines` | `number` | `0` | The most lines to show. `0` shows them all. |
| `overflow` | `string` | `TextOverflow.CLIP` | What happens to text past the last line: `CLIP` cuts it off, `ELLIPSIS` ends the line with “…”. |
| `selectable` | `bool` | `false` | Lets the user select the text and copy it. |

Selectable text takes keyboard focus when it's clicked, but only keeps
the keys that move the selection. Everything else, such as a digit or
Return, still reaches the window's
[shortcuts](../apps-windows-and-menus.md#the-keyboard).

## Image

`Image(source, options)` shows a picture. The `source` says where it
comes from:

| Source | Shows |
| --- | --- |
| `ImageSource.file(path)` | An image file, in any format the platform reads, such as PNG, JPEG, HEIC or PDF. |
| `ImageSource.memory(bytes)` | An encoded image you already hold in memory, such as a PNG you downloaded. |
| `ImageSource.symbol(name, { size, weight })` | A symbol from the system's icon set. On macOS these are SF Symbols, such as `photo` or `folder.fill`. |

This example shows a photo with buttons to turn and flip it:

```zuri
import zug { * }

class Viewer < StatefulWidget {
  create_state() {
    return ViewerState()
  }
}

class ViewerState < State {
  var rotation = 0
  var mirrored = false

  build(context) {
    var photo = Image(ImageSource.file('photo.jpg'), {
      fit: ImageFit.CONTAIN,
      rotation: self.rotation,
      flip_horizontal: self.mirrored,
    })

    var tools = Row([
      Button('Turn Left', { on_pressed: @{
        self.set_state(@{
          self.rotation = (self.rotation + 270) % 360
        })
      } }),
      Button('Turn Right', { on_pressed: @{
        self.set_state(@{
          self.rotation = (self.rotation + 90) % 360
        })
      } }),
      Button('Mirror', { on_pressed: @{
        self.set_state(@{
          self.mirrored = !self.mirrored
        })
      } }),
    ], { spacing: 8 })

    return Padding(EdgeInsets.all(16), Column([
      Expanded(photo),
      tools,
    ], { spacing: 12 }))
  }
}

run_app(App({
  windows: [Window('Photo', Viewer(), { size: Size(480, 400) })],
}))
```

Without `width` or `height`, an image is the size of the picture
itself. Give it a size, or put it somewhere that sizes it, like the
`Expanded` above, and `fit` decides how the picture fills that space.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `width`, `height` | `number` | the picture's size | The size of the image's box, in points. |
| `fit` | `string` | `ImageFit.CONTAIN` | How the picture fills its box. `CONTAIN` makes it as large as fits while keeping its shape, `FILL` stretches it to fill the box, `NONE` shows it at its own size in the middle, and `SCALE_DOWN` shrinks a large picture to fit but never enlarges a small one. |
| `tint` | `Color` | none | Colours a template image, such as a symbol, in one colour. |
| `rotation` | `number` | `0` | Turns the picture clockwise by quarter turns: `90`, `180` or `270`. Negative turns go the other way, so `-90` is the same as `270`. |
| `flip_horizontal` | `bool` | `false` | Mirrors the picture left to right. |
| `flip_vertical` | `bool` | `false` | Mirrors the picture top to bottom. |

When an image without a size is turned a quarter, its size turns with
it, so a landscape photo becomes a tall one. Flips apply to the picture
before it's turned.

A rotation that isn't a multiple of 90 raises a `ValueError`. For any
other angle, draw the image on a [`Canvas`](drawing-and-input.md#canvas)
with `painter.rotate()`.

## Icon

`Icon(name, options)` shows a symbol from the system's icon set, sized
and coloured like text. Use it beside labels, in rows of buttons and in
lists.

```zuri
import zug { * }

var folders = Column([
  Row([
    Icon('folder.fill', { color: Color.system('blue') }),
    Text('Projects'),
  ], { spacing: 6 }),
  Row([
    Icon('star.fill', { color: Color.system('yellow') }),
    Text('Favourites'),
  ], { spacing: 6 }),
  Row([
    Icon('trash', { color: Color.system('secondary_label') }),
    Text('Trash'),
  ], { spacing: 6 }),
], { spacing: 8, align_items: AlignItems.START })

run_app(App({
  windows: [
    Window('Places', Padding(EdgeInsets.all(16), folders), {
      size: Size(220, 120),
    }),
  ],
}))
```

On macOS the names are SF Symbol names. Apple's SF Symbols app lists
them all and shows what each looks like.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `size` | `number` | `13` | The symbol's size in points, matched to text of that size. |
| `weight` | `number` | `400` | The stroke weight, from `100` (thin) to `900` (black), like a font weight. `FontWeight` has names for these. |
| `color` | `Color` | the label colour | The symbol's colour. |

`Icon` is a shorter way of writing an `Image` of
`ImageSource.symbol(name)` with `ImageFit.NONE` and a tint.

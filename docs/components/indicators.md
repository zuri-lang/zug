# Progress and levels

[Components](index.md) › Progress and levels

## ProgressBar

`ProgressBar(options)` shows how far a task has got. Give it a `value`
between `min` and `max`, 0 and 1 unless you say otherwise.

Leave `value` out when you can't tell how much is left. The bar then
moves on its own to show that the task is still going.

```zuri
import zug { * }

class Upload < StatefulWidget {
  create_state() {
    return UploadState()
  }
}

class UploadState < State {
  var sent = 0
  var timer

  init_state() {
    self.timer = Timer.periodic(Duration.ms(200), @(timer) {
      self.set_state(@{
        self.sent = (self.sent + 0.05).min(1)
      })

      if self.sent >= 1 {
        timer.cancel()
      }
    })
  }

  dispose() {
    self.timer.cancel()
  }

  build(context) {
    var done = self.sent >= 1

    return Padding(
      EdgeInsets.all(20),
      Column(
        [
          Text(done ? 'Uploaded.' : 'Uploading photos…'),
          ProgressBar({ value: self.sent }),
          Text('${(self.sent * 100).round()}% sent', {
            style: TextStyle({ color: Color.system('secondary_label') }),
          }),
        ],
        { spacing: 8, align_items: AlignItems.STRETCH },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Upload', Upload(), { size: Size(320, 110) }),
    ],
  }),
)
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `value` | `number` | none | How far the task has got. Leave it out for a bar that shows activity without a measure. |
| `min` | `number` | `0` | The value of an empty bar. |
| `max` | `number` | `1` | The value of a full bar. |

In a row, a progress bar is 160 points wide unless you give it an
`Expanded` or a `Box` with a width. In a column that stretches its
children, as above, it's as wide as the column.

## Spinner

`Spinner(options)` is the small spinning indicator that says something
is happening. Use it for short waits where a bar would only flicker,
such as loading a page of results.

```zuri
import zug { * }

var loading = Row(
  [
    Spinner({ size: ControlSize.SMALL }),
    Text('Checking for updates…'),
  ],
  { spacing: 8, shrink_wrap: true },
)

run_app(
  App({
    windows: [
      Window('Updates', Center(loading), { size: Size(260, 80) }),
    ],
  }),
)
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `animating` | `bool` | `true` | Whether it spins. A spinner that isn't spinning is hidden. |
| `size` | `string` | `ControlSize.REGULAR` | `MINI`, `SMALL`, `REGULAR` or `LARGE`. |

To show a spinner only while something loads, build it only then, or
see [`PromiseBuilder`](builders.md#promisebuilder), which builds from a
promise as it settles.

## LevelIndicator

`LevelIndicator(value, options)` shows a level within a range: how full
a disk is, how well a search result matches, or a rating out of five
stars.

```zuri
import zug { * }

class Album < StatefulWidget {
  create_state() {
    return AlbumState()
  }
}

class AlbumState < State {
  var stars = 3

  build(context) {
    return Padding(
      EdgeInsets.all(20),
      Table(
        [
          [
            Text('Storage'),
            LevelIndicator(412, {
              max: 512,
              warning: 400,
              critical: 480,
              style: LevelStyle.CONTINUOUS,
            }),
          ],
          [
            Text('Rating'),
            LevelIndicator(self.stars, {
              style: LevelStyle.RATING,
              accessibility_label: 'Rating',
              on_changed: @(value) {
                self.set_state(@{
                  self.stars = value
                })
              },
            }),
          ],
        ],
        {
          columns: [ColumnWidth.fit(), ColumnWidth.flex()],
          column_spacing: 12,
          row_spacing: 10,
          align_items: AlignItems.CENTER,
        },
      ),
    )
  }
}

run_app(
  App({
    windows: [
      Window('Album', Album(), { size: Size(320, 100) }),
    ],
  }),
)
```

| Style | Draws |
| --- | --- |
| `LevelStyle.CONTINUOUS` | A smooth bar, like a capacity gauge. This is the default. |
| `LevelStyle.DISCRETE` | A bar of separate segments, one for each whole number. |
| `LevelStyle.RELEVANCY` | Thin lines that show how well something matches, as in a list of search results. |
| `LevelStyle.RATING` | Stars. |

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `min` | `number` | `0` | The lowest level. |
| `max` | `number` | `5` | The highest level. |
| `style` | `string` | `LevelStyle.CONTINUOUS` | How the level is drawn, from the table above. |
| `warning` | `number` | none | Past this level the bar is drawn in the warning colour. |
| `critical` | `number` | none | Past this level the bar is drawn in the critical colour. |
| `on_changed` | `function` | none | Lets the user click to change the level, and is called with the new one. Without it the indicator only shows the level. |

When low is the bad end, as with a battery, give `warning` a higher
value than `critical`. The colours then apply below those levels
instead of above them.

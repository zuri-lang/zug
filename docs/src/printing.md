# Printing

You print by drawing pages. zug hands your paint function a `Painter`,
the same one a [`Canvas`](drawing-and-pointer.md) draws with, once for
each page, and the platform's print dialog does the rest: choosing a
printer, copies and page ranges, a preview, and on macOS saving as a
PDF.

## Printing pages

```zuri,ignore
print_pages(context, {
  title: 'Invoice 1024',
  pages: 1,
  paint: @(painter, page) {
    painter.text(
      'Invoice 1024',
      Point(0, 0),
      TextStyle({ size: 24, weight: FontWeight.BOLD }),
    )
    painter.text('Due by 30 November.', Point(0, 40), TextStyle({ size: 12 }))
  },
}).then(@(printed) {
  self.set_state(@{
    self.status = printed ? 'Sent to the printer.' : 'Not printed.'
  })
})
```

`print_pages(context, options)` shows the print dialog, as a sheet on
the window of `context` on macOS, or on its own when you pass `nil`. The
promise resolves with `true` once the pages were printed or saved, and
`false` if the user cancelled.

`paint(painter, page)` draws one page. Pages count from 0, and the
origin is the top left of the area inside the margins, so you never
deal with the paper's edges. `painter.size()` is the size of that area.

The print dialog's preview calls `paint` too, sometimes more than once
for the same page, so it should only draw: work out what goes on each
page beforehand, or in a way that gives the same answer every time.

`title` names the job in the print queue, and becomes the file name when
the user saves a PDF.

Pages print as they look in light mode, whatever the appearance, so text
you draw without a colour comes out black.

## Pages that depend on the paper

A fixed number of pages works for a form or a label. Text that flows
from page to page doesn't fit on a fixed number: it depends on the paper
the user picks, and they can change that in the print dialog. So
`pages` can be a function. It's called with a painter of the printable
size, which you measure text with, and returns how many pages there
are. It's called again if the user picks other paper.

`text()` takes a `max_width` that wraps the text, and `measure_text()`
takes the same, to tell you how tall the wrapped text is. Between them
you can lay paragraphs out down the page and start a new page when the
next one doesn't fit:

```zuri,ignore
var body = TextStyle({ size: 12 })

# Each page, as a list of [paragraph, y] pairs.
def lay_out(paragraphs, painter) {
  var width = painter.size().width
  var pages = [[]]
  var y = 0

  for text in paragraphs {
    var height = painter.measure_text(text, body, width).height

    if y + height > painter.size().height and !pages[-1].is_empty() {
      pages.append([])
      y = 0
    }

    pages[-1].append([text, y])
    y += height + 8
  }

  return pages
}

print_pages(context, {
  title: 'Notes',
  pages: @(painter) => lay_out(self.paragraphs, painter).length(),
  paint: @(painter, page) {
    for item in lay_out(self.paragraphs, painter)[page] {
      painter.text(item[0], Point(0, item[1]), body, painter.size().width)
    }
  },
})
```

A paragraph taller than a whole page would need splitting line by line,
which this leaves out to stay short.

## Paper, orientation and margins

A `PageSetup` describes the page: `paper_size`, upright, `orientation`
(`PageOrientation.PORTRAIT` or `LANDSCAPE`) and `margins`, an
`EdgeInsets` of the space kept clear on each side. `page_size()` is the
page as it's printed, turned for landscape, and `printable_size()` is
what's left inside the margins.

```zuri
import zug { PageSetup, PaperSize, PageOrientation, EdgeInsets }

var setup = PageSetup({
  paper_size: PaperSize.LETTER,
  orientation: PageOrientation.LANDSCAPE,
  margins: EdgeInsets.all(36),
})

echo setup.printable_size().width
echo setup.printable_size().height
```

```console
720
540
```

`PaperSize` has `A3`, `A4`, `A5`, `LETTER` and `LEGAL`, in points of
1/72 inch. `default_page_setup()` gives what the system starts from:
the paper chosen in its settings and its usual margins.

`page_setup(context, { setup })` shows the platform's page setup dialog,
where the user picks the paper and which way round it goes. It resolves
with the `PageSetup` they chose, or `nil` if they cancelled. Keep it and
pass it to `print_pages()` as `setup`, so the next print starts where
they left off:

```zuri,ignore
page_setup(context, { setup: self.setup }).then(@(setup) {
  if setup != nil {
    self.setup = setup
  }
})
```

Leave `setup` out of `print_pages()` and it uses the system's.

## Saving a PDF

`save_pdf(path, options)` writes the same pages to a PDF file without
showing anything. It takes what `print_pages()` takes, and raises a
`BackendError` if the file can't be written. Pair it with `save_file()`
for an Export as PDF command:

```zuri,ignore
save_file(context, {
  name: 'Notes.pdf',
  types: ['pdf'],
}).then(@(path) {
  if path != nil {
    save_pdf(path, { title: 'Notes', pages: count, paint })
  }
})
```

## Testing

Headless, the print and page setup dialogs wait in `backend.dialogs`
like any other. Answer the print dialog with `true` and every page is
drawn into a recording painter and kept in `backend.printed`, with the
job's `title`, `setup` and `pages`, each page the list of calls its
painter got. `save_pdf()` records its pages there too, with the `path`,
and writes no file.

```zuri,ignore
t.tap('Print…')
t.backend.answer_dialog(true)
var job = t.backend.printed[0]
expect(job.pages.length()).to_be(2)
expect(job.pages[0][0]).to_equal(['text', 'Invoice 1024', Point(0, 0)])
```

Answer a page setup dialog with a `PageSetup`, or `nil` to cancel it.

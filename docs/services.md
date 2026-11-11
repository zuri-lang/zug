# Timers, promises and the system

Everything on this page runs on the interface thread, the same as an
event handler. Whatever it changes is drawn at the next frame, and an
error it raises goes to `report_error()` instead of stopping the
application (see [When something goes wrong](errors.md)). Anything
that takes time gives you back a `Promise`.

## Promises

A `Promise` is a value that arrives later: the button the user picked
in an alert, the result of work in the background, the end of an
animation.

```zuri
import zug { * }

var ready = Promise.resolved(41)
  .then(@(value) => value + 1)
  .then(@(value) {
    echo 'got ${value}'
  })
  .catch_error(@(error) {
    echo 'failed: ${error.message}'
  })
```

`then(on_value)` runs a function with the value, and gives a new
promise of what that function returns, so steps chain. When a function
returns a promise, the chain waits for it. `catch_error(on_error)`
handles a failure anywhere earlier in the chain, and `always(callback)`
runs either way. `Promise.all(list)` waits for several promises and
gives the list of their values.

Callbacks never run during the call that settled the promise. They run
straight after, before the next frame. A rejection that nothing handles
is reported, so failures never vanish silently.

To make a promise of your own, use a `Deferred`. Call `resolve(value)`
or `reject(error)` on it once, and hand out its `promise`.
[`PromiseBuilder`](components/builders.md#promisebuilder) shows a
promise as it settles.

## Timers

```zuri,ignore
var toast = Timer(Duration.seconds(2), @{
  self.set_state(@{
    self.message = nil
  })
})

var clock = Timer.periodic(Duration.seconds(1), @(timer) {
  self.set_state(@{
    self.elapsed = timer.ticks
  })
})
```

`Timer(duration, fn)` calls `fn` once after the delay, and
`Timer.periodic(duration, fn)` calls it again and again, passing the
timer, whose `ticks` counts the calls. `cancel()` stops either one, and
`is_active()` says whether it will still fire. `delay(duration, value)`
is a promise that resolves with `value` after the delay.

Make a `Duration` with `Duration.ms(250)`, `Duration.seconds(2)` or
`Duration.minutes(5)`. Timers need a running application. Cancel them
in `dispose()`, so they don't outlive their widget.

## The clipboard

`Clipboard.write_text(text)` puts text on the system clipboard, and
`Clipboard.read_text()` returns what's there, or `nil` when it holds no
text.

## Dialogs

Dialogs return promises. Pass the `context` of a widget, and the dialog
opens as a sheet on that widget's window. Pass `nil` and it opens as a
window of its own.

```zuri,ignore
show_alert(context, {
  title: 'Delete “Q3 report”?',
  message: 'You can’t undo this.',
  style: AlertStyle.WARNING,
  buttons: ['Delete', 'Cancel'],
}).then(@(answer) {
  if answer.button == 0 {
    self.delete_report()
  }
})
```

`show_alert(context, options)` takes a `title`, a `message`, a `style`
(`AlertStyle.INFORMATIONAL`, `WARNING` or `CRITICAL`), and `buttons`,
with the default button first. A single OK button is used when you
leave `buttons` out. `suppression` adds a checkbox with that label,
such as “Don’t ask again”. The promise resolves with
`{ button, suppressed }`: the index of the button pressed, and whether
the checkbox was ticked.

`pick_files(context, options)` asks for files to open. It takes a
`title`, `types` (extensions such as `csv`), `multiple`, whether
`files` and `directories` can be chosen, and a `directory` to start
in. It resolves with a list of paths, or `nil` if the user cancelled.

`save_file(context, options)` asks where to save, with a suggested
`name`, and resolves with a path, or `nil`.

```zuri,ignore
pick_files(context, { types: ['txt', 'md'] }).then(@(paths) {
  if paths != nil {
    self.open(paths[0])
  }
})
```

### Dialogs of your own

When an alert's buttons aren't enough, `show_dialog(context, builder,
options)` shows any widgets you like as a dialog on the window of
`context`. On macOS it's a sheet that slides down from the window's
title bar, and the window takes no clicks or keys until it closes.

`builder` is called with a `close` function. Call `close(result)` to
dismiss the dialog, and the promise resolves with `result`:

```zuri,ignore
show_dialog(context, @(close) => RenameDialog(self.name, close)).then(@(name) {
  if name != nil {
    self.set_state(@{
      self.name = name
    })
  }
})
```

```zuri,ignore
class RenameDialog < StatefulWidget {
  var name
  var close

  @new(name, close) {
    self.name = name
    self.close = close
  }

  create_state() {
    return RenameState()
  }
}

class RenameState < State {
  var field

  init_state() {
    self.field = TextController(self.widget.name)
  }

  dispose() {
    self.field.dispose()
  }

  build(context) {
    var close = self.widget.close

    var buttons = Row([
      Spacer(),
      Button('Cancel', { role: ButtonRole.CANCEL, on_pressed: @{
        close(nil)
      } }),
      Button('Rename', { role: ButtonRole.PRIMARY, on_pressed: @{
        close(self.field.text())
      } }),
    ], { spacing: 8 })

    return Box({
      width: 320,
      padding: EdgeInsets.all(20),
      child: Column([
        Text('Rename'),
        TextField({ controller: self.field, autofocus: true }),
        buttons,
      ], { spacing: 12, align_items: AlignItems.STRETCH }),
    })
  }
}
```

The dialog is a real window of your application, so its widgets keep
their state while it's open, see your `Style`, and handle keys the
usual way. Return presses the `PRIMARY` button and Escape the `CANCEL`
one, which is what people expect of a dialog.

A dialog is the size of its content unless you pass `size`. Content
that stretches to fill the width, like the column above, needs a width
to fill, so give it one with a `Box`. `title` is used on platforms that
give dialogs a title bar.

With `context` left `nil`, the dialog goes on the window that had focus
last. If that window closes while the dialog is open, the dialog closes
too, and the promise resolves with `nil`.

### Fonts and colours

The font and colour choosers work differently. On macOS they're panels
that float beside your windows with no OK button: the user picks, and
expects to see the change straight away. So both take an `on_changed`
that's called with every pick. The promise resolves with the last pick
when the user closes the chooser, or `nil` if they never picked
anything.

```zuri,ignore
pick_font(context, {
  style: self.style,
  on_changed: @(style) {
    self.set_state(@{
      self.style = style
    })
  },
})
```

`pick_font(context, options)` starts from `style`, a `TextStyle`, and
calls `on_changed` with that style in the new font. Only the family,
size, weight and slant change. The colour and the rest of the style
stay as you gave them. Leave `style` out to start from the system font.

`pick_color(context, options)` starts from `color` and calls
`on_changed` with each `Color` picked. Pass `alpha: true` to let the
user choose an opacity too.

There's one font chooser and one colour chooser for the whole
application, so opening a second finishes the first, and its promise
resolves with whatever was picked there. For a colour that belongs in a
form, a [`ColorWell`](components/values.md#colorwell) is usually the
better fit.

## The system

- `open_url(url)` opens a link in the application the system chooses,
  such as the browser for `https:` or Mail for `mailto:`.
- `beep()` plays the alert sound, the usual answer to a key that can't
  do anything.
- `perform_role(role)` and `clear_focus(context)` are covered with
  [menus and the keyboard](apps-windows-and-menus.md#menus).

## Work in the background

Anything slow, such as parsing a big file or crunching numbers, should
run off the interface thread, so the window keeps responding.
`run_in_background(fn, argument)` runs a function on another isolate
and resolves with what it returns:

```zuri,ignore
def count_words(path) {
  return file(path).read().split(' ').length()
}

run_in_background(count_words, '/Users/ada/novel.txt').then(@(count) {
  self.set_state(@{
    self.words = count
  })
})
```

The function runs in an isolate of its own, so it gets a copy of its
argument, not your live objects, and its result is copied back. That's
why it has to be a function defined at the top level of a module, not a
method or a closure. If it raises, the promise is rejected with an
error whose message starts with the original error's type and message.

For a helper that keeps running and trades messages with you, use a
`Worker`. Its entry function receives two channels, one to read from
and one to reply on, and runs until it returns:

```zuri,ignore
def indexer(inbox, outbox) {
  while true {
    var path = inbox.recv()

    if path == nil and inbox.is_closed() {
      return
    }

    outbox.send({ path, words: file(path).read().split(' ').length() })
  }
}

var worker = Worker(indexer, {
  on_message: @(reply) {
    echo '${reply.path}: ${reply.words} words'
  },
})

worker.send('/Users/ada/notes.txt')
```

`on_message` gets each reply on the interface thread, `on_error` gets
the error if the entry function raises, and `close()` ends the worker.

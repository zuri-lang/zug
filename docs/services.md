# Timers, promises and the system

Everything on this page runs on the interface thread, like an event
handler: what it changes is drawn at the next frame, and an error it
raises goes to `report_error()` rather than stopping the application
(see [When something goes wrong](errors.md)). Anything that takes time
gives you back a `Promise`.

## Promises

A `Promise` is a value that arrives later: the button the user picked in
an alert, the result of work on another isolate, the end of an
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

`then(on_value)` runs a function with the value and gives a new promise
of what it returns, so steps chain. A function that returns a promise is
waited for. `catch_error(on_error)` handles a failure anywhere earlier in
the chain, and `always(callback)` runs either way. `Promise.all(list)`
waits for several and gives the list of their values.

Callbacks never run during the call that settled the promise; they run
straight after, before the next frame. A rejection that nothing handles
is reported, so failures don't vanish silently.

To make a promise of your own, use a `Deferred` and call `resolve(value)`
or `reject(error)` on it once; its `promise` is what you hand out.
`PromiseBuilder` shows a promise as it settles; see
[Widgets and state](widgets-and-state.md#building-in-place).

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
`Timer.periodic(duration, fn)` calls it repeatedly, with the timer, whose
`ticks` counts the calls. `cancel()` stops either, and `is_active()`
says whether it will still fire. `delay(duration, value)` is a promise
that resolves with `value` after the delay.

A `Duration` is made with `Duration.ms(250)`, `Duration.seconds(2)` or
`Duration.minutes(5)`. Timers need a running application, and you should
cancel them in `dispose()` so they don't outlive their widget.

## The clipboard

`Clipboard.write_text(text)` puts text on the system clipboard and
`Clipboard.read_text()` returns what's there, or `nil`.

## Dialogs

Dialogs return promises. Pass the `context` of a widget and the dialog
opens as a sheet on that widget's window; pass `nil` for a separate
window.

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
(`AlertStyle.INFORMATIONAL`, `WARNING` or `CRITICAL`), `buttons` with
the default first (`OK` when left out), and `suppression`, the label of
a "don't ask again" checkbox. It resolves with `{ button, suppressed }`.

`pick_files(context, options)` asks for files to open. It takes a
`title`, `types` (extensions such as `csv`), `multiple`, what may be
chosen with `files` and `directories`, and a `directory` to start in,
and resolves with a list of paths or `nil` if the user cancelled.
`save_file(context, options)` asks where to save, with a suggested
`name`, and resolves with a path or `nil`.

## The system

- `open_url(url)` opens a link in the application the system chooses,
  such as the browser for `https:` or Mail for `mailto:`.
- `beep()` plays the alert sound, the usual answer to a key that can't
  do anything.
- `perform_role(role)` and `clear_focus(context)` are covered with
  [menus and the keyboard](apps-windows-and-menus.md).

## Work in the background

Anything slow, such as parsing a big file or crunching numbers, should
run off the interface thread so the window stays responsive.
`run_in_background(fn, argument)` runs a top-level function on another
isolate and resolves with what it returns:

```zuri,ignore
def count_words(path) {
  return file(path).read().split(' ').length()
}

run_in_background(count_words, '/tmp/novel.txt').then(@(count) {
  self.set_state(@{
    self.words = count
  })
})
```

The function runs in an isolate of its own, so it sees copies of what it
was given, not your live objects, and its result is copied back. That's
why it has to be a top-level function. If it raises, the promise is
rejected with an error whose message starts with the original error's
type and message.

For a long-running helper that you exchange messages with, use a
`Worker`. Its entry function receives two channels, one to read from and
one to reply on, and runs until it returns:

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

var worker = Worker(indexer, { on_message: @(reply) {
  echo '${reply.path}: ${reply.words} words'
} })
worker.send('/tmp/notes.txt')
```

`on_message` gets each reply on the interface thread, `on_error` the
error if the entry function raises, and `close()` ends it.

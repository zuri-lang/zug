# Calculator

A calculator built with zug, as a whole application: its own project,
its own tests, and a bundle.

```console
$ zuri install
$ zuri run
$ zuri test
```

## Building the application

```console
$ zuri bundle --format app
$ open dist/calculator.app
```

What it does:

- **Keys** with the usual precedence (`2 + 3 × 4 =` is 14), parentheses,
  powers, `%` as a pocket calculator takes it, a sign key, `=` again to
  repeat the last operation, and memory keys.
- **A scientific keypad**: powers and roots, trigonometry in degrees or
  radians, logarithms, factorials and constants.
- **Typed expressions**: the box above the display calculates whatever
  is typed into it, such as `2 * (3 + 4)^2` or `sin(30) + pi`, says what
  is wrong with one it cannot, and offers the recent ones.
- **The keyboard**: digits, operators, Return, Escape and Delete work
  without clicking, and Copy and Paste work on the value shown when no
  text field has focus.
- **A tape** of past calculations; double-click one to use its result.
- **Settings**: decimal places, thousands separators, significant digits,
  the angle unit, and whether the tape is kept between launches. They are
  kept in `~/Library/Application Support/zug Calculator`, or wherever
  `CALCULATOR_HOME` names.

The code:

| File | Holds |
| --- | --- |
| `app/engine.zu` | What each key does and what the display shows. |
| `app/expression.zu` | The parser for typed expressions. |
| `app/format.zu` | How numbers are shown. |
| `app/settings.zu` | The settings and the file they are kept in. |
| `app/ui.zu` | The windows and menus. |

zug comes from the repository this example sits in, through
`zug = { path = "../.." }`, so `zuri install` copies the library as it
is. Run it again after changing the library.

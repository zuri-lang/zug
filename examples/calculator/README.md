# Calculator

A calculator built with zug, as a whole application: its own project,
its own tests, and a bundle.

```console
$ zuri install
$ zuri run
$ zuri test
```

## Building the application

`zuri bundle` packages the calculator with a Zuri runtime into a macOS
application that runs on a Mac without Zuri installed:

```console
$ zuri install
$ zuri bundle --format app
Built …/examples/calculator/dist/calculator.app (28.6 MiB)
$ open dist/calculator.app
```

`zuri install` comes first because the bundle takes zug as it is
installed in `.zuri/libs`, and installing copies it afresh from this
repository. The `[bundle]` section of `project.toml` names the
application and gives it its identifier:

```toml
[bundle]
name = "Calculator"
identifier = "dev.zug.calculator"
```

`name` is what Finder and the menu bar show, and `identifier` is
required, in reverse domain form. An `icon` key naming an `.icns` file in
the project gives the application its own icon. `exclude` in `[project]`
keeps the tests out of the bundle; the rest of the project goes in.

To install it, drag `dist/calculator.app` into `/Applications`, or:

```console
$ cp -R dist/calculator.app /Applications/Calculator.app
```

The bundle is signed ad hoc, which is all a Mac needs to run an
application built on it. To give it to others, sign it with a Developer
ID and notarize it. That needs the hardened runtime, under which Zuri's
JIT needs the `com.apple.security.cs.allow-jit` entitlement, so sign with
an entitlements file granting it:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.cs.allow-jit</key>
  <true/>
</dict>
</plist>
```

```console
$ codesign --force --options runtime --entitlements entitlements.plist \
    --sign "Developer ID Application: Your Name" dist/calculator.app
```

then submit it with `xcrun notarytool` as for any application. `zuri
bundle --help` lists the other options, such as `--output` and `--name`.

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

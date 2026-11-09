# Shipping an application

`zuri bundle` packages your project with a Zuri runtime into a macOS
application that runs on Macs without Zuri installed. zug needs nothing
compiled, so there's no extra build step.

## Building the bundle

Describe the application in a `[bundle]` section of `project.toml`:

```toml
[project]
name = "notes"
exclude = ["tests/**", "tmp/**"]

[bundle]
name = "Notes"
identifier = "com.example.notes"
icon = "assets/Notes.icns"
```

`name` is what Finder, the Dock and the menu bar show. `identifier` is
required, in reverse domain form, and identifies the application to
macOS, so pick it once and keep it. `icon` is optional and names an
`.icns` file in the project. Everything in the project goes into the
bundle except what `exclude` leaves out, which is usually your tests and
scratch files.

Then:

```console
$ zuri install
$ zuri bundle --format app
$ open dist/notes.app
```

Run `zuri install` first, because the bundle takes your dependencies,
zug included, as they're installed in `.zuri/libs`. `zuri bundle
--help` lists the other options, such as `--output` for where the bundle
goes and `--name` for its file name. To replace a bundle that's already
there, add `--force`.

## Signing and notarizing

The bundle is signed ad hoc, which is all your own Mac needs to run it.
To give it to other people, sign it with a Developer ID and have Apple
notarize it; otherwise Gatekeeper stops them opening it.

Notarizing needs the hardened runtime, and under the hardened runtime
Zuri's JIT needs the `com.apple.security.cs.allow-jit` entitlement. Make
an `entitlements.plist` granting it:

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

sign with it:

```console
$ codesign --force --options runtime --entitlements entitlements.plist \
    --sign "Developer ID Application: Your Name" dist/notes.app
```

and submit the result with `xcrun notarytool` as you would any other
application.

`examples/calculator` is a whole application set up this way, with its
own tests and a `[bundle]` section, if you'd like a working example to
copy.

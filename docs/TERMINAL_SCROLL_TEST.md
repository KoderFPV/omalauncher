# Isolated terminal scrolling test

This branch implements the unfocused vertical-drag behavior also proposed by
[FirstIntegral in PR #91](https://github.com/SimonSchubert/omalauncher/pull/91).
It is a narrower implementation on current main, not a verified rebase of that PR.
The active-terminal input path stays unchanged. The recognizer is attached to the
terminal container, in the initial pointer pass, so it also receives events above
the Android TextureView without globally intercepting other home widgets.

## Build without a PC

Push `terminal-scroll-test` to your fork. The `Terminal scroll test APK` workflow
runs tests, builds the native Linux runtime, builds/lints the regular configuration,
and builds/lints an isolated debug APK with `-PisolatedScrollTest=true`.
Download the `omalauncher-scroll-test-arm64` artifact from the successful run.
The APK must identify itself as `dev.omalauncher.scrolltest`.

The generated debug signing key is cached for subsequent test builds. Cache loss
can change that key; this is test signing, not a production upgrade channel.

## Installation safety

- Do not uninstall or clear data for the installed production Omalauncher.
- Install only the isolated APK, labelled `Omalauncher Scroll Test`.
- Open it as a separate app; do not make it the default home app for initial tests.
- It has its own private Linux system, widgets, settings and permissions.
  Install Linux in the test app and add a terminal there. Do not move the live
  launcher's rootfs or private data into it. Allow sufficient space for two installs.
- Installation and runtime behavior still require a physical-device test; a green
  build alone is not evidence that the gesture works on a particular tablet.

## Device test checklist

Inside the TEST app's terminal, produce local scrollback:

```sh
for i in {1..200}; do printf '%03d terminal scroll test\n' "$i"; done
```

1. Tap outside the terminal to deactivate it. Drag vertically over the text:
   scroll on this first press, without showing the keyboard or moving the drawer.
2. Swipe sideways on the inactive terminal: change home pages normally. Test diagonals.
3. Tap it: keyboard opens; existing typing, native selection, scrolling and the
   keyboard toggle still work. Back deactivates it; the next vertical drag scrolls again.
4. Long press and then move: arrange/resize the home tile, not its terminal output.
5. Swipe outside the terminal: configured home/drawer gestures still work.
6. Cancel a drag by leaving the app, rotating, closing the terminal or reloading
   its widget. No stuck touch, keyboard focus or scrolling should remain.
7. Test two terminal tiles, multiple pages, different font/display scales and a
   full-screen terminal face. Starting on one terminal must never scroll another.
8. Test multi-touch and a TUI with mouse reporting (for example btop). Native touch
   is preserved where available; programs may interpret scroll input differently.

The added JVM tests cover gesture direction, slop and long-press arbitration only.
End-to-end pointer delivery, cancellation, IME behavior and AndroidView hit testing
must be verified on-device before proposing this implementation upstream.

Keep the isolation/build-workflow changes separate from the gesture change when
preparing an upstream PR, and coordinate with #91 rather than silently duplicating it.

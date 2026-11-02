# Agents.md — loup-macos-calculator

A macOS desktop calculator in Loup, built on the shared `appkit` library
([myronccai/loup-macos-appkit](https://github.com/myronccai/loup-macos-appkit),
expected checked out next to this repository at `../loup-macos-appkit/`).
[README.md](README.md) describes what it does, the requirements (macOS, the
Xcode Command Line Tools, Loup toolchain newer than 1.27.0), how to build, run
and test it, and its structure.

## Where things are

- `loup-macos-calculator.lppkg` — the manifest (`host.native`, native delivery,
  `appkit = { path = "../loup-macos-appkit" }`, `-framework AppKit`).
- `src/main.lphc` — the AppKit UI and the `--self-test` driver (`host.native`).
- `src/calculator.loup` — the calculator state machine (`host.dual`, no
  AppKit); `src/numformat.loup` — rounding, formatting and parsing of numbers.
- `src/*_test.lpscr` — `loup test --strict src` tests of the two logic modules.
- `selftest.out` — the exact output of `./loup-macos-calculator --self-test`;
  `screenshot.png` — written by
  `bash ../loup-macos-appkit/run_all.sh --screenshots loup-macos-calculator`.
- The appkit library's reference is each module's header comment
  (`../loup-macos-appkit/*.lphc`) and `../loup-macos-appkit/docs/`; working examples of every module
  are in `../loup-macos-appkit/probes/`.
- The Loup macOS developer skill, bundled with the appkit checkout:
  `../loup-macos-appkit/skills/loup-macos-developer-skill/SKILL.md`.

## How to work here

- Keep arithmetic and display rules in `calculator.loup` / `numformat.loup`
  and test them with `loup test`; `main.lphc` only forwards keys to `press`
  and shows `displayText`.
- A behaviour change usually changes both the tests and `selftest.out`;
  regenerate the latter only after checking the new lines by hand.
- Use the appkit API; the app has no raw Objective-C and no toolchain
  workarounds. The keypad tables (`PAD_KEYS`, `OPERATOR_KEYS`,
  `MEMORY_KEYS`) are `const`s built from calculator's imported `KEY_*`
  consts; the keypad-Enter shortcut is a hidden Calculate item
  (`Menu.addHiddenItem`).
- The self-test reads every result back from the controls (labels, button
  titles, menu items, the hidden item's state) and fails (exit 1, via appkit's handler reporting) on
  a missing button, a wrong clear-key title or a number too wide for the
  display.
- Verify from this checkout with
  `bash ../loup-macos-appkit/run_all.sh loup-macos-calculator` (both drivers, both self-tests, `loup test`; `--screenshots` also
  refreshes `screenshot.png`). Never launch the app without `--self-test`
  from an automated session, and keep the self-test's clipboard use inside
  `saveClipboard` / `restoreClipboard`.

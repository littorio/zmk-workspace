# Local ZMK Workspace

This document describes the local keyboard configuration layered on top of the
inherited urob ZMK workspace.

## Modules

- `modules/zmk/zmk-keyboard-tcherta` contains physical keyboard shield
  definitions on separate branches: `tcherta_shield_0` for Tcherta and
  `plenka_1` for Plenka. Neither branch contains both shields.
- `modules/zmk/zmk-orbita-layout` contains the shared Orbita logical layout.
  Both keyboards include `include/orbita_layout.keymap.inc` from this module.
- `zmk` and `zephyr` are upstream West projects. The root `config/` directory
  retains the inherited urob layouts for other keyboards.

## Supported Targets

The active build matrix contains `plenka_left`, `plenka_right`,
`tcherta_left`, `tcherta_right`,
`settings_reset`, and `tester_pro_micro` for `nice_nano_v2`.

The matrix lists both keyboards, but only the keyboard whose sources are
checked out can be built. Switch the keyboard module before building:

```bash
git -C modules/zmk/zmk-keyboard-tcherta switch tcherta_shield_0
just build tcherta

# Or:
git -C modules/zmk/zmk-keyboard-tcherta switch plenka_1
just build plenka
```

The shared Orbita module stays on `main` for both keyboards. Layout changes,
including removal of the right-thumb Enter bindings, apply to both.

`config/west.yml` defaults to `tcherta_shield_0`. A local branch switch does not
require changing the manifest. `west update` uses the manifest revision and may
undo that local selection; switch to the desired branch again afterwards. For
a reproducible Plenka setup, change the manifest revision to `plenka_1` too.

Do not run `just build all` with this two-branch arrangement: it requests shields
that are absent from the checkout. CI uses `build-ci.yaml`, limited to Tcherta
and utility targets to match the default manifest. When changing the manifest
to Plenka for CI, update `build-ci.yaml` to the matching shield targets too.

## Documentation And Local Ignore Rules

This workspace document owns the cross-keyboard workflow. Keyboard-module
documentation describes the hardware and sources on its own branch; it need
not be identical between branches. Orbita documentation describes shared layout
behavior, not which keyboard branch is currently checked out.

Do not copy an untracked root `CONTEXT.md` or `.gitignore` between keyboard
branches: it can block checkout when the destination tracks the same path.
The previously generated context claiming both shields coexist was obsolete.

The shared sealing recipe ensures this local password-file ignore rule exists
in the keyboard repository's `.git/info/exclude`, which survives branch switches:

```gitignore
boards/shields/*/DELETE_ME_orbita-password.dtsi
```

Tracked `.gitignore` files can also contain this rule when maintained on each
branch. The local exclude is configured for this checkout and is added by
`just seal` for fresh checkouts without creating branch-specific metadata.

## Orbita Layout

`orbita_layout.keymap.inc` owns layers, combos, hold-taps, and shared macros.
Physical shields own only wiring, matrix transforms, and shield-specific Kconfig
configuration.

The left language-change thumb key taps `F20` and holds the right mouse button.
This needs `CONFIG_ZMK_POINTING=y` in the physical shield configuration. Since
that option changes the Bluetooth HID descriptor, forget and re-pair the
keyboard after first flashing firmware that enables it.

## Builds And Sealing

Use `just build plenka` or `just build tcherta` for a normal development build.
Use `just seal plenka` or `just seal tcherta` when firmware must include password
macros. The sealing recipe prompts
for three passwords, writes a mode-`0600` temporary
`DELETE_ME_orbita-password.dtsi`, invokes the normal build, and removes the
temporary file on exit.

The generated DTSI and resulting firmware contain sensitive material. The DTSI
is ignored by the keyboard module; do not commit it or distribute firmware built
with real passwords.

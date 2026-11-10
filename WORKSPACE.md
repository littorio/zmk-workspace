# Local ZMK Workspace

This document describes the local keyboard configuration layered on top of the
inherited urob ZMK workspace.

## Modules

- `modules/zmk/zmk-keyboard-tcherta` contains physical keyboard shield
  definitions. The active West revision is `tcherta_shield_0`.
- `modules/zmk/zmk-orbita-layout` contains the shared Orbita logical layout.
  Tcherta keymaps include `include/orbita_layout.keymap.inc` from this module.
- `zmk` and `zephyr` are upstream West projects. The root `config/` directory
  retains the inherited urob layouts for other keyboards.

## Supported Targets

The active build matrix contains `tcherta_left`, `tcherta_right`,
`settings_reset`, and `tester_pro_micro` for `nice_nano_v2`.

Plenka source is retained on the `plenka_1` branch of
`zmk-keyboard-tcherta`. It is not present on `tcherta_shield_0` and therefore
is intentionally not an active build target. Restore or merge that shield
before adding Plenka targets back to a build matrix.

## Orbita Layout

`orbita_layout.keymap.inc` owns layers, combos, hold-taps, and shared macros.
Physical shields own only wiring, matrix transforms, and shield-specific Kconfig
configuration.

The left language-change thumb key taps `F20` and holds the right mouse button.
This needs `CONFIG_ZMK_POINTING=y` in the physical shield configuration. Since
that option changes the Bluetooth HID descriptor, forget and re-pair the
keyboard after first flashing firmware that enables it.

## Builds And Sealing

Use `just build tcherta` for a normal development build. Use `just seal
tcherta` when firmware must include password macros. The sealing recipe prompts
for three passwords, writes a mode-`0600` temporary
`DELETE_ME_orbita-password.dtsi`, invokes the normal build, and removes the
temporary file on exit.

The generated DTSI and resulting firmware contain sensitive material. The DTSI
is ignored by the keyboard module; do not commit it or distribute firmware built
with real passwords.

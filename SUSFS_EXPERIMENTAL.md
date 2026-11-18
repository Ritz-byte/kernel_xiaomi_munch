# BakaSU + SUSFS validation for Poco F4

This candidate is experimental. Compilation and a device boot test are separate requirements. The normal `Build PitchKernel` workflow still uses the working NoSUSFS revision.

## Fixed source revisions

| Component | Revision |
| --- | --- |
| Known-good source baseline | `0819091436d012932771b9042803f36a0daaf479` |
| Experimental kernel source | `9087fe81a799bcdf5544b1c547426a1e3c3b14e5` |
| BakaSU submodule | `d5e9e025b8b49155f8014684d64c91742b465f65` |
| Existing SUSFS backport | `v2.3.0` |

[Review the source changes](https://github.com/Ritz-byte/pitch_kernel_sm8250/compare/0819091436d012932771b9042803f36a0daaf479...9087fe81a799bcdf5544b1c547426a1e3c3b14e5).

## Changes

- Run BakaSU's post-exec callbacks on both successful and failed exec paths when SUSFS is selected. The existing pending-exec flag fix remains applied.
- Copy userspace filenames with `getname_flags()` before passing `struct filename **` to the SUSFS stat and access hooks. Preserve copy errors and filename ownership through `filename_lookup()`.
- Enable the SUSFS config only with `PITCHKERNEL_SUSFS_EXPERIMENTAL=1`. Reject missing symbols after `olddefconfig` and save separate AOSP and MIUI config snapshots.

The regression tests compile the changed C hook blocks, exercise valid and invalid filenames, check hook modes, and verify that the configuration helper is opt-in. The original filename call and original SUSFS post-exec guards fail these tests.

## Validation workflow

`Validate BakaSU SUSFS` runs on the experimental branch. It builds AOSP and MIUI packages with the existing stable clang and packaging steps. It checks the BakaSU pin, selected hook mode, compiled SUSFS symbols, and BPF settings in both saved configs. It uploads Actions artifacts and diagnostics. It does not publish a release or send Telegram messages.

## Device validation still required

After compilation succeeds, validate the AOSP package on the Poco F4 running crDroid Android 16: boot, manager/root access, successful and failed executable launches, module loading, and reboot. Capture pstore/ramoops if boot fails. The previous bootloop's cause has not been confirmed from device logs, so these fixes do not establish boot compatibility.

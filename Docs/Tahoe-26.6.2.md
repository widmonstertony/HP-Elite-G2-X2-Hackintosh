# macOS Tahoe 26.6.2 deployment notes

Target build: macOS Tahoe 26.6.2 (25G83) using an in-place upgrade from macOS Ventura 13.6 (22G120).

## Design choices

- Keep `MacBookPro14,1`; do not change SMBIOS during the upgrade.
- Use OpenCore 1.0.7 with `RestrictEvents` compatibility patches.
- Keep Ventura's Broadcom path restricted to Darwin 22.
- Load `IOSkywalkFamily`, `IO80211FamilyLegacy`, and `AirPortBrcmNIC` only on newer Darwin versions; block the conflicting stock `com.apple.iokit.IOSkywalkFamily` path where required.
- Use `AMFIPass` and the Tahoe-specific AMFI path only on the matching Darwin version.
- Use AlpsHID 1.2.3 with the touchpad sleep-lifecycle fix.
- Use the Tahoe termination-guard builds of `VoodooI2CHID` and the embedded `VoodooInput` plugin to avoid shutdown/termination panics while preserving the touchscreen and multi-touch trackpad.
- Use the HP Elite x2 native F3/F4 map in `VoodooPS2Keyboard`, synthesize missing key-up events, and filter the single phantom brightness-down make code emitted when the detachable keyboard reconnects. No background hot-key agent is required.
- Use native XCPM instead of the former CPUFriend profile, which biased Tahoe toward power saving and caused severe post-boot sluggishness on this machine.
- Keep the Intel HD 620 `rps-control` property as the four-byte value `01 00 00 00`; an ASCII representation is not equivalent EFI data.
- Block `AppleThunderboltNHI` and disable the experimental Thunderbolt ACPI path after a confirmed `IOThunderboltFamily 9.3.3` page-fault panic.

## Validated state — 2026-09-27

- Boot, Intel HD 620 acceleration, internal audio, Wi-Fi, Bluetooth, touchscreen, keyboard, brightness/volume keys, battery reporting and 1–4 finger Alps gestures passed on the physical machine.
- Sleep/wake passed repeated short tests. Native wake authentication was restored by setting the system screen-lock policy to `immediate`; this is a macOS user policy, not an EFI patch.
- Shutdown and reboot passed after deploying the Tahoe HID termination guards.
- The detachable keyboard was removed and reattached without forcing brightness to minimum. The in-driver reconnect guard recorded one activation and filtered three phantom scan-code events; brightness keys continued to work normally.
- The machine remained responsive after cold boot with CPUFriend disabled and native XCPM active. No recent kernel panic, GPU restart, thermal warning, swap pressure, or sleep/wake driver failure was present in the final diagnostic pass.
- The live EFI config SHA-256 before public PlatformInfo sanitization is `4f994322d97321feefea4af3711ebfb101880b2514b3e4085139c28a1484adf2` and passed OpenCore 1.0.7 `ocvalidate`. Its corrected `rps-control` representation takes effect on the next boot.
- True Thunderbolt 3 PCIe tunnelling did not pass. A Tahoe 25G83 hot-unplug event reproduced the same `IOThunderboltFamily 9.3.3` null-page fault previously seen on Ventura.

## Upgrade order

1. Generate unique PlatformInfo values and validate the configuration with OpenCore 1.0.7 `ocvalidate`.
2. Back up the complete EFI System Partition and important personal data.
3. Deploy the transition EFI and boot the existing Ventura system once.
4. Verify keyboard, 1–4 finger touchpad gestures, Wi-Fi, graphics acceleration, battery reporting, and a short sleep/wake cycle.
5. Run the full Tahoe 26.6.2 installer against the existing system volume.
6. During installer restarts, select `macOS Installer` in OpenCore until it disappears.
7. On the first Tahoe desktop, apply only the OCLP-CustoMac `Modern Wi-Fi` root patch, then reboot.
8. Validate Wi-Fi reconnect, Bluetooth, two-way AirDrop, and sleep before applying any separate audio root patch.

## Important limitations

- BCM4360 is not natively supported by Tahoe. Wi-Fi and AirDrop depend on a root-patched system volume and are not guaranteed until tested on the physical machine.
- OCLP-CustoMac is a third-party Custom Mac fork. Root patches lower macOS security settings and may need to be reverted and reapplied around every system update.
- Apple Watch unlock depends on working Wi-Fi, Bluetooth, AWDL, Apple ID continuity, and valid unique PlatformInfo. It cannot be guaranteed by SMBIOS selection alone.
- Thunderbolt PCIe tunnelling is deliberately unavailable in this profile. Re-enabling it without mapped ACPI and controller firmware known to match this board may restore the panic. See [Thunderbolt.md](Thunderbolt.md).
- The internal camera is not enumerated by the current USB map and needs separate hardware/USB mapping work.
- Waking from the detachable PS/2 keyboard is not enabled. The current VoodooPS2 power path turns off keyboard clock/IRQ during sleep; use the power button to wake.

## Acceptance gates

- Wi-Fi scans, joins both 2.4 GHz and 5 GHz networks, and reconnects after reboot.
- AirDrop discovers and transfers one file in both directions.
- Touchpad retains 1–4 finger gestures after sleep.
- Sleep passes a 5-minute test and then a 30-minute test without a panic or unexpected shutdown report.
- Graphics acceleration, brightness, battery status, USB-C devices, audio, and Bluetooth work after a cold boot.

The tested installation passed the core gates above; two-way AirDrop and adapter-specific USB-C/DisplayPort behavior still need validation on each deployment. If a root patch breaks networking, revert root patches before changing EFI. If Tahoe cannot boot, restore the prior EFI from external recovery media; restoring EFI does not downgrade the operating system itself.

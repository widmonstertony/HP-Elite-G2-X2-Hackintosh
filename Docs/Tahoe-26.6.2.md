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
- Use the HP Elite x2 native F3/F4 map in `VoodooPS2Keyboard` for brightness down/up; no background hot-key agent is required.
- Disable the Thunderbolt NHI PCI path after a confirmed `IOThunderboltFamily 9.3.3` page-fault panic.

## Validated state — 2026-08-25

- Boot, Intel HD 620 acceleration, internal audio, Wi-Fi, Bluetooth, touchscreen, keyboard, brightness/volume keys, battery reporting and 1–4 finger Alps gestures passed on the physical machine.
- Sleep/wake passed repeated short tests. Native wake authentication was restored by setting the system screen-lock policy to `immediate`; this is a macOS user policy, not an EFI patch.
- Shutdown and reboot passed after deploying the Tahoe HID termination guards.
- The live EFI config SHA-256 before public PlatformInfo sanitization was `e0c8bf99cc3930ff9d9d840362d5b1d9cd77475c8398e5427fcdf7c23ede0efc` and passed OpenCore 1.0.7 `ocvalidate`.
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

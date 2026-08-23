# macOS Tahoe 26.6.2 transition notes

Target build: macOS Tahoe 26.6.2 (25G83) using an in-place upgrade from macOS Ventura 13.6 (22G120).

## Design choices

- Keep `MacBookPro14,1`; do not change SMBIOS during the upgrade.
- Use OpenCore 1.0.7 with `RestrictEvents` compatibility patches.
- Keep Ventura's Broadcom path restricted to Darwin 22.
- Load `IOSkywalkFamily`, `IO80211FamilyLegacy`, and `AirPortBrcmNIC` only on newer Darwin versions; block the conflicting stock `com.apple.iokit.IOSkywalkFamily` path where required.
- Use `AMFIPass` and the Tahoe-specific AMFI path only on the matching Darwin version.
- Use AlpsHID 1.2.3 with the touchpad sleep-lifecycle fix.
- Disable the Thunderbolt NHI PCI path after a confirmed `IOThunderboltFamily 9.3.3` page-fault panic.

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
- Thunderbolt PCIe tunnelling is deliberately unavailable in this profile. Re-enabling it without a mapped, tested ACPI solution may restore the panic.
- The internal camera is not enumerated by the current USB map and needs separate hardware/USB mapping work.

## Acceptance gates

- Wi-Fi scans, joins both 2.4 GHz and 5 GHz networks, and reconnects after reboot.
- AirDrop discovers and transfers one file in both directions.
- Touchpad retains 1–4 finger gestures after sleep.
- Sleep passes a 5-minute test and then a 30-minute test without a panic or unexpected shutdown report.
- Graphics acceleration, brightness, battery status, USB-C devices, audio, and Bluetooth work after a cold boot.

Do not enable automatic macOS updates until these gates pass. If Tahoe boots but a root patch breaks networking, revert root patches before changing EFI. If Tahoe cannot boot, restore the prior EFI from external recovery media; restoring EFI does not downgrade the operating system itself.

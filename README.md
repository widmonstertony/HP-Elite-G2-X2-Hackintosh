# HP Elite x2 1012 G2 Hackintosh

OpenCore configuration for the HP Elite x2 1012 G2, maintained for macOS Ventura 13.6 and a direct upgrade to macOS Tahoe 26.6.2 (25G83).

## Hardware

- Intel Core i7-7600U / Intel HD Graphics 620
- 16 GB RAM / 512 GB NVMe
- Broadcom BCM4360 Wi-Fi (`14E4:43A0`)
- Alps USB touchpad (`044E:1216`)
- SMBIOS: `MacBookPro14,1`
- OpenCore: 1.0.7 RELEASE

## Current status

| Feature | Ventura 13.6 | Tahoe 26.6.2 |
| --- | --- | --- |
| Boot / graphics acceleration | Tested | Upgrade pending final hardware test |
| Alps touchpad 1–4 finger gestures | Tested | Pending final hardware test |
| Broadcom Wi-Fi | Tested | Requires OCLP-CustoMac Modern Wi-Fi root patch |
| AirDrop / AWDL | Tested on Ventura | Must be tested after the Wi-Fi root patch |
| Bluetooth | Tested | Pending final hardware test |
| Sleep / wake | Short test passed after AlpsHID fix | Must pass 5-minute and 30-minute tests |
| Thunderbolt PCIe tunnelling | Disabled | Disabled |
| USB-C USB / charging / DisplayPort Alt Mode | Separate from disabled NHI; test per device | Pending final hardware test |
| Camera | Not enumerated | Not fixed by this EFI |

Thunderbolt NHI is intentionally disabled in `DeviceProperties` because `IOThunderboltFamily 9.3.3` caused a reproducible Ventura kernel panic. This profile prioritizes a stable upgrade over Thunderbolt PCIe tunnelling.

## Before using this EFI

The public `config.plist` is sanitized. Generate your own `MacBookPro14,1` PlatformInfo and replace all four placeholders:

- `SystemSerialNumber`
- `MLB`
- `SystemUUID`
- `ROM`

Never reuse identifiers from another machine and never publish working identifiers.

Back up the entire EFI System Partition and keep a bootable recovery USB before replacing an existing EFI. The EFI directory alone does not back up personal files and cannot downgrade a Tahoe system volume to Ventura.

## Tahoe 26.6.2 upgrade

See [Docs/Tahoe-26.6.2.md](Docs/Tahoe-26.6.2.md) for the exact transition strategy, post-install Wi-Fi patch, validation gates, known limitations, and rollback boundaries.

The bundled EFI is a transition profile: it boots the existing Ventura installation and carries the Tahoe-specific Broadcom compatibility path. Tahoe support must still be confirmed on the physical machine after installation and root patching.

## Validation

Validate `Hp Install OC/EFI/OC/config.plist` with the `ocvalidate` binary from the matching OpenCore 1.0.7 release before deploying any edit.

The file list and SHA-256 hashes for this public snapshot are recorded in `EFI-SHA256SUMS.txt`.

## Credits

- [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher)
- [OCLP-CustoMac](https://github.com/kgp-macPro/OCLP-CustoMac)

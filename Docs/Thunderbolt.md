# Thunderbolt and USB-C status

## Confirmed failure

The HP Elite x2 1012 G2 controller is not stable as a native macOS Thunderbolt controller with the current board firmware and ACPI. Three captured panics—two on Ventura 13.6 and one on Tahoe 26.6.2 build 25G83—have the same shape:

- page fault at address `0x0` in `com.apple.iokit.IOThunderboltFamily(9.3.3)`;
- `kernel_task` is the panicked task;
- the failure occurs while the machine is running, not during a normal shutdown;
- the Tahoe failure was reproduced by Thunderbolt hot removal.

The stable EFI therefore keeps `SSDT-TBHP.aml` disabled and hides the boot-visible RP01/NHI PCI paths. Do not interpret the physical USB-C connector as proof that Thunderbolt PCIe tunnelling is active: USB, charging and DisplayPort Alt Mode are separate functions.

The controller identified on this machine is an Intel JHL6540 (`8086:15d9`). The available same-model experimental SSDT advertises a JHL6340 and therefore is not a board-verified drop-in replacement for this machine.

## Why the experimental SSDT is not enabled

The same-model `SSDT-TbtOnPch` implementation is explicitly marked as experimental by its author: the driver was made visible with a force-power driver, the author did not have a Thunderbolt device to validate it, and sleep behavior remained conditional. The other established same-model OpenCore project reports boot-time-only Thunderbolt detection, loss of USB after wake, occasional kernel panic, and possible need for custom controller firmware.

Sources:

- [whatnameisit/HP-Elite-X2-1012-G2-Hackintosh](https://github.com/whatnameisit/HP-Elite-X2-1012-G2-Hackintosh)
- [midi1996/X2G2-opencore-hackintosh](https://github.com/midi1996/X2G2-opencore-hackintosh)

## Supported approach

- Treat the connector as USB-C/DisplayPort-only with this stable profile.
- Connect and remove USB-C adapters only while awake, and test each dock separately.
- Do not hot-plug or hot-unplug a true Thunderbolt PCIe device with the current stable EFI.
- Full Thunderbolt requires board-specific controller identification, ACPI, force-power behavior and potentially a controller-firmware flash. Firmware flashing can brick the controller and is intentionally outside this stable EFI.

A separate experimental profile should be used for any future Thunderbolt work. It must retain the stable EFI on a recovery USB and pass cold boot, device-before-boot, hot-plug, hot-unplug, sleep/wake and shutdown tests before promotion.

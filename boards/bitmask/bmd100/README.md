# BMD100

## Flash layout

The `bmd100` board target uses the flash layout BMD100 boards ship with, so an application built for it boots on the bootloaders already on the board. Every BMD100 leaves the factory with MCUboot on the application core, in single-slot mode with USB serial recovery, and the b0n bootloader on the network core. The debug port is locked, so USB is the way to load your own firmware.

| Core | Partition | Address | Size |
|---|---|---|---|
| Application | `mcuboot` | `0x00000` | 80 KB |
| Application | `image-0` (your application) | `0x14000` | 912 KB |
| Application | `storage` (settings) | `0xf8000` | 32 KB |
| Network | `b0n` | `0x00000` | 33.4 KB |
| Network | `b0-provision-data` | `0x08580` | 640 B |
| Network | `image-0` (the Bluetooth controller) | `0x08800` | 222 KB |

Build with sysbuild and these settings in `sysbuild.conf`:

```
SB_CONFIG_BOOTLOADER_MCUBOOT=y
SB_CONFIG_MCUBOOT_MODE_SINGLE_APP=y
SB_CONFIG_BOOT_SIGNATURE_TYPE_NONE=y
# Only to replace the network core image as well
SB_CONFIG_NETCORE_IPC_RADIO=y
SB_CONFIG_NETCORE_IPC_RADIO_BT_HCI_IPC=y
SB_CONFIG_SECURE_BOOT_NETCORE=y
SB_CONFIG_NETCORE_APP_UPDATE=y
```

Images are hash-checked, not signed. The build's MCUboot and b0n aren't loaded onto the board; only the images are.

To load them, start MCUboot's serial recovery: hold the button while plugging the board in, or, from your application, call `bootmode_set(BOOT_MODE_TYPE_BOOTLOADER)` and reboot (with `CONFIG_RETENTION_BOOT_MODE=y`). The LED turns red and the board appears as the USB serial device "BMD100 Bootloader". Then upload the images with [smpmgr](https://github.com/intercreate/smpmgr), the network core first if you built one:

```
SMP="smpmgr --port <port> --line-length 512 --line-buffers 8"
$SMP image upload --slot 3 build/signed_by_mcuboot_and_b0_ipc_radio.bin
$SMP image upload build/<app>/zephyr/zephyr.signed.bin
$SMP os reset
```

Don't upload the network core image anywhere but slot 3: in slot 0 or 1 it overwrites the application. After a network core upload, b0n takes a few seconds to copy the image into the network core's flash, and the reset is refused with `EBUSY` until it's done, so repeat it until it succeeds.

The settings partition still holds the previous firmware's settings, so erase it if your application uses a different settings backend.

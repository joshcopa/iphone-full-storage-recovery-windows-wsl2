# iPhone full-storage recovery — Windows 11 and WSL2

An anonymized successful recovery case: an iPhone stuck in Recovery Mode after an update failed due to insufficient storage was updated without requesting a factory reset. Existing personal data was accessible after boot.

> **Warning:** This is an experimental case report, not a guaranteed recovery procedure. Even an Update can cause data loss. If data is irreplaceable, consider a qualified recovery specialist. Never choose Erase or bypass safety prompts. Check firmware compatibility and current Apple signing status.

## Acknowledgements

This recovery was made possible by the open-source community. This repository documents the combination of existing solutions, not authorship of their patches.

- [3SENN — iphone-stuck-in-recovery-full-storage](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage): original Cryptex override and recovery methodology.
- [hamagharbi — usbmuxd PR #292](https://github.com/libimobiledevice/usbmuxd/pull/292): USB/IP zero-length-packet workaround that allowed the restore to proceed beyond NORData.
- [idevicerestore](https://github.com/libimobiledevice/idevicerestore), [libirecovery](https://github.com/libimobiledevice/libirecovery), [libtatsu](https://github.com/libimobiledevice/libtatsu), [usbmuxd](https://github.com/libimobiledevice/usbmuxd): firmware installation, recovery communication, TSS and multiplexed transport.
- [usbipd-win](https://github.com/dorssel/usbipd-win): Windows USB forwarding.
- [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice), [libusbmuxd](https://github.com/libimobiledevice/libusbmuxd), [libplist](https://github.com/libimobiledevice/libplist), [libimobiledevice-glue](https://github.com/libimobiledevice/libimobiledevice-glue), [libusb](https://github.com/libusb/libusb): supporting libraries.
- [usbmuxd issue #247](https://github.com/libimobiledevice/usbmuxd/issues/247) and [usbipd-win issue #924](https://github.com/dorssel/usbipd-win/issues/924): related transport investigations.
- [Microsoft WSL](https://github.com/microsoft/WSL) and [Ubuntu](https://ubuntu.com/wsl): Linux environment on Windows.

Thank you to all maintainers, contributors and issue reporters who made this recovery possible.

## Technical overview

The observed failure was `Failed preallocation due to space` during install_splat (failure 100). Additional Windows/WSL transport failures required AutoBind, stopping interfering Apple processes, and a patched usbmuxd. A successful Update subsequently completed and the device booted.

Tested configuration: Windows 11, WSL2 Ubuntu 26.04, usbipd-win 5.3.0, iPhone 15 family, iOS 27.2.1 (24B156), idevicerestore base 60192e9 with Cryptex patch, libirecovery base d767ca5, libtatsu cdbca26 and usbmuxd commit 4afca633b8460416e39dab3fc0e8ae10123cfec6. Additional local hotplug diagnostics and iBEC timeout changes were present; their necessity was not isolated.

## Recovery outline

1. Follow the original 3SENN guide to build the patched idevicerestore and prerequisites. Verify a compatible Apple-signed IPSW and the Upgrade identity, RestoreBehavior=Update.
2. Use usbipd-win to forward the phone to WSL2. Keep auto-attach active and account for the Recovery (05ac:1281) and Restore Mode (05ac:12ac) transitions.
3. Close Apple management software if it prevents Restore Mode capture (PNP_VetoOutstandingOpen). Do not uninstall drivers.
4. Use the usbmuxd PR #292 workaround if NORData fails with the 65536-byte message exceeding the 65535-byte limit. Build separately and run only one daemon.
5. Calculate the appropriate Cryptex size using the upstream guide. The historical suggested value was 6784 MiB for a real size of 6036 MiB. This is not a universal value.
6. Run the patched idevicerestore in Update mode. Never pass -e, -y or --ignore-errors. Do not interrupt an active restore.
7. Wait for the phone to finish its own startup progress after Restore Finished / DONE. Verify important data and make an encrypted backup immediately.

## Limitations and privacy

One successful case; no independent clean-machine reproduction or forensic verification of every file. No unique device identifiers, personal information, original private logs or firmware binaries are included. Upstream projects retain their licenses and credits.

# iPhone full-storage recovery — Windows 11 and WSL2

[Español](README.es.md) · [Technical report & commands](RECOVERY-REPORT.md) · [Troubleshooting](TROUBLESHOOTING.md)

**Nine attempts, several USB communication obstacles, and finally: `DONE`!** 🎉

An iPhone stuck in Recovery Mode after a failed update due to insufficient storage successfully completed an Update installation on Windows 11 using WSL2. Existing personal data was accessible after boot, without a factory reset.

If you are facing a similar problem, this case offers a starting point: identify the actual error, work through one obstacle at a time, and learn from the people who have already investigated it. Recovery is not guaranteed, but an error message is useful evidence for deciding what to do next.

> **Warning:** This is an experimental case report, not a guaranteed recovery procedure. Even an Update can cause data loss. If data is irreplaceable, consider a qualified recovery specialist. Never choose Erase or bypass safety prompts. Check firmware compatibility and current Apple signing status.

## Is this case relevant to you?

| What you see | Where to start |
| --- | --- |
| Error 100 / `install_splat` / `Failed preallocation due to space` | [Original Cryptex recovery method](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage) |
| iBEC reconnect or Restore Mode capture problems on Windows | [Troubleshooting](TROUBLESHOOTING.md) |
| NORData failure with a 65536-byte message exceeding the 65535-byte limit | [Technical report and USB/IP workaround](RECOVERY-REPORT.md) |

Other restore errors may have different causes. Match the actual log before attempting a firmware update.

## Acknowledgements — made possible by open source

This recovery was made possible by the open-source community. This repository documents the combination of existing solutions, not authorship of their patches.

- [3SENN — iphone-stuck-in-recovery-full-storage](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage): original Cryptex override and recovery methodology.
- [hamagharbi — usbmuxd PR #292](https://github.com/libimobiledevice/usbmuxd/pull/292): USB/IP zero-length-packet workaround that allowed the restore to proceed beyond NORData.
- [idevicerestore](https://github.com/libimobiledevice/idevicerestore), [libirecovery](https://github.com/libimobiledevice/libirecovery), [libtatsu](https://github.com/libimobiledevice/libtatsu), [usbmuxd](https://github.com/libimobiledevice/usbmuxd): firmware installation, recovery communication, TSS and multiplexed transport.
- [usbipd-win](https://github.com/dorssel/usbipd-win): Windows USB forwarding.
- [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice), [libusbmuxd](https://github.com/libimobiledevice/libusbmuxd), [libplist](https://github.com/libimobiledevice/libplist), [libimobiledevice-glue](https://github.com/libimobiledevice/libimobiledevice-glue), [libusb](https://github.com/libusb/libusb): supporting libraries.
- [usbmuxd issue #247](https://github.com/libimobiledevice/usbmuxd/issues/247) and [usbipd-win issue #924](https://github.com/dorssel/usbipd-win/issues/924): related transport investigations.
- [Microsoft WSL](https://github.com/microsoft/WSL) and [Ubuntu](https://ubuntu.com/wsl): Linux environment on Windows.

A huge thank you to all maintainers, contributors and issue reporters who made this recovery possible. The best part of sharing this experience is that it might help someone else recover their data, too.

## Technical overview

The observed failure was `Failed preallocation due to space` during install_splat (failure 100). Additional Windows/WSL transport failures were addressed with AutoBind, stopping interfering Apple processes, and a patched usbmuxd. A successful Update subsequently completed and the device booted.

Tested configuration: Windows 11, WSL2 Ubuntu 26.04, usbipd-win 5.3.0, iPhone 15 family, iOS 27.2.1 (24B156), idevicerestore base 60192e9 with Cryptex patch, libirecovery base d767ca5, libtatsu cdbca26 and usbmuxd commit 4afca633b8460416e39dab3fc0e8ae10123cfec6. Additional local hotplug diagnostics and iBEC timeout changes were present; their necessity was not isolated.

## Recovery outline

Start with the [original Cryptex guide](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage), then consult our [technical report](RECOVERY-REPORT.md) for the historical commands and [troubleshooting notes](TROUBLESHOOTING.md) for WSL2-specific errors. These documents supplement the upstream guide rather than replace it.

1. Follow the original 3SENN guide to build the patched idevicerestore and prerequisites. Verify a compatible Apple-signed IPSW and the Upgrade identity, RestoreBehavior=Update.
2. Use usbipd-win to forward the phone to WSL2. Keep auto-attach active and account for the Recovery (05ac:1281) and Restore Mode (05ac:12ac) transitions.
3. Close Apple management software if it prevents Restore Mode capture (PNP_VetoOutstandingOpen). Do not uninstall drivers.
4. Review the experimental usbmuxd PR #292 workaround if NORData fails with the 65536-byte message exceeding the 65535-byte limit. Build separately and run only one daemon.
5. Calculate the appropriate Cryptex size using the upstream guide. The historical suggested value was 6784 MiB for a real size of 6036 MiB. This is not a universal value.
6. Run the patched idevicerestore in Update mode. Never pass -e, -y or --ignore-errors. Do not interrupt an active restore.
7. Wait for the phone to finish its own startup progress after Restore Finished / DONE. Verify important data and make an encrypted backup immediately.

## Sharing the result

The results were shared with [3SENN (issue #36)](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage/issues/36) and [usbmuxd PR #292](https://github.com/libimobiledevice/usbmuxd/pull/292#issuecomment-7202018614). These are reports of a successful case, not endorsements by the maintainers. If you have a similar result, share only anonymized technical details.

## Limitations and privacy

One successful case; no independent clean-machine reproduction or forensic verification of every file. No unique device identifiers, personal information, original private logs or firmware binaries are included. Upstream projects retain their licenses and credits.

If this helps you reach `DONE`, remember that the next milestone is a verified backup. Happy troubleshooting, and thank you for keeping open-source knowledge moving forward!

## Found this useful?

The documentation is freely accessible. If it helped, a GitHub star, an anonymized recovery report, or sharing the guide with someone who needs it are all welcome ways to give back. Please also thank and support the upstream developers who made the recovery possible.

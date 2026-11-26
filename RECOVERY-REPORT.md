# Technical recovery report

[Back to README](README.md) · [Español](README.es.md) · [Troubleshooting](TROUBLESHOOTING.md)

From repeated recovery errors to a successful boot: this report records what changed between attempts, what was observed, and what remains uncertain.

## Environment

Historical configuration: Windows 11; WSL2 Ubuntu 26.04; usbipd-win 5.3.0; iPhone 15, 128 GB, iPhone15,4 / d37ap; iOS 27.2.1 build 24B156. These are non-unique product identifiers. No device-specific identifiers are included.

- idevicerestore base 60192e9 plus the [original Cryptex patch](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage) and local iBEC timeout modification.
- libirecovery base d767ca5, with local hotplug reference management and tracing.
- libtatsu cdbca26.
- usbmuxd 1.1.1-73-g4afca63, [PR #292](https://github.com/libimobiledevice/usbmuxd/pull/292).

The necessity of each local modification was not isolated. This is a historical report, not a complete clean-machine build guide.

## Failure progression

| Attempts | Observation |
| --- | --- |
| 1–2 | TSS succeeded; iBEC reconnect timed out. |
| 3 | iBEC reconnect observed after the original 60-second timeout. Local wait extended to 600 seconds. |
| 4–5 | iBEC and boot components sent; Restore Mode not captured. |
| 6 | AutoBind added; Windows reported PNP_VetoOutstandingOpen for the Restore Mode interface. |
| 7 | After stopping interfering Apple processes, Restore Mode was captured; NORData failed with an oversized-message transport error. |
| 8 | Attempt to leave Restore Mode through software reboot failed. Device subsequently returned to Recovery by manual restart. |
| 9 | Patched usbmuxd active, Upgrade installation completed; existing data accessible after boot. |

Original storage failure (selected diagnostic excerpts):
```text
Failed preallocation due to space
No hope in this succeeding, bailing.
FAILURE:100
```
The failed allocation was 6329204736 bytes for Cryptex1,SystemOS.

Transport failure:
```text
Unable to send NORData
message was too large (65536 bytes, max = 65535)
```

## Historical command reference

Commands are illustrative. Review upstream prerequisites, verify actual firmware signing and compatibility, and replace placeholders locally. Never share ECID or raw logs. An Update writes firmware and may still cause data loss. No erase flags or safety bypasses.

In elevated PowerShell, use the actual BUSID from usbipd list. Rules apply to matching device types and should be removed afterward.

```powershell
usbipd policy add --effect allow --operation AutoBind --hardware-id 05ac:1281
usbipd policy add --effect allow --operation AutoBind --hardware-id 05ac:12ac
# Replace YOUR_BUSID before running:
usbipd bind --busid YOUR_BUSID
usbipd attach --wsl --busid YOUR_BUSID --auto-attach
```

Keep auto-attach running during the update. If Windows reports an outstanding-open veto, close Apple management applications. After ensuring no other Apple operation is running, the historical intervention was:

```powershell
Stop-Process -Name AppleMobileDeviceProcess,iPodService,iTunesHelper -Force -ErrorAction SilentlyContinue
```

Bonjour was also stopped temporarily, but its necessity was not established. Do not uninstall drivers.

In Ubuntu, follow upstream build prerequisites for usbmuxd. Build the workaround separately rather than installing it globally:

```bash
cd ~
git clone https://github.com/libimobiledevice/usbmuxd.git usbmuxd-fix
cd usbmuxd-fix
git fetch origin pull/292/head:fix-usbipd-zlp
git switch --detach 4afca633b8460416e39dab3fc0e8ae10123cfec6
./autogen.sh --prefix="$HOME/usbmuxd-fix/install"
make -j4
./src/usbmuxd --version
ldd ./src/usbmuxd
```

Check dependencies and review source. With no active restore, stop the old daemon gracefully and launch the patched executable in its own terminal:

```bash
sudo "$HOME/usbmuxd-fix/src/usbmuxd" -f -v -v
```

Ensure only one daemon is running. Check lsusb and use irecovery -q locally to confirm Recovery; do not publish its output without redaction.

The original Cryptex guide supplied the size suggestion tool. With no restore log supplied to that tool, it reported 6036 MiB and suggested 6784 MiB (+748 MiB) for the historical build; this is not universal. Follow the upstream sizing instructions for the actual device and firmware. Verify BuildManifest Upgrade identity and RestoreBehavior=Update. The successful command structure was:

```bash
ECID='YOUR_ECID'
IPSW='/path/to/compatible-signed-firmware.ipsw'
CRYPTEX_MIB='YOUR_CALCULATED_SIZE'
sudo env LD_LIBRARY_PATH="$HOME/libirecovery/install/lib" \
  IDEVICERESTORE_CRYPTEX_SIZE_MIB="$CRYPTEX_MIB" \
  "$HOME/idevicerestore/install/bin/idevicerestore" \
  -d -P --variant 'Customer Upgrade Install (IPSW)' \
  -i "$ECID" --logfile="$HOME/iphone-update.log" "$IPSW"
```

This assumes the historical local build prefixes. Confirm all placeholders and dependencies first. Never pass -e, -y or --ignore-errors. A no-action check (-n) is not a complete restore simulation.

## Successful result — the welcome sight of DONE

```text
Status: Restore Finished
DONE
progress: 7 1.000000
```

The device completed its own subsequent progress bar, displayed Hello, and existing data was accessible after unlocking. No independent forensic comparison of every file was made. Create an encrypted backup promptly, keep its password safe, and verify the backup before deleting any data.

## Cleanup

Only after the update and startup have completed: stop temporary USB forwarding and the patched daemon, detach from WSL if necessary, remove only the added AutoBind policies (consult usbipd policy remove --help), and restore previously running services if appropriate.

## Credits

See [the main acknowledgements](README.md#acknowledgements). All upstream code and patches remain attributed to their original authors. This report adds observations about integrating their solutions on Windows/WSL2.

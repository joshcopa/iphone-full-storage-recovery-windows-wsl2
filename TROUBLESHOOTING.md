# Troubleshooting

[Back to README](README.md) · [Español](TROUBLESHOOTING.es.md) · [Technical report](RECOVERY-REPORT.md)

An error is a clue, not a diagnosis. Start with the exact message and the stage that failed, then check the corresponding layer before changing anything.

Do not retry blindly. If irreplaceable data is at risk, consider a specialist. Never choose Erase or bypass safety prompts.

| Symptom | Interpretation and checks |
| --- | --- |
| Error 100 / install_splat / Failed preallocation due to space | Inspect the actual restore log. This case involved Cryptex preallocation. Not every error 100 has this cause. Follow the original 3SENN size-override guide only for matching circumstances. |
| iBEC sent, no reconnect | Check USB/IP auto-attach, lsusb and kernel logs. A longer reconnect delay was observed in this case. A local timeout increase was used, but it is not independently validated as necessary. |
| Device failed to enter Restore Mode | The generic error does not prove image personalization failed. Check Windows USB enumeration and usbipd events for 05ac:12ac. |
| CR_REMOVE_VETOED / PNP_VetoOutstandingOpen | A Windows application or service may have an open handle. Close Apple management apps and inspect processes; do not uninstall drivers. |
| Unable to send NORData with 65536 > 65535 message | Matching symptoms of the USB/IP ZLP issue. Review the experimental workaround in usbmuxd PR #292, compile separately, verify the patched daemon actually runs. |
| Unable to exit Restore Mode | Do not blindly repeat an update. Verify no active restore, diagnose device state and consult recovery guidance before any restart. |
| DONE but phone still shows progress | The host installation completed; device startup may still be in progress. Do not interrupt while progressing. |
| Hello after update | Not proof of data loss or preservation. Confirm normal access to existing data before claiming success. Stop if an unexpected erase or setup-as-new prompt appears. |

## Useful checks

- Windows: usbipd list, usbipd state, relevant Windows application event logs.
- WSL: lsusb, dmesg, pgrep -a idevicerestore, pgrep -a usbmuxd.
- Verify executable paths, library versions and actual daemon ownership before changing processes.
- Never kill an active restore or disconnect the device during installation.

## Privacy before sharing diagnostics

Remove unique device identifiers (serial, ECID, UDID, IMEI), nonces, signing tickets, usernames, personal file paths, hostnames, IP addresses and account details. Prefer short excerpts that show only the relevant error. Do not publish complete restore logs or firmware files.

## References

- [Original Cryptex method](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage)
- [usbmuxd PR #292](https://github.com/libimobiledevice/usbmuxd/pull/292)
- [usbmuxd issue #247](https://github.com/libimobiledevice/usbmuxd/issues/247)
- [usbipd-win](https://github.com/dorssel/usbipd-win)

Thank you to the original developers and community investigators. If you share a new finding, include the relevant error and software versions, explain what changed, and distinguish observations from assumptions.

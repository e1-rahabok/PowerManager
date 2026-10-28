# Work log

A dated record of what has been done on this laptop, and why. Newest entries first.

## 28 October 2026: overnight crashes while docked

### Why

The laptop crashed overnight while docked, and again on 27/28 October. The user asked why, and what to change.

### Evidence (Windows Crash folder, exported 24 and 28 October)

Bugchecks from the Kernel-Power 41 and WER 1001 events:

| Date | Time | Bugcheck | Connected Standby | Overnight |
|---|---|---|---|---|
| 13 Oct | 14:21 | 0x116 VIDEO_TDR_FAILURE | No | No |
| 16 Oct | 07:11 | 0x133 DPC_WATCHDOG_VIOLATION | No | No |
| 18 Oct | 13:45 | 0x116 VIDEO_TDR_FAILURE | No | No |
| 19 Oct | 02:21 | 0x133 DPC_WATCHDOG_VIOLATION | No | Yes |
| 20 Oct | 02:28 | 0x7E SYSTEM_THREAD_EXCEPTION_NOT_HANDLED | Yes | Yes |
| 23 Oct | 14:34 | 0x3B SYSTEM_SERVICE_EXCEPTION | Yes | No |
| 25 Oct | 00:50 | 0x19C (WIN32K_POWER_WATCHDOG_TIMEOUT, not verified in dump) | No | Yes |
| 28 Oct | 07:40 | 0x7E SYSTEM_THREAD_EXCEPTION_NOT_HANDLED | Yes | Yes |

What the logs show:

- **12 to 20 Oct: NVIDIA GPU errors.** 33 TDR events (12 Oct) and 43 (19 Oct). "UCODE Fatal Error" and GPU reset events. 88 PCIe corrected errors from the NVIDIA GPU on 20 Oct.
- **After 20 Oct: GPU errors almost stop.** 3 PCIe errors on 25 Oct, 2 on 28 Oct.
- **The 20 and 28 Oct 0x7E crashes match.** Same parameter pattern and same address low bits (0x931f). Likely the same driver. Not yet named.
- **Common modules in recent dumps:** NVIDIA HDMI/DP audio (nvhda64v.sys), HD audio bus (HDAudBus.sys), USB-C controller (UcmUcsiCx.sys). The dock connects over USB-C.
- **Sleep state:** 4 of 8 crashes were overnight. 3 of 8 had Connected Standby active. No hibernate activity in the logs.

### Conclusion

- 12 to 20 Oct: NVIDIA GPU driver or firmware, likely.
- 20 to 28 Oct: a driver in the power, USB-C, or audio path, during Connected Standby. Not yet named.
- The dock and overnight standby are shared conditions. Neither is proven as the cause.

### Actions taken

1. **Disabled wake from the dock and USB devices.** Ran `powercfg /devicedisablewake` on:
   - USB4 Root Router (1.0)
   - USB4 Root Router (2.0)
   - Realtek USB GbE Family Controller #2 (the dock's Ethernet)

   Result: `powercfg /devicequery wake_armed` now lists only Intel(R) USBIO Bridge Device and Intel(R) Wi-Fi 7 BE201 320MHz.

   **Why:** these devices could wake the laptop from standby. Unexpected wakes during standby overnight are a possible crash trigger. Disabling them stops those wakes.

   **Side effect:** the dock keyboard and mouse can no longer wake the laptop. Use the power button.

   **Not done:** USBIO Bridge Device is still armed. It is optional and was left for a decision.

   **Does not fix the crash cause.** It only reduces wakes.

### Not done yet

- WinDbg `!analyze -v` on `102826-25453-01.dmp` (28 Oct) and `102026-23781-01.dmp` (20 Oct), to name the faulting driver.
- Clean reinstall of the NVIDIA driver with DDU. Prefer the Dell-validated driver.
- Dell dock firmware, USB-C and Thunderbolt driver, chipset, and BIOS updates.
- A/B test overnight: docked vs undocked, and Connected Standby on vs off.
- A fresh export of Crashes View and GPU Errors View after 28 Oct.

### Related

- [KNOWN-ISSUES.md](KNOWN-ISSUES.md): external monitors on the dock are not detected after standby or hibernate. The same standby path may be involved.

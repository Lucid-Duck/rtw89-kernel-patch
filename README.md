# rtw89 USB TX Flow Control Patch

> **This repo has been superseded by [tx-resources-flow-control](https://github.com/Lucid-Duck/tx-resources-flow-control)**, which contains the v2 patch series, full test results, and mailing list status. This repo is archived for historical reference.

---

Patch for linux-wireless mailing list - fixes mac80211 TX flow control contract violation in rtw89 USB driver.

## Status

- [x] v1 submitted to linux-wireless (2026-01-25)
- [x] Review feedback received from Ping-Ke Shih and Bitterblue Smith
- [x] USB2/USB3 throughput testing complete
- [x] **v2 submitted** -- see [tx-resources-flow-control](https://github.com/Lucid-Duck/tx-resources-flow-control)

## Test Results Summary

Tested on D-Link DWA-X1850 (RTL8832AU) with iperf3:

| Config | Download Improvement | Notes |
|--------|---------------------|-------|
| USB3 5GHz | **+44%** (494→709 Mbps) | 88% fewer retransmits |
| USB3 2.4GHz | **+25%** (54→68 Mbps) | |
| USB2 5GHz | **+15%** (196→225 Mbps) | |
| USB2 2.4GHz | +6% (123→131 Mbps) | |

Full results: [TEST-RESULTS-FINAL-2026-01-29.md](TEST-RESULTS-FINAL-2026-01-29.md)

## Files

- `usb.c` / `usb.h` - Patched driver files
- `original/` - Unmodified driver files (commit 2544ebf)
- `usb.c.patch` / `usb.h.patch` - Unified diff patches
- `0001-wifi-rtw89-usb-fix-TX-flow-control-by-tracking-in-fl.patch` - git-format-patch output

## The Bug

`rtw89_usb_ops_check_and_reclaim_tx_resource()` returns a hardcoded `42`:
```c
return 42; /* TODO some kind of calculation? */
```

This violates mac80211's TX flow control contract, causing uncontrolled URB accumulation under sustained TX load.

## The Fix

Add atomic counters (`tx_inflight[]`) to track in-flight URBs per TX channel:
- Increment before `usb_submit_urb()` with rollback on failure
- Decrement in completion callback
- Return `(MAX_URBS - inflight)` to mac80211

## Mailing List Thread

https://lore.kernel.org/linux-wireless/20260125221943.36001-1-lucid_duck@justthetip.ca/

## Base Driver

morrownr/rtw89 commit 2544ebf

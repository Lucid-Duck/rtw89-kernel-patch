# rtw89 USB TX Flow Control Patch

Patch for submission to linux-wireless mailing list.

## Status
- [x] Patch created against wireless-next
- [x] checkpatch.pl passes (0 errors, 0 warnings)
- [x] Commit message follows kernel style
- [ ] Get maintainer list with `get_maintainer.pl`
- [ ] Configure git send-email
- [ ] Send to linux-wireless@vger.kernel.org

## Files
- `0001-wifi-rtw89-usb-fix-TX-flow-control-by-tracking-in-fl.patch` - The formatted patch
- `usb.c` - Modified driver file (for reference)
- `usb.h` - Modified header file (for reference)

## Next Steps (on Linux)

1. Clone wireless-next:
```bash
git clone --depth=1 https://git.kernel.org/pub/scm/linux/kernel/git/wireless/wireless-next.git
cd wireless-next
```

2. Apply and verify the patch:
```bash
git am ../0001-wifi-rtw89-usb-fix-TX-flow-control-by-tracking-in-fl.patch
scripts/checkpatch.pl -g HEAD
```

3. Get maintainers:
```bash
scripts/get_maintainer.pl -f drivers/net/wireless/realtek/rtw89/usb.c
```

4. Configure git send-email (Gmail example):
```bash
git config --global sendemail.smtpserver smtp.gmail.com
git config --global sendemail.smtpserverport 587
git config --global sendemail.smtpencryption tls
git config --global sendemail.smtpuser your@gmail.com
# Use app password, not regular password
```

5. Send patch:
```bash
git send-email --to linux-wireless@vger.kernel.org \
    --cc <maintainers from step 3> \
    0001-wifi-rtw89-usb-fix-TX-flow-control-by-tracking-in-fl.patch
```

## The Bug Being Fixed

`rtw89_usb_ops_check_and_reclaim_tx_resource()` returns a hardcoded `42`:
```c
return 42; /* TODO some kind of calculation? */
```

This violates mac80211's TX flow control contract, causing uncontrolled URB accumulation.

## The Fix

Add atomic counters to track in-flight URBs per TX channel, implementing proper backpressure.

## Original PR
https://github.com/morrownr/rtw89/pull/53

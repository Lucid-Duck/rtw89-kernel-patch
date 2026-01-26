# Linux Wireless Mailing List Discourse

Record of all patch submissions and maintainer responses.

---

## [PATCH] wifi: rtw89: usb: fix TX flow control by tracking in-flight URBs

**Submitted:** 2026-01-25 22:19 UTC
**Author:** Lucid Duck <lucid_duck@justthetip.ca>
**Message-ID:** `20260125221943.36001-1-lucid_duck@justthetip.ca`
**Archive:** https://lore.kernel.org/linux-wireless/20260125221943.36001-1-lucid_duck@justthetip.ca/

### Patch Summary

Fixes `rtw89_usb_ops_check_and_reclaim_tx_resource()` which returned hardcoded `42`, violating mac80211's TX flow control contract.

---

## RE: [PATCH] wifi: rtw89: usb: fix TX flow control by tracking in-flight URBs

**From:** Ping-Ke Shih <pkshih@realtek.com>
**Date:** Mon, 26 Jan 2026 03:39:47 +0000
**Message-ID:** `290226f1d7144477a668f045cbd8eb56@realtek.com`
**Archive:** https://lore.kernel.org/linux-wireless/290226f1d7144477a668f045cbd8eb56@realtek.com/

### Full Response

```
+ developers of WiFi USB adapters

Lucid Duck <lucid_duck@justthetip.ca> wrote:

> rtw89_usb_ops_check_and_reclaim_tx_resource() currently returns a
> hardcoded placeholder value of 42, violating mac80211's TX flow control
> contract. This causes uncontrolled URB accumulation under sustained TX
> load since mac80211 believes resources are always available.

Then URB becomes exhausted?

> Fix this by implementing proper TX backpressure:
>
> - Add per-channel atomic counters (tx_inflight[]) to track URBs between
>   submission and completion
> - Increment counter before usb_submit_urb() with rollback on failure
> - Decrement counter in completion callback
> - Return available slots (max - inflight) to mac80211, or 0 at capacity
> - Exclude firmware command channel (CH12) from flow control
>
> Tested on D-Link DWA-X1850 (RTL8832AU) with:
> - Sustained high-throughput traffic
> - Module load/unload stress tests
> - Hot-unplug during active transmission
> - 30-minute soak test verifying counters balance at idle
>
> Signed-off-by: Lucid Duck <lucid_duck@justthetip.ca>

[...]

> diff --git a/drivers/net/wireless/realtek/rtw89/usb.h b/drivers/net/wireless/realtek/rtw89/usb.h
> index 203ec8e99..f72a8b1b2 100644
> --- a/drivers/net/wireless/realtek/rtw89/usb.h
> +++ b/drivers/net/wireless/realtek/rtw89/usb.h
> @@ -20,6 +20,9 @@
>  #define RTW89_MAX_ENDPOINT_NUM          9
>  #define RTW89_MAX_BULKOUT_NUM           7
>
> +/* TX flow control: max in-flight URBs per channel */
> +#define RTW89_USB_MAX_TX_URBS_PER_CH    32

Curiously. How did you decide this value? Have you tested USB2 and USB3
devices? How about their throughput before/after this patch?

> +
>  struct rtw89_usb_info {
>       u32 usb_host_request_2;
>       u32 usb_wlan0_1;
> @@ -63,6 +66,9 @@ struct rtw89_usb {
>       struct usb_anchor tx_submitted;
>
>       struct sk_buff_head tx_queue[RTW89_TXCH_NUM];
> +
> +     /* TX flow control: track in-flight URBs per channel */

I feel we don't need repeatedly adding this comment. If you like it,
just keep one.

> +     atomic_t tx_inflight[RTW89_TXCH_NUM];
>  };
>
>  static inline struct rtw89_usb *rtw89_usb_priv(struct rtw89_dev *rtwdev)
> --
> 2.52.0
>
```

### Action Items

Ping-Ke Shih's questions require response:

1. **"Then URB becomes exhausted?"**
   - Confirm: Yes, without flow control URBs accumulate until submission fails

2. **"How did you decide this value [32]?"**
   - Need to explain rationale for `RTW89_USB_MAX_TX_URBS_PER_CH = 32`

3. **"Have you tested USB2 and USB3 devices?"**
   - Document testing coverage

4. **"How about their throughput before/after this patch?"**
   - Provide performance measurements if available

5. **"I feel we don't need repeatedly adding this comment"**
   - Remove duplicate "TX flow control" comment in v2

### Response Status

- [ ] Draft response
- [ ] Send reply via git send-email
- [ ] Submit v2 patch if needed

---

## Reply Template

To reply, use:

```bash
git send-email \
    --in-reply-to=290226f1d7144477a668f045cbd8eb56@realtek.com \
    --to=pkshih@realtek.com \
    --cc=linux-wireless@vger.kernel.org \
    --cc=lucid_duck@justthetip.ca \
    --cc=mh_chen@realtek.com \
    --cc=rtl8821cerfe2@gmail.com \
    /path/to/YOUR_REPLY
```

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

## Response 1: Ping-Ke Shih (Realtek)

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

static inline struct rtw89_usb *rtw89_usb_priv(struct rtw89_dev *rtwdev)
> --
> 2.52.0
>
```

### Questions Raised

1. **"Then URB becomes exhausted?"** — Confirm failure mode
2. **"How did you decide this value [32]?"** — Rationale for `RTW89_USB_MAX_TX_URBS_PER_CH`
3. **"Have you tested USB2 and USB3 devices?"** — Test coverage question
4. **"How about their throughput before/after this patch?"** — Performance data request
5. **"I feel we don't need repeatedly adding this comment"** — Style feedback

---

## Response 2: Mh_chen (Realtek)

**From:** Mh_chen <mh_chen@realtek.com>
**Date:** Mon, 26 Jan 2026 10:14 UTC
**Archive:** https://lore.kernel.org/linux-wireless/

### Summary

Forwarded Ping-Ke Shih's message to additional developers (Isaiah) for wider WiFi USB driver review. No new questions — primarily served as escalation to relevant maintainers.

---

## Response 3: Bitterblue Smith

**From:** Bitterblue Smith <rtl8821cerfe2@gmail.com>
**Date:** Mon, 26 Jan 2026 14:09 UTC
**Archive:** https://lore.kernel.org/linux-wireless/

### Questions and Concerns

1. **CH12 checks redundant:** "The CH12 checks in the completion handler are unnecessary since there is one in the resource checking function."

2. **Design question:** "Is there a reason to add a new counter instead of just using the length of each tx_queue?"
   - Suggests using `skb_queue_len(&rtwusb->tx_queue[ch])` instead of separate atomic counters
   - Implies the implementation might be over-engineered

### Action Items for v2

- [ ] Remove redundant CH12 exclusion check in completion handler
- [ ] Justify atomic counter vs skb_queue_len() approach, or simplify

---

## My Reply to Ping-Ke Shih

**Sent:** Mon, 26 Jan 2026 21:00:27 -0800
**Message-ID:** `20260127050028.478-1-lucid_duck@justthetip.ca`
**Archive:** https://lore.kernel.org/linux-wireless/20260127050028.478-1-lucid_duck@justthetip.ca/

```
On Mon, 26 Jan 2026, Ping-Ke Shih wrote:
> Then URB becomes exhausted?

Yes. Without proper flow control, mac80211 continuously queues frames
since we always report resources available. URBs accumulate until
submission fails, causing TX stalls or instability under load.

> Curiously. How did you decide this value? Have you tested USB2 and USB3
> devices? How about their throughput before/after this patch?

The value of 32 was based on similar USB wireless drivers (mt76, ath9k_htc)
as a reasonable starting point. I'm open to tuning this if testing reveals
a better value, and it may need adjustment for optimal USB2 vs USB3
performance at different bands.

I have both USB2 and USB3 test capability and will be running more
rigorous throughput testing on both configurations shortly. Initial
testing showed the patch stable under sustained load, but I want to
collect proper iperf3 measurements before providing specific numbers.

I'll follow up with detailed test results and a v2 addressing your
comments.

> I feel we don't need repeatedly adding this comment. If you like it,
> just keep one.

Understood. Will clean this up in v2.

Thanks for the review.

--
Lucid Duck
```

---

## Outstanding Action Items

### For v2 Patch

- [ ] Remove duplicate "TX flow control" comment (Ping-Ke feedback)
- [ ] Address Bitterblue's CH12 redundancy concern
- [ ] Justify or simplify atomic counter approach (Bitterblue feedback)
- [ ] Run USB2 and USB3 throughput tests (Ping-Ke request)
- [ ] Collect iperf3 before/after measurements
- [ ] Document test results in reply

### Reply Needed

- [ ] **Bitterblue Smith** — Need to address his design questions about atomic counters vs skb_queue_len()

---

## Reply Template (for Bitterblue)

```bash
git send-email \
    --in-reply-to=<bitterblue-message-id> \
    --to=rtl8821cerfe2@gmail.com \
    --cc=linux-wireless@vger.kernel.org \
    --cc=pkshih@realtek.com \
    /path/to/YOUR_REPLY
```

---

*Last updated: 2026-01-27*

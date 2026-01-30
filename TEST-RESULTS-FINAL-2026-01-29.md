# RTW89 TX Flow Control Patch - LAN Test Results

**Date:** 2026-01-29
**Hardware:** D-Link DWA-X1850 (RTL8832AU, USB ID 2001:3321)
**Driver:** morrownr/rtw89 commit 2544ebf
**Test Network:** Windows PC (192.168.99.111) via 8 Hertz WAN IP (WiFi-to-WiFi)
**iperf3 server:** Windows PC with Alfa adapter on same WiFi network

---

## UNPATCHED Driver (`return 42` placeholder)

### USB3 5GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 265 | 777 | 1 |
| 2 | 302 | 700 | 1 |
| 3 | 336 | 803 | 0 |
| 4 | 436 | 798 | 0 |
| 5 | 666 | 803 | 0 |
| 6 | 378 | 774 | 0 |
| 7 | 596 | 800 | 0 |
| 8 | 604 | 786 | 3 |
| 9 | 661 | 531 | 1 |
| 10 | 692 | 796 | 1 |
| **Avg** | **493.6** | **756.8** | **8** |

### USB3 2.4GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 48.8 | 88.3 | 5 |
| 2 | 41.2 | 113 | 20 |
| 3 | 51.3 | 135 | 4 |
| 4 | 74.0 | 144 | 5 |
| 5 | 54.3 | 131 | 6 |
| 6 | 47.8 | 133 | 2 |
| 7 | 47.0 | 128 | 5 |
| 8 | 43.8 | 122 | 3 |
| 9 | 66.1 | 135 | 8 |
| 10 | 66.9 | 153 | 4 |
| **Avg** | **54.1** | **128.2** | **62** |

### USB2 2.4GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 124 | 164 | 0 |
| 2 | 134 | 159 | 1 |
| 3 | 135 | 147 | 2 |
| 4 | 126 | 153 | 1 |
| 5 | 127 | 157 | 3 |
| 6 | 127 | 152 | 1 |
| 7 | 123 | 164 | 0 |
| 8 | 122 | 146 | 1 |
| 9 | 111 | 135 | 2 |
| 10 | 105 | 153 | 1 |
| **Avg** | **123.4** | **153.0** | **12** |

### USB2 5GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 251 | 254 | 0 |
| 2 | 232 | 255 | 0 |
| 3 | 222 | 252 | 0 |
| 4 | 248 | 255 | 0 |
| 5 | 200 | 258 | 0 |
| 6 | 91.5 | 252 | 0 |
| 7 | 146 | 257 | 0 |
| 8 | 196 | 256 | 0 |
| 9 | 177 | 254 | 0 |
| 10 | 192 | 256 | 0 |
| **Avg** | **195.6** | **254.9** | **0** |

---

## PATCHED Driver (tx_inflight tracking)

### USB2 5GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 216 | 254 | 0 |
| 2 | 208 | 251 | 0 |
| 3 | 242 | 256 | 0 |
| 4 | 251 | 253 | 0 |
| 5 | 252 | 254 | 0 |
| 6 | 210 | 257 | 0 |
| 7 | 195 | 250 | 0 |
| 8 | 248 | 257 | 0 |
| 9 | 234 | 257 | 0 |
| 10 | 195 | 256 | 0 |
| **Avg** | **225.1** | **254.5** | **0** |

### USB2 2.4GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 145 | 149 | 1 |
| 2 | 143 | 161 | 2 |
| 3 | 87.3 | 158 | 3 |
| 4 | 122 | 148 | 1 |
| 5 | 117 | 154 | 1 |
| 6 | 110 | 145 | 4 |
| 7 | 146 | 160 | 1 |
| 8 | 158 | 159 | 2 |
| 9 | 147 | 146 | 2 |
| 10 | 132 | 137 | 3 |
| **Avg** | **130.7** | **151.7** | **20** |

### USB3 5GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 743 | 740 | 0 |
| 2 | 662 | 771 | 0 |
| 3 | 644 | 754 | 0 |
| 4 | 773 | 746 | 0 |
| 5 | 437 | 765 | 0 |
| 6 | 804 | 722 | 1 |
| 7 | 750 | 758 | 0 |
| 8 | 724 | 789 | 0 |
| 9 | 693 | 728 | 0 |
| 10 | 857 | 761 | 0 |
| **Avg** | **708.7** | **753.4** | **1** |

### USB3 2.4GHz

| Test | Download (Mbps) | Upload (Mbps) | UL Retr |
|------|-----------------|---------------|---------|
| 1 | 75.7 | 129 | 5 |
| 2 | 79.2 | 133 | 15 |
| 3 | 82.7 | 126 | 0 |
| 4 | 84.5 | 138 | 3 |
| 5 | 71.0 | 139 | 12 |
| 6 | 50.5 | 153 | 2 |
| 7 | 54.4 | 136 | 3 |
| 8 | 48.4 | 135 | 4 |
| 9 | 57.7 | 138 | 4 |
| 10 | 70.8 | 138 | 3 |
| **Avg** | **67.5** | **136.5** | **51** |

---

## Comparison Summary

### USB3 5GHz
| Metric | UNPATCHED | PATCHED | Difference |
|--------|-----------|---------|------------|
| Download Avg | 493.6 Mbps | 708.7 Mbps | **+44%** |
| Upload Avg | 756.8 Mbps | 753.4 Mbps | Same |
| Upload Retr | 8 | 1 | **-88%** |

### USB3 2.4GHz
| Metric | UNPATCHED | PATCHED | Difference |
|--------|-----------|---------|------------|
| Download Avg | 54.1 Mbps | 67.5 Mbps | **+25%** |
| Upload Avg | 128.2 Mbps | 136.5 Mbps | +6% |
| Upload Retr | 62 | 51 | -18% |

### USB2 5GHz
| Metric | UNPATCHED | PATCHED | Difference |
|--------|-----------|---------|------------|
| Download Avg | 195.6 Mbps | 225.1 Mbps | **+15%** |
| Upload Avg | 254.9 Mbps | 254.5 Mbps | Same |
| Upload Retr | 0 | 0 | Same |

### USB2 2.4GHz
| Metric | UNPATCHED | PATCHED | Difference |
|--------|-----------|---------|------------|
| Download Avg | 123.4 Mbps | 130.7 Mbps | +6% |
| Upload Avg | 153.0 Mbps | 151.7 Mbps | Same |
| Upload Retr | 12 | 20 | +67% |

---

## Key Findings

1. **USB3 5GHz Download**: PATCHED is **44% faster** (494 → 709 Mbps)
2. **USB3 5GHz Upload Retransmits**: PATCHED has **88% fewer** (8 → 1)
3. **USB3 2.4GHz Download**: PATCHED is **25% faster** (54 → 68 Mbps)
4. **USB2 5GHz Download**: PATCHED is **15% faster** (196 → 225 Mbps)
5. **Upload throughput**: Generally unchanged across all configs
6. **USB2 2.4GHz**: Mixed results, slight regression in retransmits

The patch provides clear download improvements across all configurations, with the biggest gains on USB3 5GHz.

---

*Last updated: 2026-01-29*

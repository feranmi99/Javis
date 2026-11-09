# 📝 SaaS Tiered Pricing & Discount Calculation Logic Rules & Edge Cases
*Documentation Spec for Notes.app / Engineering Knowledge Base*

---

## 1. Executive Overview & Pricing Architecture
The SaaS Pricing Engine provides dynamic, accurate, and predictable cost projections for prospective customers and sales teams. The pricing model incorporates three fundamental dimensions:
1. **Base Tier Plan Subscription** (Base fee + included seats allowance).
2. **Additional Seat Volume Multipliers** (Graduated bracket discount vs. Flat discount).
3. **Billing Frequency Incentives** (Annual / Multi-Year upfront discounts).
4. **Value-Added Service Add-ons** (Fixed infrastructure add-ons & Per-seat add-ons).
5. **Coupons / Promotional Vouchers** (Percentage discount vs. Flat monetary credits).

---

## 2. Tier Plan Definitions

| Plan Tier | Base Monthly Fee | Included Seats | Over-Allowance Rate (per seat/mo) | Target Audience | Key Features Included |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Starter** | $29 | 3 seats | $15 / seat | Indie devs & early startups | Core Analytics, 5 Integrations, Community Support |
| **Growth (Pro)** | $79 | 5 seats | $25 / seat | Fast-scaling teams | Advanced Analytics, 25 Integrations, Priority Support, Custom Domain |
| **Enterprise (Scale)**| $199 | 10 seats | $40 / seat | Mid-to-Large Organizations | Unlimited Analytics, Custom Integrations, 24/7 SLA, Audit Logs, Dedicated CSM |

---

## 3. Seat Calculation & Volume Discount Thresholds

### A. Billable Seats Formula
$$\text{Extra Seats} = \max(0, \text{Total Requested Seats} - \text{Included Seats})$$

### B. Volume Discount Threshold Brackets
Volume discounts apply exclusively to additional seats (or total seat charge depending on configuration):

| Volume Bracket (Total Seats) | Discount Percentage | Tier Objective |
| :--- | :--- | :--- |
| **1 – 10 Seats** | **0%** | Standard entry threshold |
| **11 – 25 Seats** | **10%** | Encourage small team upgrades |
| **26 – 50 Seats** | **15%** | Mid-market volume acceleration |
| **51 – 100 Seats** | **20%** | Corporate departmental rollout |
| **101+ Seats** | **30%** | Enterprise scale incentive |

### C. Discount Model Options
1. **Flat Threshold Mode (Default)**: The discount percentage determined by the total seat count applies to all additional seats.
2. **Graduated / Tiered Bracket Mode (Tax-bracket style)**:
   - Seats 1–10: 0% discount
   - Seats 11–25: 10% discount
   - Seats 26–50: 15% discount
   - Seats 51–100: 20% discount
   - Seats 101+: 30% discount

---

## 4. Billing Frequency Discounts

| Billing Frequency | Multiplier | Discount Incentive | Effective Savings |
| :--- | :--- | :--- | :--- |
| **Monthly** | 1 month | 0% | Standard flexible commitment |
| **Quarterly** | 3 months | 5% | 5% savings |
| **Annual (Default Recommended)** | 12 months | 20% | Equivalent to 2.4 months free |
| **Biennial (2 Years)** | 24 months | 30% | Deep commitment savings |

---

## 5. Add-ons & Extended Services

- **Dedicated IP & Custom SSL**: Fixed fee of $49/month
- **24/7 Priority Support & 99.99% SLA**: Fixed fee of $99/month
- **Advanced Audit Logs & SOC2 Compliance Pack**: Fixed fee of $149/month
- **AI Copilot & Autonomous Agents**: Seat-based add-on at $15/seat/month

---

## 6. Promo Codes / Voucher Rules

| Promo Code | Type | Value | Eligibility / Conditions |
| :--- | :--- | :--- | :--- |
| `LAUNCH25` | Percentage | 25% off subtotal | First-time buyers, stacks with annual discount |
| `STARTUP50` | Fixed Credit | $50 flat credit | Minimum spend of $100 |
| `DEVREL10` | Percentage | 10% off subtotal | Universal developer discount |

---

## 7. Edge Cases & Boundary Handling

1. **Zero or Negative Seat Input**:
   - If a user inputs $0$, $-5$, or non-numeric characters, the input automatically clamps to minimum $1$ seat.
2. **Total Seats Below Plan Allowance**:
   - When Total Seats $\le$ Included Seats, Extra Seats $= 0$. Base Plan cost remains invariant.
3. **Discount Exceeds Total Price**:
   - When fixed vouchers (e.g. `$50` off) exceed the subtotal, the final billable price floors at `$0.00`. It will never generate a negative invoice or payout.
4. **Rounding & Currency Precision**:
   - All intermediate floating-point figures are computed at double precision.
   - Final currency outputs are rounded to 2 decimal places using half-up arithmetic (`Number(val.toFixed(2))`).
5. **High Seat Count (>10,000 Seats)**:
   - UI provides a prompt/badge indicating "Enterprise Custom SLA Available" and flags contact routing.
6. **Local Storage Failure / Quota Exceeded**:
   - LocalStorage operations are wrapped in `try...catch` blocks to protect against private browsing restrictions, corrupted JSON, or `QuotaExceededError`.
7. **Currency Switching (Multi-Currency)**:
   - USD ($), EUR (€), GBP (£), and CAD ($) conversion factors are dynamically calculated without modifying base USD state.

---

## 8. Calculation Formula Sequence

$$\text{Seat Subtotal} = \sum (\text{Over-Allowance Seats} \times \text{Seat Rate} \times (1 - \text{Volume Discount}))$$
$$\text{Addon Subtotal} = \text{Fixed Addons} + (\text{Per-Seat Addons} \times \text{Total Seats})$$
$$\text{Base Monthly Subtotal} = \text{Base Plan Fee} + \text{Seat Subtotal} + \text{Addon Subtotal}$$
$$\text{Billed Subtotal (pre-promo)} = \text{Base Monthly Subtotal} \times (1 - \text{Billing Frequency Discount}) \times \text{Months}$$
$$\text{Final Billed Total} = \max(0, \text{Billed Subtotal} - \text{Promo Discount})$$
$$\text{Effective Monthly Cost} = \frac{\text{Final Billed Total}}{\text{Months}}$$

# Deliverect vs Stream - Cost-Benefit Analysis Framework

**Date:** 2026-01-20
**Purpose:** Quantify financial impact of migration decision
**Owner:** Finance & Engineering

---

## Executive Summary Template

| Metric | 3-Year Total |
|--------|--------------|
| **Total Migration Costs** | $[CALCULATED] |
| **Total Stream Costs (Ongoing)** | $[CALCULATED] |
| **Total Deliverect Costs (Ongoing)** | $[CALCULATED] |
| **Total Benefits from Stream** | $[CALCULATED] |
| **Net Benefit (Cost)** | $[CALCULATED] |
| **ROI %** | [CALCULATED]% |
| **Payback Period** | [CALCULATED] months |
| **NPV (10% discount)** | $[CALCULATED] |

**Recommendation:** [Proceed / Do Not Proceed / More Data Needed]

---

## 1. Migration Costs (One-Time)

### 1.1 Engineering Labor

| Activity | Hours | Rate | Total Cost | Notes |
|----------|-------|------|------------|-------|
| **Discovery & Planning** | | | | |
| API documentation review | [40] | $[150]/hr | $[6,000] | Senior engineer |
| Gap analysis | [80] | $[150]/hr | $[12,000] | Senior + mid-level |
| Migration plan creation | [40] | $[150]/hr | $[6,000] | Senior engineer |
| **Development** | | | | |
| Stream API integration | [160] | $[150]/hr | $[24,000] | 4 weeks, 1 engineer |
| Data migration scripts | [80] | $[120]/hr | $[9,600] | Mid-level engineer |
| Menu sync adapter | [120] | $[120]/hr | $[14,400] | Critical path |
| Harbor service updates | [80] | $[120]/hr | $[9,600] | Integration layer |
| **Testing** | | | | |
| Unit tests | [40] | $[120]/hr | $[4,800] | Standard coverage |
| Integration tests | [80] | $[120]/hr | $[9,600] | Critical |
| End-to-end tests | [40] | $[120]/hr | $[4,800] | Pilot validation |
| Performance testing | [40] | $[120]/hr | $[4,800] | Sync latency |
| **Deployment** | | | | |
| Pilot deployment | [40] | $[150]/hr | $[6,000] | DevOps + eng |
| Monitoring setup | [40] | $[120]/hr | $[4,800] | Observability |
| Rollback plan | [20] | $[150]/hr | $[3,000] | Safety net |
| Full rollout | [80] | $[150]/hr | $[12,000] | Phased approach |
| **Documentation** | | | | |
| Technical documentation | [40] | $[120]/hr | $[4,800] | Runbooks |
| User guides | [20] | $[100]/hr | $[2,000] | Operations team |
| **SUBTOTAL** | **1,020** | | **$[138,000]** | |

**Formula:** `Total Engineering Cost = Σ(Hours × Hourly Rate)`

### 1.2 External Costs

| Item | Cost | Notes |
|------|------|-------|
| Stream setup/onboarding fee | $[TBD] | Ask Stream sales |
| Deliverect contract termination | $[TBD] | Check contract terms |
| Consultant fees (if needed) | $[TBD] | Optional |
| Legal review | $[5,000] | Contract review |
| **SUBTOTAL** | **$[TBD]** | |

### 1.3 Risk Contingency

| Risk Item | Probability | Impact | Expected Cost |
|-----------|-------------|--------|---------------|
| Migration takes 50% longer | 40% | $69,000 | $[27,600] |
| Data migration issues | 30% | $20,000 | $[6,000] |
| Performance issues require rework | 20% | $30,000 | $[6,000] |
| Rollback required | 10% | $50,000 | $[5,000] |
| **SUBTOTAL** | | | **$[44,600]** |

**Formula:** `Risk Cost = Probability × Impact Cost`

### 1.4 Opportunity Cost

| Item | Cost | Notes |
|------|------|-------|
| Features not built (engineering time) | $[100,000] | What could we build instead? |
| Revenue lost during transition | $[TBD] | Estimate downtime impact |
| **SUBTOTAL** | **$[TBD]** | |

---

### **TOTAL MIGRATION COSTS** = **$[182,600 + TBD]**

---

## 2. Ongoing Platform Costs (Annual)

### 2.1 Deliverect Costs (Current State)

| Cost Category | Monthly | Annual | Notes |
|---------------|---------|--------|-------|
| **Platform Fees** | | | |
| Per-location fee | $[TBD] | $[TBD] | Ask current contract |
| Per-order fee | $[TBD] | $[TBD] | Transaction-based? |
| Base subscription | $[TBD] | $[TBD] | Monthly SaaS fee |
| **Support & Maintenance** | | | |
| Engineering support (5% time) | $[750] | $[9,000] | Minimal, already integrated |
| Infrastructure costs | $[100] | $[1,200] | Harbor service allocation |
| **SUBTOTAL** | **$[TBD]** | **$[TBD]** | |

### 2.2 Stream Costs (Projected)

| Cost Category | Monthly | Annual | Notes |
|---------------|---------|--------|-------|
| **Platform Fees** | | | |
| Per-location fee | $[TBD] | $[TBD] | Get Stream pricing |
| Per-order fee | $[TBD] | $[TBD] | Estimate based on volume |
| Base subscription | $[TBD] | $[TBD] | Monthly SaaS fee |
| White label (if used) | $[30] | $[360] | Optional feature |
| Premium features | $[TBD] | $[TBD] | Auto throttle, monitoring |
| **Support & Maintenance** | | | |
| Engineering support (10% time) | $[1,500] | $[18,000] | Higher in first year |
| Infrastructure costs | $[200] | $[2,400] | Additional APIs |
| **SUBTOTAL** | **$[TBD]** | **$[TBD]** | |

### 2.3 Cost Comparison (3-Year Projection)

| Platform | Year 1 | Year 2 | Year 3 | Total |
|----------|--------|--------|--------|-------|
| **Deliverect** | $[TBD] | $[TBD] | $[TBD] | **$[TBD]** |
| **Stream** | $[TBD] | $[TBD] | $[TBD] | **$[TBD]** |
| **Difference** | $[TBD] | $[TBD] | $[TBD] | **$[TBD]** |

**Note:** Adjust for expected growth in number of locations and order volume.

---

## 3. Quantifiable Benefits (Annual)

### 3.1 Auto Throttling Benefit

**Current Problem:**
- Kitchen overwhelm leads to: slower prep times, quality issues, delayed orders, bad reviews
- Manual channel pausing is reactive, not proactive

**Metrics to Measure:**
| Metric | Current | With Stream | Improvement |
|--------|---------|-------------|-------------|
| Overwhelm incidents/month | [20] | [8] | 60% reduction |
| Orders during overwhelm | [100] | [40] | 60 orders saved |
| Bad reviews from overwhelm | [15] | [6] | 9 reviews saved |
| Average order value | $[35] | $[35] | - |

**Calculation:**
```
Monthly Value = (Orders Saved × Order Value × Customer Lifetime Multiplier)
              + (Reviews Saved × Review Impact Value)

Orders Saved Value:
= 60 orders × $35 × 3 (LTV multiplier)
= $6,300/month = $75,600/year

Review Impact Value:
= 9 reviews × $500 (estimated customer acquisition cost saved)
= $4,500/month = $54,000/year

TOTAL AUTO THROTTLING BENEFIT = $129,600/year
```

**Conservative Estimate:** $[75,000]/year (50% of calculated)

### 3.2 Store Monitoring Benefit

**Current Problem:**
- Store outages on platforms go undetected for hours
- Lost orders during outage period
- Customer frustration from "ghost" restaurant

**Metrics to Measure:**
| Metric | Current | With Stream | Improvement |
|--------|---------|-------------|-------------|
| Avg outage detection time | 120 min | 15 min | 87.5% faster |
| Outages per month | [4] | [4] | Same occurrence |
| Orders lost per outage | [50] | [6] | 44 orders saved |
| Average order value | $[35] | $[35] | - |

**Calculation:**
```
Monthly Value = (Outages × Orders Saved × Order Value × LTV)

= 4 outages × 44 orders × $35 × 3 (LTV)
= $18,480/month = $221,760/year

TOTAL STORE MONITORING BENEFIT = $221,760/year
```

**Conservative Estimate:** $[150,000]/year (68% of calculated)

### 3.3 Multi-Channel Pricing Benefit

**Current Problem:**
- Same price across all platforms despite different commission rates
- DoorDash: 25-30% commission
- Uber Eats: 25-30% commission
- Grubhub: 20-25% commission
- Can't optimize margin by platform

**Metrics to Measure:**
| Platform | Orders/Month | Current Price | Optimal Price | Margin Gain/Order |
|----------|--------------|---------------|---------------|-------------------|
| DoorDash (30% comm) | [2,000] | $35 | $38 | $3 |
| Uber Eats (28% comm) | [1,500] | $35 | $37 | $2 |
| Grubhub (22% comm) | [1,000] | $35 | $35 | $0 |

**Calculation:**
```
Monthly Value = Σ(Orders × Margin Gain)

= (2,000 × $3) + (1,500 × $2) + (1,000 × $0)
= $6,000 + $3,000 + $0
= $9,000/month = $108,000/year

TOTAL MULTI-CHANNEL PRICING BENEFIT = $108,000/year
```

**Conservative Estimate:** $[75,000]/year (70% of calculated, assumes price sensitivity)

### 3.4 Item Snoozing Benefit

**Current Problem:**
- Manual menu updates for daily items (bakeries, limited items)
- Risk of selling out-of-stock items
- Time spent on menu updates

**Metrics to Measure:**
| Metric | Current | With Stream | Improvement |
|--------|---------|-------------|-------------|
| Time spent on menu updates | 20 min/day | 5 min/day | 15 min saved |
| Labor cost per hour | $[20] | $[20] | - |
| Out-of-stock orders avoided | 5/week | 1/week | 4 orders saved |

**Calculation:**
```
Time Savings:
= 15 min/day × 365 days × ($20/hour ÷ 60 min)
= $1,825/year

Out-of-Stock Avoidance:
= 4 orders/week × 52 weeks × $35 × 0.5 (refund/credit cost)
= $3,640/year

TOTAL ITEM SNOOZING BENEFIT = $5,465/year
```

**Conservative Estimate:** $[5,000]/year

### 3.5 Engineering Efficiency

**Current Problem:**
- Engineering time spent on manual interventions
- Ad-hoc fixes for menu sync issues

**Metrics to Measure:**
| Metric | Current | With Stream | Improvement |
|--------|---------|-------------|-------------|
| Eng hours on menu issues | 10 hrs/month | 5 hrs/month | 5 hrs saved |
| Hourly rate (blended) | $[120] | $[120] | - |

**Calculation:**
```
Annual Savings = 5 hrs/month × 12 months × $120/hr
               = $7,200/year
```

**Conservative Estimate:** $[7,000]/year

---

### **TOTAL ANNUAL BENEFITS** = **$[312,000]** (conservative)

| Benefit Category | Amount | % of Total |
|------------------|--------|------------|
| Auto Throttling | $75,000 | 24% |
| Store Monitoring | $150,000 | 48% |
| Multi-Channel Pricing | $75,000 | 24% |
| Item Snoozing | $5,000 | 2% |
| Engineering Efficiency | $7,000 | 2% |

---

## 4. Non-Quantifiable Benefits

| Benefit | Impact Level | Notes |
|---------|--------------|-------|
| Improved customer experience | 🟡 Medium | Fewer overwhelm situations |
| Better kitchen morale | 🟡 Medium | Less stress during peak |
| Broader platform options | 🟢 Low | Future expansion capability |
| Competitive advantage | 🟢 Low | Better operational control |
| Brand reputation | 🟡 Medium | Fewer bad reviews |

---

## 5. Risks & Hidden Costs

### 5.1 Migration Risks

| Risk | Probability | Financial Impact | Mitigation |
|------|-------------|------------------|------------|
| Extended downtime | 20% | -$50,000 | Phased rollout, rollback plan |
| Feature parity gaps | 30% | -$30,000 | Thorough pre-migration testing |
| Performance degradation | 15% | -$20,000 | Pilot phase measurement |
| Customer complaints | 25% | -$15,000 | Communication plan |
| Staff training issues | 10% | -$5,000 | Training program |

**Expected Risk Cost:** $[26,250]

### 5.2 Ongoing Risks

| Risk | Annual Impact | Mitigation |
|------|---------------|------------|
| Stream platform outages | -$[10,000] | SLA guarantees, multi-vendor strategy |
| Price increases | -$[5,000] | Multi-year contract |
| Feature deprecation | -$[3,000] | Regular vendor check-ins |

**Expected Annual Risk:** $[18,000]

---

## 6. Financial Analysis

### 6.1 3-Year Cash Flow Projection

| Period | Costs | Benefits | Net Cash Flow | Cumulative |
|--------|-------|----------|---------------|------------|
| **Migration (Month 0)** | -$182,600 | $0 | -$182,600 | -$182,600 |
| **Year 1** | -$[TBD Stream] | $312,000 | $[TBD] | $[TBD] |
| **Year 2** | -$[TBD Stream] | $343,200 | $[TBD] | $[TBD] |
| **Year 3** | -$[TBD Stream] | $377,520 | $[TBD] | $[TBD] |

**Assumptions:**
- Benefits grow 10% annually (more locations, higher order volume)
- Stream costs remain flat (locked pricing)
- Deliverect costs avoided in Year 1+

### 6.2 ROI Calculation

```
ROI Formula:
ROI = (Total Benefits - Total Costs) / Total Costs × 100%

Example with placeholder numbers:
Total Benefits (3 years) = $1,032,720 ($312k + $343k + $378k)
Total Costs (3 years) = $182,600 + ($150,000 × 3) = $632,600
Net Benefit = $1,032,720 - $632,600 = $400,120

ROI = $400,120 / $632,600 × 100% = 63.3%
```

**Target ROI:** > 50% over 3 years

### 6.3 Payback Period

```
Payback Period = Initial Investment / Annual Net Benefit

Example:
Initial Investment = $182,600
Annual Net Benefit = $312,000 - $150,000 (Stream costs) = $162,000

Payback Period = $182,600 / $162,000 = 1.13 years (13.5 months)
```

**Target Payback:** < 18 months

### 6.4 Net Present Value (NPV)

```
NPV Formula:
NPV = Σ [Cash Flow / (1 + r)^t] - Initial Investment

Where:
r = discount rate (10%)
t = year

Year 0: -$182,600 / (1.1)^0 = -$182,600
Year 1: +$162,000 / (1.1)^1 = +$147,273
Year 2: +$178,200 / (1.1)^2 = +$147,273
Year 3: +$196,020 / (1.1)^3 = +$147,273

NPV = -$182,600 + $147,273 + $147,273 + $147,273 = $259,219
```

**Target NPV:** > $0 (positive value creation)

### 6.5 Break-Even Analysis

```
Break-Even Point = Fixed Costs / (Benefit per Unit - Variable Cost per Unit)

For this analysis:
Fixed Costs = $182,600 (migration)
Monthly Benefit = $26,000 ($312k / 12)
Monthly Stream Cost = $12,500 (estimated)
Net Monthly Benefit = $13,500

Break-Even Months = $182,600 / $13,500 = 13.5 months
```

---

## 7. Sensitivity Analysis

### 7.1 What if benefits are 50% of estimate?

| Scenario | 3-Year Benefits | ROI | Payback |
|----------|-----------------|-----|---------|
| **Base Case** | $1,032,720 | 63% | 13.5 mo |
| **50% Benefits** | $516,360 | -18% | Never |

**Conclusion:** Need at least 50% of estimated benefits for positive ROI

### 7.2 What if migration costs are 2x?

| Scenario | Migration Cost | ROI | Payback |
|----------|----------------|-----|---------|
| **Base Case** | $182,600 | 63% | 13.5 mo |
| **2x Cost** | $365,200 | 38% | 27 mo |

**Conclusion:** Still positive ROI but longer payback

### 7.3 What if Stream costs 50% more?

| Scenario | Annual Stream Cost | ROI | Payback |
|----------|-------------------|-----|---------|
| **Base Case** | $150,000 | 63% | 13.5 mo |
| **50% Higher** | $225,000 | 20% | 24 mo |

**Conclusion:** ROI drops significantly with higher ongoing costs

---

## 8. Decision Matrix

### 8.1 Go/No-Go Criteria

| Criterion | Threshold | Actual | Pass? |
|-----------|-----------|--------|-------|
| ROI > 50% (3 years) | 50% | [TBD]% | [✅/❌] |
| Payback < 18 months | 18 mo | [TBD] mo | [✅/❌] |
| NPV > $0 | $0 | $[TBD] | [✅/❌] |
| Auto throttle benefit | $50,000 | $[TBD] | [✅/❌] |
| Store monitoring benefit | $100,000 | $[TBD] | [✅/❌] |
| Feature parity confirmed | 100% | [TBD]% | [✅/❌] |
| Stream costs acceptable | < $200k/yr | $[TBD] | [✅/❌] |

**Decision Rule:** Must pass ALL criteria to proceed

### 8.2 Recommendation Template

Based on the analysis:

**Total 3-Year Value:**
- Migration Cost: $[TBD]
- Ongoing Cost Difference: $[TBD]
- Total Benefits: $[TBD]
- Net Benefit: $[TBD]
- ROI: [TBD]%
- Payback: [TBD] months

**Recommendation:** [CHOOSE ONE]

✅ **PROCEED** - Strong financial case, benefits exceed costs
- Conditions: [List any prerequisites]
- Timeline: [Proposed timeline]
- Next Steps: [Immediate actions]

🟡 **PILOT FIRST** - Promising but uncertain
- Risks to validate: [List key uncertainties]
- Success criteria: [Pilot metrics]
- Timeline: [3-6 month pilot]

❌ **DO NOT PROCEED** - Insufficient ROI
- Key issues: [List problems]
- Alternative: [What to do instead]
- Revisit when: [Conditions that would change decision]

---

## 9. Data Collection Checklist

### Required from Stream
- [ ] Per-location monthly fee
- [ ] Per-order transaction fee (if any)
- [ ] Base subscription cost
- [ ] Premium feature costs (throttling, monitoring)
- [ ] Setup/onboarding fees
- [ ] Contract terms (length, cancellation)
- [ ] SLA guarantees (uptime, support response)

### Required from Current Operations
- [ ] Current Deliverect costs (all fees)
- [ ] Number of locations
- [ ] Monthly order volume per platform
- [ ] Current overwhelm incident frequency
- [ ] Current store outage frequency and duration
- [ ] Average order value
- [ ] Customer lifetime value
- [ ] Engineering time spent on menu issues

### Required from Finance
- [ ] Discount rate for NPV calculation
- [ ] Budgeted engineering hourly rates
- [ ] Customer acquisition cost
- [ ] Review impact valuation
- [ ] Risk tolerance level

---

## 10. Next Steps

1. **Fill in [TBD] values** - Collect data from Stream, Finance, Operations
2. **Calculate all formulas** - Complete financial analysis
3. **Review with stakeholders** - Present findings to leadership
4. **Make recommendation** - Go/No-Go/Pilot decision
5. **If Go:** Create detailed migration plan
6. **If No-Go:** Document decision and revisit criteria
7. **If Pilot:** Define pilot success metrics and timeline

---

## Appendix: Calculation Worksheets

### Worksheet A: Auto Throttling Benefit

```
Inputs:
- Current overwhelm incidents/month: _______
- Expected incidents with Stream: _______
- Orders affected per incident: _______
- Average order value: $_______
- Customer LTV multiplier: _______
- Bad reviews per incident: _______
- Cost per bad review: $_______

Calculation:
Orders Saved = (Current - Expected) × Orders per Incident × 12
Review Impact = (Current - Expected) × Reviews per Incident × 12

Annual Benefit = (Orders Saved × AOV × LTV) + (Reviews × Cost)
               = $_______
```

### Worksheet B: Store Monitoring Benefit

```
Inputs:
- Current outage detection time: _______ minutes
- Stream outage detection time: _______ minutes
- Outages per month: _______
- Orders lost per outage hour: _______
- Average order value: $_______
- Customer LTV multiplier: _______

Calculation:
Time Saved = (Current - Stream) / 60 hours
Orders Saved = Time Saved × Orders per Hour
Monthly Impact = Orders Saved × AOV × LTV × Outages

Annual Benefit = Monthly Impact × 12
               = $_______
```

### Worksheet C: Multi-Channel Pricing Benefit

```
Platform 1 (DoorDash):
- Orders/month: _______
- Commission: _______%
- Price increase opportunity: $_______
- Monthly gain: _______

Platform 2 (Uber Eats):
- Orders/month: _______
- Commission: _______%
- Price increase opportunity: $_______
- Monthly gain: _______

Platform 3 (Grubhub):
- Orders/month: _______
- Commission: _______%
- Price increase opportunity: $_______
- Monthly gain: _______

Total Monthly Gain: $_______
Annual Benefit: $_______ × 12 = $_______
```

---

**Document Owner:** Finance Team & Engineering Lead
**Last Updated:** 2026-01-20
**Review Schedule:** Update after data collection, pilot (if applicable), quarterly
**Related Docs:**
- `deliverect-stream-executive-summary.md` (leadership summary)
- `deliverect-stream-capability-matrix.md` (feature comparison)
- `deliverect-stream-migration-roadmap.md` (implementation plan)

---
title: "Risk Formulas and Quantitative Analysis"
layer: 2
domain: 1
tags:
  - cissp
  - domain-01
  - risk-formulas
  - ale
  - sle
  - aro
  - fair
  - quantitative-risk
related_domains:
  - "[[risk-management-frameworks]]"
  - "[[security-governance-policies-standards]]"
  - "[[business-continuity-and-disaster-recovery]]"
  - "[[gdpr-deep-dive]]"
  - "[[pci-dss-4-0]]"
---

# Risk Formulas and Quantitative Analysis

## Feynman Explanation

The math of risk is just multiplication: how much can we lose, and how often will we lose it? Once you have a number, you can answer the only question the CFO cares about: "Is this control worth what we're paying for it?" Without this math, security spending is a mood ring — it changes with whoever spoke last in the meeting.

## Technical Details

### Qualitative vs Quantitative

| Aspect | Qualitative | Quantitative |
|---|---|---|
| Scale | High/Medium/Low or 1-5 | Dollar values, probabilities |
| Audience | Executives, broad stakeholders | CFOs, insurance, board |
| Cost to produce | Low | High (data collection) |
| Defensibility | Weak ("who said it's 'high'?") | Strong ("our ALE is $X") |
| Best use | Quick triage, threat identification | Investment decisions, insurance |

Most mature programs use **both** — qualitative to prioritize, quantitative for the top-tier risks.

### Classic NIST/ISO Formulas

#### Single Loss Expectancy (SLE)

$$SLE = Asset\ Value\ (AV) \times Exposure\ Factor\ (EF)$$

- **AV** = dollar value of the asset (a laptop = $2,000; a customer DB = $5M)
- **EF** = percentage of the asset lost in a single incident, expressed as a decimal (e.g., 0.6 for 60% destruction)

**Example:** A $5,000 server with a ransomware event that destroys 80% of its data:
$$SLE = \$5{,}000 \times 0.80 = \$4{,}000$$

#### Annual Rate of Occurrence (ARO)

The number of times per year the threat is expected to materialize. A 1-in-10-year event has ARO = 0.1; a weekly event has ARO = 52.

#### Annual Loss Expectancy (ALE)

$$ALE = SLE \times ARO$$

The expected loss per year from a given risk. This is the "should we spend anything on this?" number.

**Example continued:** If ransomware hits the same server cluster once every 4 years (ARO = 0.25):
$$ALE = \$4{,}000 \times 0.25 = \$1{,}000\ /\ year$$

#### Cost-Benefit Analysis (CBA) for Controls

$$ACS = ALE_{before} - ALE_{after} - Annual\ Cost\ of\ Control$$

Where ACS = Annual Cost Savings. **Deploy the control if and only if ACS ≥ 0.**

**Example:** Same server. Current ALE = $1,000/year. Adding EDR + immutable backups:
- New ALE = $100/year (90% reduction, 0.025 ARO)
- Cost of control = $400/year

$$ACS = \$1{,}000 - \$100 - \$400 = \$500\ /\ year\ (positive\ =>\ deploy)$$

#### Return on Security Investment (ROSI)

$$ROSI = \frac{ALE_{before} - ALE_{after} - Control\ Cost}{Control\ Cost}$$

$$ROSI = \frac{ACS}{Control\ Cost}$$

Same example:
$$ROSI = \frac{\$500}{\$400} = 1.25 = 125\%$$

Some texts use the (ALE - ACS) / Cost form. Both express the same idea: dollars of risk reduced per dollar of control spent.

### Total Cost of Ownership (TCO) for Security

Don't forget the hidden costs of a control:
- Acquisition / licensing
- Implementation / integration
- Training (people)
- Operations / monitoring
- Decommissioning
- User productivity loss
- Audit / compliance evidence collection

A control with the best CBA on paper can be net-negative if it locks up a security team for 6 months.

### FAIR Quantitative Model

FAIR (Factor Analysis of Information Risk) breaks risk into 4 primary factors:

$$Risk = Loss\ Event\ Frequency\ (LEF) \times Loss\ Magnitude\ (LM)$$

#### Loss Event Frequency (LEF)

$$LEF = Threat\ Event\ Frequency\ (TEF) \times Vulnerability\ (V)$$

- **TEF** = how often the threat acts (e.g., phishing emails per year)
- **V** = probability (0.0 to 1.0) that the threat succeeds against the asset

#### Loss Magnitude (LM)

$$LM = Primary\ Loss\ (PL) + Secondary\ Loss\ (SL)$$

- **Primary Loss** = direct costs (asset replacement, productivity loss, response cost)
- **Secondary Loss** = indirect costs (regulatory fines, reputational damage, IP theft, competitive disadvantage)

Secondary loss is the **boardroom conversation** — it's where a $200K breach becomes a $50M one.

#### FAIR's strength

It produces a **probabilistic distribution** (e.g., "annual loss exposure 90% confidence: $4.2M - $9.8M"), not a point estimate. This is what cyber insurers and CFOs actually want to plan against.

### Annualized Rate of Occurrence Examples

| Event | Frequency | ARO |
|---|---|---|
| Earthquake in San Jose | ~1 in 100 years | 0.01 |
| Major hurricane on FL coast | ~1 in 7 years | 0.14 |
| Power outage (regional) | ~2 per year | 2.0 |
| Phishing email received | ~200 per year | 200 |
| Employee laptop theft | ~1 per 4 years for a 1,000-laptop fleet | ~250 |
| Ransomware on a specific server | Once every 4 years | 0.25 |

### A Worked Example (Board-Level)

**Scenario:** Customer credit-card database breach.
- AV = $50M (regulatory fine, notification, brand damage modeled)
- EF = 0.6 (60% of records lost)
- ARO = 0.25 (we believe one such breach every 4 years)

$$SLE = \$50M \times 0.6 = \$30M$$
$$ALE = \$30M \times 0.25 = \$7.5M/year$$

**Proposed control:** Tokenization + encryption-at-rest + segmentation.
- Expected ARO reduction: 0.25 → 0.05 (80% effective)
- New ALE = $30M × 0.05 = $1.5M/year
- Annual control cost: $2.0M

$$ACS = \$7.5M - \$1.5M - \$2.0M = \$4.0M/year\ (deploy)$$

$$ROSI = \frac{\$4.0M}{\$2.0M} = 200\%$$

This is the kind of calculation that gets a budget approved.

## CISO / Risk Manager View

**Three rules for using this math well:**

1. **Never present a single number.** Present a range. Single-point estimates will be ripped apart the first time reality deviates. A range ("$4M-$10M annual exposure") is defensible.
2. **Always separate the primary from the secondary loss.** The board will accept "the laptop costs $2,000" but it is the GDPR fine, customer churn, and stock drop that matter. Force the conversation about secondary loss.
3. **Pair ALE with TCO.** The cheapest-looking control often has the highest hidden cost. The most expensive-looking control often has the best TCO.

**What the board actually buys:**

| Question | Formula they expect | What they really want |
|---|---|---|
| "Are we secure?" | — | A clear risk appetite statement |
| "How much should we spend?" | ROSI / NPV | A defensible budget with a 3-year risk trajectory |
| "Should we buy this tool?" | CBA | "Yes, saves $X, costs $Y, payback Z months" |
| "Are we getting better?" | ALE trend | A chart showing risk going down over time |

The CISSP exam rewards the same discipline: when given a question with numbers, write the formula first, then plug in.

## Related Connections

### Sibling L2
- [[risk-management-frameworks]] - Frameworks that consume these formulas
- [[security-governance-policies-standards]] - Risk acceptance documentation
- [[business-continuity-and-disaster-recovery]] - Downtime cost in ALE

### L3
- [[gdpr-deep-dive]] - Secondary loss model includes GDPR fines up to 4% revenue
- [[pci-dss-4-0]] - PCI fines are an ALE input

### Cross-Domain
- [[domain-02-asset-security]] - Asset valuation feeds AV
- [[domain-07-security-operations]] - Actual incident frequency recalibrates ARO
- [[domain-08-software-development-security]] - AppSec defect rates are an ARO input

## Sources / References

- NIST SP 800-30 Rev. 1 - Guide for Conducting Risk Assessments
- NIST SP 800-39 - Managing Information Security Risk
- ISO/IEC 27005:2022
- Open Group FAIR Standard - https://www.opengroup.org/certifications/142-1
- (ISC)² CISSP CBK, 2024
- Douglas W. Hubbard - "How to Measure Anything in Cybersecurity Risk"
- Jack Jones - "Measuring and Managing Information Risk: A FAIR Approach"

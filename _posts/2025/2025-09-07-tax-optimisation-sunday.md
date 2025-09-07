---
title: "The 6:00 a.m. Tax Realisation (And the Tool That Turns Dread Into Strategy)"
categories:
  - GitHub Copilot
  - Personal Finance
tags:
  - AI
  - Productivity
  - Vibe coding
  - Tax
---

It’s a grey British Sunday. Kettle on. Everyone else in the house is asleep. You open your payslip (again) and feel that quiet, familiar annoyance: *“I earn how much—and keep how little?”*  

You’ve heard of the 40% band, maybe even the 45%. But nobody warned you about the invisible cliffs:  

- The personal allowance taper past £100k.  
- That weird effective 60% zone.  

You’re not reckless. You’re just busy. And the system’s opacity quietly taxes that, too. Let's do some vibe coding to understand this mess

---

## Why This Project Exists
This project was born at 6:00 a.m.—a small rebellion against guesswork. Not a spreadsheet beast. A clear, visual UK tax optimisation companion that answers three deceptively simple questions:  

1. Where’s my money actually going?  
2. What am I losing that I could legally keep?  
3. What levers change the outcome fastest?  

---

## The Problem: Fog, Not Numbers
Most people can quote their gross salary. Fewer know:  

- Their true average vs marginal effective rate.  
- How much tax + NI + tapered allowances quietly remove.  
- Whether a pension contribution would recover lost personal allowance or retain Child Benefit.  
- That salary sacrifice (pension / EV / cycle) can save NI as well as tax.  

We don’t have a shortage of calculators. We have a shortage of clarity.  

---

## The Experience
Instead of dumping a number at you, the tool stacks your whole pre-sacrifice gross into coloured bands:  

- **Payments to government** (Income Tax, NI, Child Benefit charge)  
- **Redirected** (salary sacrifice, pension)  
- **What you genuinely take home**  

You drag a pension percentage and instantly watch tax shrink, marginal cliffs flatten, effective relief rates jump. Add a salary sacrifice (future EV or cycle) and you see the difference—not in abstract percentages—but in reclaimed annual pounds.  

It reframes pension from *“money I lose today”* to *“money I redirect while unlocking tax efficiency.”*  

![Main Visualisation](/assets/tax-app/web-main-visual.png)

---

## Key Moments It Surfaces
- “That extra bonus pushes me into losing £X of personal allowance.”  
- “Another 3% pension here is effectively 55–65% *‘off’* because it restores lost allowances.”  
- “My NI hasn’t moved—right, because this is a straight employee contribution, not sacrifice.”  
- “If I sacrifice instead, both tax and NI fall. That’s real cash back.”  

---

## Why This Matters (Beyond the Numbers)
Understanding your tax position:  

- Reduces anxiety (uncertainty is a tax on attention)  
- Improves negotiation (know what part of a raise you’ll actually keep)  
- Accelerates long-term wealth (high-relief pension £ buys more future optionality)  
- Protects family benefits (Child Benefit erosion is stealthy)  
- Encourages intentional trade-offs (EV vs cash allowance vs bonus timing)  

---

## A Glimpse Under the Hood (Light Touch)
A pure calculation engine crunches:  

- Personal allowance taper (£1 lost per £2 over £100k)  
- Child Benefit charge (1% per £200 in the £60k–£80k band)  
- NIC bands  
- Pension effect on taxable income (employee) vs NI-able income (sacrifice)  

Then it generates a composable JSON breakdown feeding stacked visual bars and savings summaries. No spreadsheets. No black box.  

Here are the tax rules used by the engine - let me know if you think there is anything wrong

```typescript
// Centralized thresholds & rates (easier future updates)
// NOTE: Adjust once official 2025/26 figures confirmed.
export interface IncomeTaxBand {
  from: number;
  to?: number;
  rate: number; // e.g. 0.2
}

export const PERSONAL_ALLOWANCE = 12_570;
export const PERSONAL_ALLOWANCE_TAPER_START = 100_000;
export const PERSONAL_ALLOWANCE_TAPER_LOSS_PER = 2; // lose £1 per £2
export const PERSONAL_ALLOWANCE_FLOOR = 0;

// Bands here are applied to taxable income AFTER personal allowance.
// We express band spans relative to 0 taxable amount after PA.
export const INCOME_TAX_BANDS: IncomeTaxBand[] = [
  { from: 0, to: 37_700, rate: 0.2 },          // Basic (after allowance)
  { from: 37_700, to: 125_140 - PERSONAL_ALLOWANCE, rate: 0.4 }, // Higher
  { from: 125_140 - PERSONAL_ALLOWANCE, rate: 0.45 } // Additional
];

// National Insurance (Employee, Class 1) – assumed continuing rates
export const NI_PRIMARY_THRESHOLD = 12_570;
export const NI_UPPER_EARNINGS_LIMIT = 50_270;
export const NI_MAIN_RATE = 0.08; // 8%
export const NI_ADDITIONAL_RATE = 0.02; // 2%

// Child Benefit amounts (assumed carryover; update when official)
export const CHILD_BENEFIT_FIRST_CHILD_WEEKLY = 25.60;
export const CHILD_BENEFIT_ADDITIONAL_CHILD_WEEKLY = 16.95;
export const CHILD_BENEFIT_YEAR_WEEKS = 52;

// High Income Child Benefit Charge updated thresholds
export const CHILD_BENEFIT_CHARGE_START = 60_000;
export const CHILD_BENEFIT_CHARGE_END = 80_000; // fully removed by here

export interface TaxSystemConfig {
  taxYearLabel: string;
}

export const CONFIG: TaxSystemConfig = {
  taxYearLabel: '2025/26 (assumed)'
};

```

---

## Common “Aha” Examples
- **£115k salary, modest pension** → allowance taper spike. Increase pension % → reclaim allowance, huge relief.  
- **Mid-£60k earner with two kids** → Child Benefit leakage. A tweak in sacrifice brings back family cash flow.  
- **EV salary sacrifice modelling (coming next)** → Net cost far lower once Tax + NI + low BiK rate is shown.  

### Dig into the £115k Salary Example

Let’s walk through a classic “aha” scenario:  
Suppose your salary is £115,000 and you’re making only 5% pension contribution. At this level, you’re well into the personal allowance taper zone—meaning for every £2 you earn above £100,000, you lose £1 of your tax-free allowance. The result? Your effective tax rate on this band can spike to 60% or more. Just look at those numbers

![115K with 5% Pension](/assets/tax-app/web-115K-5.png)

Now, use the tool to increase your pension contribution by say 10 percent. Instantly, you’ll see:

- Your taxable income drops below the taper threshold.
- Your personal allowance is fully restored.
- The tax you “save” is far greater than the pension amount you contribute—sometimes over half of each extra pound.
- Take home has taken a small drop but look at that pension contribution. You'll be happy about that in the years to come

![115K with 15% Pension](/assets/tax-app/web-115K-15.png)



---

## This Isn’t Aggressive Tax Avoidance
It’s simply understanding the rules as written and using them deliberately:  

- Pension contributions (within annual allowance)  
- Salary sacrifice (properly documented)  
- Child Benefit preservation  
- Clear visibility of trade-offs  

---

## What It’s Not (Yet)
- Dividend layering  
- Share scheme optimisation  
- Student loan stacking  
- Gift Aid interplay  
- Scottish/Welsh rate divergence  

Those will come—once the core stays fast and trustworthy.  

---

## Why Build It Now?
Because the marginal pound of energy you spend before breakfast shouldn’t be wasted decoding taper math.  
Because telling your future self *“I just never looked”* is a rubbish strategy.  
And because the system won’t get simpler—so tooling must get clearer.  

---

## Call to Action
Try it. Move one slider. Feel that flicker of control return. Then decide:  

- Do I redirect more now?  
- Do I restructure my bonus?  
- Should I push HR for salary sacrifice on the upcoming scheme?  

Clarity compounds—financially and mentally.  

---

## Footnote (Quiet but Important)
This is informational, not personal tax advice. Complex cases (tapered annual pension allowance, MPAA, adjusted income > £260k, etc.) need a professional check. But you deserve transparency before you reach for paid advice.  

Want to see EV and cycle-to-work modelling appear next? That’s on the roadmap.  

Let’s retire *“I think?”* from every UK tax sentence we say this year.  

*(And yes—it really did start at 6:00 a.m. while the house was still half asleep.)*  

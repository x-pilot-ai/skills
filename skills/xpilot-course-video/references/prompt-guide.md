# Prompt guide

Content first. The agent plans scenes and visuals from what you write, so the brief should carry the substance.

## Rules for every type

- Start with audience and goal in one or two lines.
- Use `#` for the title and `##` for each section. One `##` section ≈ one scene. Keep a single video to about 12 scenes.
- Each section: 3–6 lines of real content. Facts, numbers, formulas, dates, names, ordered steps, comparisons.
- Do not specify components, layouts or animations ("bar chart", "split screen"). Describe the content; the agent picks the visual.
- Never invent data. Only include numbers the user gave or that come from uploaded files.
- One video, one topic. Split large topics into a series of conversations.

## Choose a type

| Type | Use for | video_style | video_duration |
|---|---|---|---|
| Course lecture | University or vocational lessons, concept explanations | `clean_lecture` | `deep` |
| Science explainer | Physics, chemistry, biology processes | `science_explainer` | `deep` |
| Workplace training | Policies, compliance, procedures, onboarding | `professional_training` | `short` |
| Product showcase | Features, launches, sales demos | `product_showcase` | `short` |
| Story | History, biographies, brand stories | `cinematic_story` | `deep` |
| Document to video | Papers, specs or slides turned into a video | `auto` | `deep` |

Durations: `short` 1–3 min, `deep` 3–10 min, `long` 10–20 min. Use `auto` style when unsure.

## Skeletons

### Course lecture

```
# Price elasticity of demand
Audience: first-year economics students. Goal: calculate and interpret elasticity.

## Hook
Why do some products sell the same amount after a price rise?

## Definition and formula
Ed = %ΔQ / %ΔP. |Ed| > 1 is elastic, |Ed| < 1 is inelastic.

## Worked example
Price 10 → 12, quantity 100 → 80. Ed = -20% / 20% = -1.0.

## Comparison
Salt (inelastic) vs airline tickets (elastic).

## Summary
Elasticity decides whether a price rise increases revenue.
```

### Science explainer

```
# Photosynthesis
Audience: high-school biology. Goal: explain how plants store light energy.

## Core question
How does a plant turn light into chemical energy?

## Overall reaction
6CO2 + 6H2O → C6H12O6 + 6O2, driven by light in the chloroplast.

## Two stages
1. Light reactions in the thylakoid produce ATP and NADPH.
2. The Calvin cycle in the stroma fixes CO2 into sugar.

## Why it matters
Photosynthesis is the entry point of almost all energy in food chains.
```

### Workplace training

```
# Expense reimbursement
Audience: new employees. Goal: submit a claim correctly the first time.

## Step 1: Collect receipts
Within 30 days of the expense.

## Step 2: Submit in the finance system
Choose the expense type and attach receipts.

## Decision point
Claims above 5,000 need department-head approval.

## Step 3: Finance review
Payment within 5 working days.

## Common mistakes
Wrong invoice title; submitting across months.
```

### Product showcase

```
# Get started with Acme Reports in 3 minutes
Audience: operations managers. Goal: start a free trial.

## Problem
Manual reporting takes 4 hours a week.

## Key features (see uploaded screenshots)
1. One-click import  2. Automatic charts  3. Share links

## Result
Customers report saving 70% of reporting time.

## Call to action
Start a 14-day free trial.
```

Upload 2–5 real product screenshots and pass them with `files_role: "primary"`.

### Story

```
# Apollo 11
Audience: general viewers. Goal: tell the story of the first Moon landing.

## 1961
Kennedy commits to landing on the Moon within the decade.

## 16 July 1969
Launch from Kennedy Space Center.

## 20 July 1969
Armstrong steps onto the Moon: "one small step for man".

## Legacy
A turning point in the Cold War space race.
```

Real dates let the agent build a timeline.

### Document to video

```
Turn the uploaded PDF into a 5-minute explainer for product managers.
Focus on the conclusions and data in chapters 2 and 3; skip the appendix.
Keep the original formulas and figures; do not invent numbers.
```

Upload the pdf, pptx or docx (≤10 MB each, max 5) and pass `files_role: "primary"`.

## Editing a finished video

Send a concrete instruction in `send_message`: name the scene and the change ("Scene 3: replace the example with a retail case, keep it under 30 seconds"). Language, length, style, voice, avatar and brand kit are fixed at creation; changing them means a new conversation.

# Week 5: Reusable AI Skill

## Skill Summary

This project builds a reusable AI skill named `support-sla-clock`.

The skill calculates exact SLA due times for support tickets when the SLA runs only during business hours. It handles time zones, workdays, holidays, and business-hour windows, which makes the Python script load-bearing rather than decorative.

## Why I Chose This Skill

I chose this workflow because it is narrow, practical, and genuinely requires deterministic code. A language model can explain SLA logic, but it cannot reliably compute exact due timestamps across time zones, weekends, holidays, and fractional business hours without a script that performs the schedule math precisely.

## Folder Structure

```text
.
├─ .agents/
│  └─ skills/
│     └─ support-sla-clock/
│        ├─ SKILL.md
│        ├─ references/
│        │  └─ demo-prompts.md
│        └─ scripts/
│           └─ sla_due.py
└─ README.md
```

## What The Skill Does

The skill helps an agent:

- recognize when a user needs a precise SLA deadline
- collect the exact schedule inputs needed
- run a Python script to calculate the due timestamp
- report the result clearly, including assumptions and overdue status when relevant

## How To Use It

Example script invocation:

```bash
python3 .agents/skills/support-sla-clock/scripts/sla_due.py \
  --opened-at "2026-04-27T16:20:00-04:00" \
  --sla-hours 6 \
  --timezone "America/New_York" \
  --business-start "09:00" \
  --business-end "17:00" \
  --workdays "0,1,2,3,4"
```

Optional arguments:

- `--holidays "2026-07-03,2026-12-25"`
- `--now "2026-04-28T15:00:00-04:00"`

In an agent workflow, the model should invoke the skill when the user asks for an exact deadline based on business-hour SLA rules rather than a rough estimate.

## What The Script Does

`sla_due.py` performs the deterministic part of the workflow:

- parses ISO 8601 timestamps
- validates the IANA time zone
- applies local business-hour windows
- skips non-working weekdays
- skips explicit holiday dates
- accumulates fractional SLA hours across multiple days
- optionally calculates whether the ticket is overdue

## Demo Prompts

The skill was designed to handle at least these three prompt types:

1. Normal case: exact due time for a ticket opened late in the day.
2. Edge case: due time crossing a holiday and using fractional hours.
3. Cautious case: insufficient inputs, where the agent should ask for the missing schedule details instead of guessing.

Suggested prompts are stored in `.agents/skills/support-sla-clock/references/demo-prompts.md`.

## Example Outputs

### Normal Case

Input:

- Opened: `2026-04-27T16:20:00-04:00`
- SLA: `6` business hours
- Time zone: `America/New_York`
- Business hours: `09:00-17:00`
- Workdays: Monday-Friday

Expected result summary:

- 40 minutes are consumed on Monday
- 5 hours 20 minutes remain
- due time is Tuesday, `2026-04-28T14:20:00-04:00`

### Edge Case

Input:

- Opened: `2026-07-02T16:30:00-04:00`
- SLA: `4.5` business hours
- Time zone: `America/New_York`
- Business hours: `09:00-17:00`
- Holiday: `2026-07-03`

Expected result summary:

- 30 minutes are consumed on July 2
- July 3 is skipped as a holiday
- July 4 and July 5 are skipped as weekend days
- 4 hours remain on Monday
- due time is Monday, `2026-07-06T13:00:00-04:00`

### Cautious Case

The agent should not guess if the user only says "opened yesterday afternoon" or gives a vague time zone like "Brazil time." It should ask for:

- exact timestamp
- exact IANA time zone
- business-hour window
- workday policy

## What Worked Well

- The skill is narrow and reusable.
- The script is clearly central to the workflow.
- The description is specific enough for an agent to activate it appropriately.
- The command-line interface makes the deterministic step easy to test and demonstrate.

## Limitations

- The current version supports one fixed business-hours window per workday.
- Holidays must be passed as explicit dates.
- The skill computes schedule math only; it does not choose the correct legal or contractual SLA policy.

## Video Link

[Project Demo Video](https://youtu.be/3JCp9KxuCg4)

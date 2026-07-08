# Notes

## One fix, explained

**The undercharging bug.** `rental_days()` computed `(to_date - from_date).days`,
which is an *exclusive* day count. But the app bills inclusively — both the
pickup day and the return day count. So a same-day rental (from == to) came
out to 0 days, and `total = daily_rate * 0 = 0`. Every other rental was also
short by exactly one day (e.g. Jan 10–15 priced as 5 days instead of 6).
The fix just adds 1: `(to_date - from_date).days + 1`, which matches the
stated business rule and makes a same-day rental correctly bill as 1 day.

## The double-booking failure, made concrete

Existing booking: Canon DSLR Camera, **Jan 10–15**.
Original code let you also book the same camera for **Jan 5–20** (a range that
fully engulfs the existing one) — `dates_overlap` only checked if the new
booking's *start* date fell inside the existing range, and Jan 5 doesn't.
The fix (`start_a <= end_b and start_b <= end_a`) now correctly rejects this,
along with the fully-inside case (e.g. Jan 12–13) and the straddling cases,
which had the same problem.

## AI used

I used an AI assistant (Claude) to help review the code, localize each bug,
and draft the fixes and this write-up. For each fix I didn't just accept the
suggestion — I checked it against the stated business rules myself and ran
small targeted tests before treating anything as done:

- For `dates_overlap`, I hand-tested it against six cases (before, after,
  starts-before/ends-inside, fully-inside, engulfing, and touching on the
  boundary) to confirm the new formula gives the right True/False for each.
- For `rental_days`, I checked the same-day case and the original Jan 10–15
  example by hand against the daily rate to confirm the total matched what
  the business rule describes.
- For the maintenance rule, I traced through both endpoints
  (`/api/availability` and `POST /api/bookings`) to make sure the check
  was applied everywhere the rule needs to hold, not just one.
- For the frontend bug, I read through the actual event listener wiring line
  by line rather than trusting a description of the symptom, which is what
  turned up the missing listener on the "to" field.
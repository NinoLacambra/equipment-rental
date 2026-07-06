# NOTES

## One Fix Explained

**Task 2 — PHP 0.00 bug.**

`rental_days()` used the raw date difference:

```python
return (to_date - from_date).days
```

For a same-day rental this returns 0, so `total = daily_rate * 0 = 0`, and every rental was undercharged by one day. The business rules bill inclusively — both start and end day count.

Fix:

```python
return (to_date - from_date).days + 1
```

Sanity check: the seed Jan 10–15 booking has a total of PHP 9,000 = 6 × 1,500, confirming 6 (inclusive) is the intended count.

---

## Double-Booking Failure Example

Existing booking: **Canon DSLR Camera, 2026-01-10 → 2026-01-15**

The original code wrongly allowed: **2026-01-08 → 2026-01-12**

The old check `start_b <= start_a <= end_b` only tested whether the _new start date_ fell inside the existing range, missing any case where the new booking began _before_ the existing one. The corrected check `start_a <= end_b and start_b <= end_a` blocks it.

---

**How I verified:**

- Reproduced each bug in the running app before fixing it.
- Retested each fix with the same scenario.
- Tested Task 3 via `curl` directly against the API — this caught a real bug in my first attempt where I'd written `!=` instead of `==` in the maintenance checks, which had inverted the logic even though the frontend appeared to work.
- Cross-checked pricing math against the seed booking (Jan 10–15 = PHP 9,000 = 6 × 1,500).

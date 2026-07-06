# Equipment Rental

A small Flask app for renting out equipment (cameras, drills, projectors, etc.). It exposes a JSON API and a single-page frontend for creating bookings.

Originally provided as a take-home exercise; this repository contains the fixes and the small feature described in ./NOTES.md.

---

## Stack

- **Backend:** Python 3.10+ / Flask
- **Frontend:** Plain HTML + vanilla JavaScript (`index.html`)
- **Storage:** Local JSON file (`bookings.json`)

No database or external services required.

---

## Project layout

```
├── app.py              # Flask backend — booking logic and JSON API
├── index.html          # Single-page frontend
├── bookings.json       # Persisted bookings
├── requirements.txt    # Python dependencies
├── NOTES.md            # Fix explanations and notes
└── README.md
```

---

## Business rules

- A booking reserves one piece of equipment for a date range.
- Rentals are billed **inclusively** — both the start day and the end day count. A same-day rental is 1 day.
- A booking cannot overlap an existing booking for the same equipment.
- Equipment with `status: "maintenance"` cannot be booked and is not shown as available.

---

## Getting started

### Prerequisites

- Python **3.10 or newer** (`python3 --version`)
- A modern web browser

### Setup (macOS / Linux / WSL)

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

### Setup (Windows PowerShell)

```powershell
python -m venv venv
venv\Scripts\Activate.ps1
pip install -r requirements.txt
python app.py
```

> If PowerShell blocks activation with an execution-policy error, run once in the same window:
> `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`

Then open **http://localhost:5000** in your browser.

To leave the virtual environment when you're done:

```bash
deactivate
```

---

## API reference

Base URL: `http://localhost:5000`

### `GET /api/equipment`

Returns the full equipment catalog, including items under maintenance (the frontend shows those as disabled).

```json
[
  {
    "id": 1,
    "name": "Canon DSLR Camera",
    "daily_rate": 1500.0,
    "status": "available"
  },
  {
    "id": 3,
    "name": "HD Projector",
    "daily_rate": 900.0,
    "status": "maintenance"
  }
]
```

### `GET /api/availability?from=YYYY-MM-DD&to=YYYY-MM-DD`

Returns equipment that is **bookable** for the given inclusive date range. Excludes:

- items whose status is `maintenance`
- items already booked for any overlapping day

### `GET /api/bookings`

Returns all persisted bookings.

### `POST /api/bookings`

Creates a booking.

**Request body:**

```json
{
  "equipment_id": 1,
  "customer": "Jane Doe",
  "from_date": "2026-02-10",
  "to_date": "2026-02-12"
}
```

**Responses:**

| Status | Meaning                                                                             |
| -----: | ----------------------------------------------------------------------------------- |
|    201 | Booking created. Response body includes the saved booking (with `id` and `total`).  |
|    400 | Unknown equipment, invalid date range, or equipment under maintenance.              |
|    409 | Requested dates overlap an existing booking. Response includes the conflicting one. |

**Example — same-day rental:**

```bash
curl -X POST http://localhost:5000/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"equipment_id": 2, "customer": "Test", "from_date": "2026-02-05", "to_date": "2026-02-05"}'
```

```json
{
  "id": 2,
  "equipment_id": 2,
  "equipment_name": "Cordless Drill",
  "customer": "Test",
  "from_date": "2026-02-05",
  "to_date": "2026-02-05",
  "total": 480.0,
  "status": "confirmed"
}
```

---

## Testing the fixes manually

The four issues addressed in this repo can be reproduced from a clean `bookings.json` (with only the seed Canon DSLR booking, Jan 10–15).

**1. Double-booking is now blocked**

```bash
curl -X POST http://localhost:5000/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"equipment_id": 1, "customer": "X", "from_date": "2026-01-08", "to_date": "2026-01-12"}'
# → 409 Conflict
```

**2. Same-day rental is billed correctly**

```bash
curl -X POST http://localhost:5000/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"equipment_id": 2, "customer": "X", "from_date": "2026-02-05", "to_date": "2026-02-05"}'
# → total: 480.0  (not 0.0)
```

**3. Maintenance equipment is rejected**

```bash
curl -X POST http://localhost:5000/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"equipment_id": 3, "customer": "X", "from_date": "2026-03-01", "to_date": "2026-03-05"}'
# → 400 "HD Projector is under maintenance and cannot be booked"
```

**4. Frontend price updates on either date**

Open http://localhost:5000, pick an equipment and a start date — then change the end date a few times. The total should recalculate on every change.

---

# Ride_Safe

Ride_Safe is a Flask-based, privacy-first ridesharing and carpool coordination service. The project emphasizes confidentiality and integrity for sensitive ride and profile data by combining ECC and RSA encryption with HMAC (server-side) integrity checks.


---

## Table of Contents

- [About](#about)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the App](#running-the-app)
- [Database / Tables](#database--tables)
- [Testing](#testing)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

---

## About

Ride_Safe is a server-side web application implemented in Flask. It provides ride request/accept flows, rider and driver dashboards, encrypted storage for trip data, and emergency (SOS) reporting. The codebase is organized with Flask blueprints under `routes/` so teammates can work on separate modules (auth, keys, profile, trips, chat, sessions, admin).

## Key Features

- Encrypted trip payloads: pickup, dropoff, and timing are encrypted using ECC and stored as JSON in the database.
- Integrity protection: every encrypted payload is MACed with an HMAC-SHA256 tag (server-side key) and verified before decryption to detect tampering.
- Role-aware dashboards: separate driver and rider dashboards that only reveal decrypted trip details when appropriate and authorized.
- Trip lifecycle management: request -> accept -> arrive -> in_progress -> completed, with status transitions enforced server-side.
- SOS / emergency reporting: `trigger_sos` creates an `emergencies` record and preserves driver/rider context for follow-up.
- Profile encryption: sensitive profile fields are stored encrypted (RSA helpers present) and decrypted only when needed.
- Supabase/Postgres integration: psycopg2 helpers (db.py) and support for Supabase connection strings.
- Template-driven UI: Jinja2 templates under `templates/` with static assets in `static/` for quick UX iteration.

(See `routes/trips.py`, `db.py`, `app.py`, and `crypto/` for implementation details.)

## Tech Stack

- Python 3.8+
- Flask (blueprint-based)
- PostgreSQL / Supabase (psycopg2)
- Cryptography: ECC and RSA helpers in `crypto/` and HMAC utilities
- Templating: Jinja2 (`templates/`)

## Installation

Prerequisites

- Python 3.8+ and pip
- PostgreSQL (or Supabase)

Clone and install

```bash
git clone https://github.com/maisha0055/Ride_Safe.git
cd Ride_Safe
python -m venv .venv
source .venv/bin/activate   # macOS / Linux
.venv\Scripts\activate    # Windows
pip install -r requirements.txt
```

## Configuration

Create a `.env` file at the project root (dotenv is used). Important variables referenced in `config.py`:

```env
FLASK_SECRET_KEY=replace-with-secure-random
DATABASE_URL=postgres://user:pass@host:port/dbname
SUPABASE_URL=your-supabase-url
SUPABASE_KEY=your-supabase-key
RIDESAFE_MAC_KEY=replace-with-secure-mac-key
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=you@example.com
MAIL_PASSWORD=supersecret
MAIL_FROM=you@example.com
MAIL_USE_TLS=true
```

Security notes

- Keep `RIDESAFE_MAC_KEY` secret: it is used to compute/verify HMACs for encrypted trip payloads.
- Never commit `.env` or secret values to the repository.

## Running the App

Run directly for development (app.py includes a main guard):

```bash
python app.py
```

By default the server listens on port `5001` (or the value of the `PORT` environment variable). In development debug mode is enabled.

Access the app at: http://localhost:5001/

## Database / Tables

The code expects the following tables (names & columns used by the code):

- `users` — user identities, `username`, `contact_encrypted`, etc.
- `user_keys` — `user_id`, `ecc_public_key`, `ecc_private_key_encrypted`, `rsa_private_key_encrypted`
- `profiles` — `user_id`, `name_encrypted`, `phone_encrypted`, `vehicle_info_encrypted`
- `trips` — `id`(uuid), `rider_id`, `driver_id`, `pickup_encrypted`(json), `dropoff_encrypted`(json), `timing_encrypted`(json), `status`, `created_at`
- `emergencies` — created by `db.ensure_emergencies_table()`; stores `trip_id`, `rider_id`, `driver_id`, `driver_name`, `driver_phone`, `driver_vehicle`, `status`, timestamps

If you use Supabase, create the tables in your project or provide SQL migrations that match these expectations.

## Testing

Run tests with pytest (there is a `tests/` directory and `test_db.py`):

```bash
pytest
# or
python -m pytest -q
```

## Contributing

- Fork the repo and open PRs from feature branches: `git checkout -b feat/your-feature`
- Add tests for significant logic changes
- Keep secrets out of commits and document any required database migrations

## License

This repository contains an MIT License file. See `LICENSE` for details.

## Contact

maisha0055


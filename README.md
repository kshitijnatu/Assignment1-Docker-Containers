# Assignment1 – Docker Compose (Postgres + Python)

## What this stack does
This project runs a two-container stack: a PostgreSQL 16 database initialized with sample `trips` data, and a Python 3.11 app that queries the DB and produces a small JSON summary. The app writes the summary to a mounted output folder and also prints the same summary to stdout.

## Prerequisites
- Docker + Docker Compose (`docker compose`)
- (Optional) `make`

## How to build/run/stop

### Using `make` (recommended)
```sh
make all
```

Stop and remove containers + named volumes:
```sh
make down
```

Recreate a clean `out/` folder (also stops the stack):
```sh
make clean
```

### Using Docker Compose directly
Build and start:
```sh
docker compose up --build
```

Stop and remove containers + volumes:
```sh
docker compose down -v
```

## Example output (Pasted summary of the JSON output by the app)
When the stack runs, the `app` container prints something like:

```text
=== Summary ===
{
  "total_trips": 6,
  "avg_fare_by_city": [
    {
      "city": "New York",
      "avg_fare": 19.0
    },
    {
      "city": "San Francisco",
      "avg_fare": 20.25
    },
    {
      "city": "Charlotte",
      "avg_fare": 16.25
    }
  ],
  "top_by_minutes": [
    {
      "city": "San Francisco",
      "minutes": 28,
      "fare": 29.3
    },
    {
      "city": "New York",
      "minutes": 26,
      "fare": 27.1
    },
    {
      "city": "Charlotte",
      "minutes": 21,
      "fare": 20.0
    },
    {
      "city": "Charlotte",
      "minutes": 12,
      "fare": 12.5
    },
    {
      "city": "San Francisco",
      "minutes": 11,
      "fare": 11.2
    },
    {
      "city": "New York",
      "minutes": 9,
      "fare": 10.9
    }
  ]
}
```

Notes:
- The DB is seeded from [`db/init.sql`](db/init.sql).
- `APP_TOP_N` is set to `10` in [`compose.yml`](compose.yml), so you’ll see up to 10 rows in `top_by_minutes` (with the current seed data, that’s all 6 rows).

## Where outputs are written
- The app writes JSON to `/out/summary.json` **inside** the container.
- That path is bind-mounted to `./out/summary.json` on your machine via [`compose.yml`](compose.yml).

So after running, you should have:
- `out/summary.json` (relative to the repo root)

## Configuration
The app reads these environment variables (set in [`compose.yml`](compose.yml)):
- `DB_HOST` (default `db`)
- `DB_PORT` (default `5432`)
- `DB_USER` / `DB_PASS` / `DB_NAME`
- `APP_TOP_N` (default in code is `5`, compose sets `10`)

Implementation is in [`app/main.py`](app/main.py).


## Project layout
- [`compose.yml`](compose.yml): defines `db` + `app`, healthcheck, env vars, output mount
- [`db/Dockerfile`](db/Dockerfile) + [`db/init.sql`](db/init.sql): Postgres image + seed data
- [`app/Dockerfile`](app/Dockerfile) + [`app/main.py`](app/main.py): Python app that queries DB and writes `out/summary.json`
- [`Makefile`](Makefile): convenience commands

## Contents of out/summary.json
```text
{
  "total_trips": 6,
  "avg_fare_by_city": [
    {
      "city": "New York",
      "avg_fare": 19.0
    },
    {
      "city": "San Francisco",
      "avg_fare": 20.25
    },
    {
      "city": "Charlotte",
      "avg_fare": 16.25
    }
  ],
  "top_by_minutes": [
    {
      "city": "San Francisco",
      "minutes": 28,
      "fare": 29.3
    },
    {
      "city": "New York",
      "minutes": 26,
      "fare": 27.1
    },
    {
      "city": "Charlotte",
      "minutes": 21,
      "fare": 20.0
    },
    {
      "city": "Charlotte",
      "minutes": 12,
      "fare": 12.5
    },
    {
      "city": "San Francisco",
      "minutes": 11,
      "fare": 11.2
    },
    {
      "city": "New York",
      "minutes": 9,
      "fare": 10.9
    }
  ]
}
```

## Reflection Of Assignment

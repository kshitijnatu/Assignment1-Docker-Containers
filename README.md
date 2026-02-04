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

## Troubleshooting

### DB not ready / app fails to connect
Even with `depends_on`, the DB can take a few seconds to accept connections.

Things to try:
- Check container status: `docker compose ps`
- Inspect logs:
  - DB: `docker compose logs -f db`
  - App: `docker compose logs -f app`
- Verify Postgres readiness from inside the DB container:
  - `docker compose exec db pg_isready -U $DB_USER -d $DB_NAME`
- If you see repeated connection retries, bring the stack down and restart:
  - `docker compose down`
  - `docker compose up --build`

If you changed `db/init.sql` and don’t see new data, remember that Postgres only runs init scripts on a **fresh** data volume. Reset volumes and retry:
- `docker compose down -v`
- `docker compose up --build`

### Permission errors writing to `out/`
The app writes to `/out/summary.json` inside the container, bind-mounted to `./out/summary.json` on your machine. If the file/folder is not writable you may see errors like “Permission denied”.

Fixes:
- Recreate the folder:
  - `rm -rf out && mkdir -p out`
- Ensure it’s writable:
  - `chmod -R u+rwX out`


### Port conflict on 5432
If you already have Postgres running locally, Compose may fail to bind the port.

Fixes:
- Stop the local service using 5432, or
- Change the host port mapping in `compose.yml` (e.g., map to `5433:5432`).

### Output file not created
If `out/summary.json` doesn’t appear:
- Confirm the app container actually ran and exited: `docker compose ps`
- Check app logs: `docker compose logs app`
- Ensure the bind mount path in `compose.yml` points to `./out/summary.json` and that the `out/` directory exists.

## Reflection Of Assignment
### What I learned
I learned how to create a Dockerfile and add all of the commands to create multiple containers for a simple python application. I also learned how to coordinate multiple containers with Docker Compose to ensure that the application runs smoothly and as intended. Another thing I learned is how to create a README.md file so that I can document how the application works and what commands are needed to successfully run the application.

### What I would improve
I would create a .env file so that I do not hardcode any credentials in my [`compose.yml`](compose.yml), which would make it easier to change configurations without editing the Compose file, and I would also add more structured logging (with clear connection attempts and output file paths) to improve observability and debugging.
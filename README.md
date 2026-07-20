# Axpo development challenge

A small weather-observations stack over AEMET OpenData (Antarctic stations), plus a UI to query it. The brief is in [`assignment.md`](assignment.md).

## Layout

| Path | Role |
|------|------|
| [`backend/`](backend/) | FastAPI service (`meteo_service`) — git submodule |
| [`frontend/`](frontend/) | React SPA (Bun) — git submodule |
| [`assignment.md`](assignment.md) | Original challenge text |

## Stack

**Backend** — FastAPI with a **hexagonal (ports & adapters)** layout around an `observations` bounded context. Domain and use cases stay free of FastAPI / httpx / SQLAlchemy; adapters cover HTTP, AEMET, and SQLite. Details: [`backend/README.md`](backend/README.md).

**Frontend** — **React 19** + TypeScript, run and bundled with **Bun** (no Vite). TanStack Query for fetches, TanStack Table for the grid, Recharts for per-metric charts. Details: [`frontend/README.md`](frontend/README.md).

## Assignment vs delivered

### Part 1 — API

| Asked | Delivered |
|-------|-----------|
| FastAPI or Flask over AEMET Antartida | FastAPI `GET /api/v1/observations` |
| API key from env | `AEMET_API_KEY` via pydantic-settings |
| `start` / `end` (`AAAA-MM-DDTHH:MM:SS`), optional location, two Antarctic stations, aggregation (`None` / hourly / daily / monthly), data fields (temp / pressure / speed; empty = all) | Same surface; location is IANA only (offsets like `+02:00` rejected); default `Europe/Madrid` |
| Aggregate ~10‑min samples when requested; daily/monthly respect station TZ | Yes (station TZ for buckets; see backend README) |
| Map `nombre` / `fhora` / `temp` / `pres` / `vel` | Response: `station` `{id,name}`, `datetime`, `temperature`, `pressure`, `speed` |
| Output datetimes in `Europe/Madrid` with offset; DST behaviour | Always Madrid + offset; DST covered in tests / docs |
| Tests + git repo | pytest (+ quality gate); this repository |

### Part 2 — Scale

| Asked | Delivered |
|-------|-----------|
| Drop the two-station restriction | **Not on `main`**: still allowlisted to `89064` / `89070`. AEMET has no single product that matches Part 1’s shape for all Spanish stations; rationale + dual-product experiment branch in [`backend/README.md`](backend/README.md) |
| SQLite cache of source data | Yes — raw samples, empty-miss / any-rows-hit |
| Proper logging | Yes (cache hit/miss, fetch/upsert counts) |

### Part 3 — Frontend

| Asked | Delivered |
|-------|-----------|
| UI to query data in a table, maybe a chart | Filters form, table, and one chart per metric (temp / pressure / speed) |
| Supporting endpoints if needed | Uses existing observations API (browser → FastAPI via CORS) |

![Filters and temperature chart](docs/images/dashboard_1.jpg)

![Observations table](docs/images/dashboard_2.jpg)

## Clone

```bash
git clone --recurse-submodules https://github.com/illescasDaniel/axpo_aemet_assignment.git
```

## How to run

1. Backend — from [`backend/`](backend/): see [`backend/README.md`](backend/README.md) (`uv sync`, `.env`, `uv run task dev`).
2. Frontend — from [`frontend/`](frontend/): see [`frontend/README.md`](frontend/README.md) (`bun install`, `.env`, `bun dev`).

## Known gaps (documented)

- Open station catalogue / multi-product AEMET (Part 2) — see backend README + branch `archive/aemet-dual-product`
- Cache gap-fill for partial windows; no TTL / negative cache
- Auth, rate limits, multi-process cache coalescing — out of take-home scope

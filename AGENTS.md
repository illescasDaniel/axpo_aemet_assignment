# Agent guidance (axpo-assignment)

Preferences for humans and coding agents working in this repo.

## Code style: simple but powerful

- Prefer **short, readable** solutions over clever or layered abstractions.
- Use libraries for what they’re good at (e.g. **pydantic-settings** for env validation) instead of hand-rolled equivalents.
- **No dead weight:** if it doesn’t help runtime, tests, DX, or clarity, remove it (`[project.scripts]` you don’t use, placeholder `main()`, etc.).
- **Empty `__init__.py`:** usually not needed (namespace packages). Don’t add them by default. **Keep** `backend/src/meteo_service/__init__.py` — `uv_build` requires that top-level package marker. Add or keep other `__init__.py` only when there is a concrete reason (re-exports, tooling that demands a regular package, etc.), not “for completeness.”
- **Easy to inject:** build small objects (`Settings`, clients, repositories) and pass them where needed (FastAPI `Depends`, constructors) rather than scattering `os.environ` reads.
- Indent with **spaces** (Ruff format); line length 120.
- **`-> None` sparingly.** Omit it on most functions that return nothing (handlers, tests, scripts, etc.) — it adds noise without much benefit. **Do** annotate `-> None` on **Protocol** methods and their adapter/fake implementations when the method is a deliberate side-effect API (e.g. `upsert_many`, `commit`). Skip `__init__` and methods whose signature already makes a void return obvious (e.g. `__aexit__`, or a parameterless `rollback()`). Keep other return annotations.

### Settings (pydantic-settings)

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")
    aemet_api_key: str = Field(min_length=1)

@lru_cache
def get_settings() -> Settings:
    # basedpyright does not model env-based BaseSettings init — ignore is intentional.
    return Settings()  # pyright: ignore[reportCallIssue]
```

Do **not** rely on `Field(...)` alone to silence pyright; use the ignore on the `Settings()` call site.

Load via `get_settings()` and inject — don’t re-parse env in every module.

## Architecture

- Backend: **hexagonal** (ports & adapters) around an `observations` bounded context.
- Domain/application stay free of FastAPI/httpx/SQLAlchemy details.
- Adapters: `api`, `aemet`, `database`. Fakes for ports live under `tests/`, not production adapters.
- Unit of Work owns DB commit/rollback; repositories do not commit.

## Backend stack (intentional)

- FastAPI (`fastapi[standard]`), httpx2, SQLAlchemy async + aiosqlite, pydantic-settings
- ruff, basedpyright, pytest, pytest-asyncio, taskipy, pip-audit
- API tests: **async** `httpx2.AsyncClient` + `ASGITransport` (not sync `TestClient`)

## Quality gate

From `backend/`:

```bash
./scripts/quality/checks.sh --fix   # autofix (ruff/shfmt) + full gate
./scripts/quality/checks.sh         # verify clean (no autofix)
# or:
uv run task checks-fix
uv run task checks
```

### After code changes (agents)

1. Run `./scripts/quality/checks.sh --fix` from `backend/`.
2. If the gate still fails with issues that cannot be autofixed (basedpyright, pytest, pip-audit, logic errors, etc.), **fix them manually**.
3. Re-run `./scripts/quality/checks.sh --fix` until the gate passes before considering the work done.

CI for the backend lives under `backend/.github/` (uv quality gate). Frontend CI lives under `frontend/.github/` (`bun run checks`).

Don’t recreate Antarctica-specific naming at the package level (`meteo_service` / `observations` stay place-agnostic).

## Git / scope

- Only commit when asked.
- Don’t expand scope beyond the task; match existing patterns in the repo.

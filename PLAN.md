## Solution Plan

**Issue:** [#155 — Health check references settings.redis_host, which does not exist on Settings](https://github.com/ascherj/pathreview/issues/155)

### Understand

**Root cause:** When the health check endpoint tries to instantiate a Redis connection in `api/routes/health.py` (lines 44–49), it references `settings.redis_host` and `settings.redis_port`. These fields do not exist in the Settings model in `core/config.py`, causing an `AttributeError` before the Redis health check can run.

**Expected behavior:** The health endpoint should successfully connect to Redis (if running) and report its status as healthy or unhealthy based on actual connectivity.

**Current behavior:** The endpoint crashes with `AttributeError` before it can probe Redis.

### Map

**Files to modify:**
- `core/config.py` — Add `redis_host` and `redis_port` fields to the Settings model
- `api/routes/health.py` — Already references these fields (lines 44–49); will work once Settings is updated

### Plan

1. Add `redis_host` and `redis_port` fields to the Settings model in `core/config.py` (with defaults matching `redis_url`)
2. Update `.env.example` to include `REDIS_HOST` and `REDIS_PORT` entries so developers know these fields exist
3. Verify the health check endpoint can now instantiate a Redis connection without `AttributeError`
4. Test GET `/health` with Redis running and confirm it reports `"redis": "healthy"`
5. Write unit tests for the `/health` endpoint covering success and failure cases
6. Verify no other code references these fields (grep confirms they only appear in health.py)

### Inputs & Outputs

**Input:** GET request to `/health` endpoint

**Output:** HTTP 200 with health status JSON showing redis status, or HTTP 503 if any dependency is unhealthy

### Risks & Unknowns

**Risk:** Adding new configuration fields could cause issues if the defaults don't match the environment. Mitigation: Use the same defaults as `redis_url` parsing.

**Known:** The project already uses `.env` files and environment variables via pydantic-settings, so the new fields will be automatically populated from `REDIS_HOST` and `REDIS_PORT` env vars.

### Edge Cases

The fix should handle:
- Redis server is down → health check should report unhealthy (graceful error handling already in place)
- Redis connection timeout → error caught and status set to unhealthy
- Missing env vars → defaults from `redis_url` apply 
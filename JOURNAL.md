## Week 7 — Issue selection

**Issue link:** [Issue #155](https://github.com/ascherj/pathreview/issues/155)

**Issue title:** Health check references settings.redis_host, which does not exist on Settings


**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The **GET /health** endpoint in `api/routes/health.py` attempts to create a Redis connection using `settings.redis_host` and `settings.redis_port` but the Settings model in `core/config.py` does not defines these two fields which leads to an `AttributeError` in the `/health` route. The fix requires to either parse these fields from the existing `redis_url` field in settings or add them explicitly into the settings.  I personally plan to add these to the settings field so that the configuration is explicit and is managable through specific injected fields. 


**Branch name:** fix/155-redis-host-reference-health-check

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger


**Checklist reasoning:**

[X] **Understanding:** Can explain problem (endpoint uses non-existent fields), found files (health.py, config.py), understand done-state (GET /health works, reports Redis status)

[X] **Tier Fit:** First contribution, Tier 1 appropriate—self-contained 2-file fix

[X] **Codebase Ready:** Read health.py lines 44–49, config.py Settings model, no existing /health tests (will write first)

[X] **Scope Realistic:** 2–3 hours estimated, no blockers, redis_host only referenced in health.py

### Bug Reproduction

1. Start the Docker containers, server, and frontend with `make setup` and `make run`
2. Open your browser and navigate to `http://localhost:8000/health`
3. Observe the error: 
`json
{"detail":{"status":"unhealthy","dependencies":{"postgres":"unhealthy","redis":"unhealthy","vector_db":"healthy"},"safety_events_last_hour":0,"timestamp":"2026-07-29T01:38:52.933153"}}
`

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/skytechnicalsrelations/pathreview/commit/9ce935ea594432e53d76498b0e5753fe1b28635e

**Reproduction summary:**
Since the bug is trigerred when accessing the `/health` path, it can be easily reproduced by sending a GET request to that path. To do so, the simplest step is to goto a browser and type in http://localhost:8000/health  or `curl http://localhost:8000/health`.

Upon sending that GET request to the `/health` endpoint, you will see an error json message from the server saying "redis":"unhealthy". Also, you can see server logs that correspond to the error message associated with GET requests being sent to the server (e.g. see below)

2026-07-28 21:57:17 [info     ] application_startup_completed 
INFO:     Application startup complete.
2026-07-28 21:57:26 [error    ] postgres_health_check_failed   error="Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')" request_id=83a4cd22-8b52-494d-ab09-8a6c47daaba8
2026-07-28 21:57:26 [error    ] redis_health_check_failed      error="'Settings' object has no attribute 'redis_host'" request_id=83a4cd22-8b52-494d-ab09-8a6c47daaba8
2026-07-28 21:57:26 [debug    ] vector_db_health_check_passed  request_id=83a4cd22-8b52-494d-ab09-8a6c47daaba8
INFO:     127.0.0.1:54932 - "GET /health HTTP/1.1" 503 Service Unavailable

**PLAN.md link:** https://github.com/skytechnicalsrelations/pathreview/blob/fix/155-redis-host-reference-health-check/PLAN.md

**Walkthrough video (recommended):** 

**Blockers or open questions:**

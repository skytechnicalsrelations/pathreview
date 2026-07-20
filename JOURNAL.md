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
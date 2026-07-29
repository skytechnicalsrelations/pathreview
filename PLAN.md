## Solution plan

**Issue:** [Health check references settings.redis_host, which does not exist on Settings
](https://github.com/ascherj/pathreview/issues/155)

### Understand
What is the root cause of this issue? What behavior is expected vs. actual?
The root cause of this issue is that when instantiating a Redis connection inside `api/routes/health.py`, the Settings config object is missing the required `redis_host` and `redis_port` fields causing an Exception to be raised and making the redis dependency appear as failing. 

The expected behavior is that if the Redis instance is live, this should return a healthy result but since the settings object is missing the two redis fields, it is incorrectly saying that the redis instance is unhealthy. 

### Map
Which files, functions, or modules are involved?
List the specific files you expect to touch.

In this issue, only the `core/config.py` module is involved as the Settings object needs to have those two `redis_port` and `redis_host` information added. 


### Plan
What are the steps to fix this issue?
Break it into 3–5 concrete sub-tasks.
1. Add the redis_port and redis_host fields to the settings object in `config.py`
2. reload the server and send a GET request to /health endpoint and verify that with the correct host and port info, the health check endpoint shows that the redis health is healthy instead of incorrect current unhealthy status.

### Inputs & outputs
What does your fix take as input? What should it produce or change?
The input is a GET request to the /health endpoint. After the fix is applied, it should show the redis service as healthy. 

### Risks & unknowns
What could go wrong? What are you still unsure about?
Since the change is involved with adding new fields to the settings object, this should not lead to any breaking changes. The risk is that if we are not using a .env file and are hardcoding the host and port info directly in the module, it might accidentally be misconfigured but that is beyond the scope of this bug and is more of a best practices discussion when it comes to hardcoding the credentials directly in the config file. Maybe we can use dotenv to load these config info from an env file or some secrets manager instead of hardcoding it. 

### Edge cases
What inputs or states should your fix handle gracefully?
None as there is only one possible input ie. GET request to the /health endpoint. 
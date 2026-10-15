# Overview

There were a few changes that I needed to make as part of this assignment. I've listed the main ones and the rationale for them below. While the application was missing significant functionality and would require code changes to be production ready I've kept my changes limited to those required to complete the requested tasks.

sqlite3 setup: 
- I setup the image to mount a /data directory to save the sqlite db to a persistent file. This allows the service to restart though the service should be refactored to work with a standalone DB like Postgres before being deployed in production.
- I set the app to create the schema in sqlite on launch, to ensure this could run on restarts I modified the create table command to be idempotent. There's other ways this may be done but this was the quickest.
address binding:
- I forced the server to launch on 0.0.0.0 as I wasn't getting a connection outside the container originally, it was binding to the loopback address as the default.

# 1. Dockerisation
I created a multi-stage build to reduce the complexity and attack surface of the production image. This image also imports the sqlite library which should be removed before a production release as part of refactoring for a different database. Including it in the build prevents the library from causing potential issues at runtime.

# 2. Frontend
Given the frontend serves static files there's minimal work required. It would require the creation and configuration of the following:
* Private S3 Bucket - File store backing Cloudfront with the latest files.
* Cloudfront - Managing domain name, TLS, security headers, caching, API request routing
* IAM - appropriate roles/policies required for Cloudfront to access the secured S3 bucket.

# 3. CI Pipeline
The CI pipeline perfoms basic responsibilities for a CI pipeline, ensuring the quality of the code and it is free from high/critical security issues. This pipeline could be significantly improved in it's speed, security, and integration with external systems.

Ways the pipeline could be improved:
- Concurrency to the actions taken in the test job
- Adding further security checks e.g. SAST/DAST
- Add scheduled scans of the images
- Implementing caching for build dependencies and Docker layers
- Add safety measures to pipeline e.g. job timeouts
- Tool choices e.g. OCI build could fail if Docker daemon not present in jobs runners (Kaniko or AWS CodeBuild)

# 4. Observability
Currently the only visibility into the app is the health endpoint and some unstructured console logs, which is insufficent to understand the state of the app.

What's needed:
- Metrics - request rate, errors and duration per route, plus process metrics like memory.
- Logging - structured JSON logs shipped to a central store, with one access log per request including a request ID, status and duration.
- Tracing - OpenTelemetry, Deno has a built in implementation.
- Frontend error tracking - errors on a users browswer should have visibility.
- Backup monitoring - alerting if the DB backups fails.
- Frontend to API correlation - pass trace context from the SPA to the API.
- Centralised log/trace/metric solution with dashboarding and alerts.

Changes to the implementation:
- Split the health check into liveness and readiness endpoints.
- Add middleware to log and record metrics for each request.
- Replace the console logging with a structured logger. 
- Log the startup message once the server is actually listening and include the bound address.
- Return a request ID on API responses and include it in frontend error reports.

# 5. Scaling to 20 Engineers
Scaling from 4 to 20 engineers will mandate trading some flexibility and speed for ridgity (in the right areas) and repeatability. I would look to implement controls around configuration, infra, codebase and deployment states as well as implementing standards libraries.

Things SRE would need to implement:
- Golden paths - for Dockerfile, CI, Deno common libraries will need to be implemented to allow for consistency and maintainability of our surrounding infrasturcture.
- Infrastructure as Code - Terraform/CDK/OpenTofu versioned modules created and centrally maintained state files to ensure infra is deployed as expected.
- Branch protection and stratergies - Selecting a branching stratergy and then enforcing branch/merge rules is critical to avoid constant conflicts. The branching stratergy should also align with testing processes.
- Automated Dependency Management - Renovate or other to maintain the services dependencies.
- Alerts/Notifications - Centeralising the reporting of deployments to aide in production support or alerting on newly discovered security issues.
- Documentation - providing information to assist new developers and giving context to those troubleshooting the app.
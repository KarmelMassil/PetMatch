# PetMatch
This repository contains a GitHub Actions CI/CD pipeline for the Pet Store application, developed for Assignment 4 (2025-26) of Cloud Computing and SE. It runs 2 instances of the `pet-store` service and 1 instance of the `pet-order` service, each backed by its own MongoDB instance, and validates the whole system through a 3-job build → test → query pipeline.
## Features
* **Build Job:** Builds the `pet-store` and `pet-order` Docker images and uploads them as an artifact for the later jobs.
* **Test Job:** Starts the full application with Docker Compose and runs the `assn4_tests.py` pytest suite against it (`pytest -v`).
* **Query Job:** Re-runs the application, replays the queries and purchases in `query.txt` against the live services, and records each result in `response.txt`.
* **Structured Logging:** Every run produces a `log.txt` with the start time, submitter name(s), image build status, container status, and pytest result.
* **Multi-Instance Architecture:** Two independent `pet-store` instances share one MongoDB instance, while `pet-order` uses a separate MongoDB instance, matching the assignment's required topology.
* **Dockerized Environment:** `docker-compose.yml` wires all 4 containers (2x pet-store, pet-order, and 2x MongoDB) together on a shared network.
## Files Included
* `.github/workflows/assignment4.yml` — the 3-job CI/CD workflow
* `docker-compose.yml`
* `pet-store/` — `Dockerfile`, `app.py`, `requirements.txt`
* `pet-order/` — `Dockerfile`, `app.py`, `requirements.txt`
* `tests/assn4_tests.py` — pytest suite used by the test job
* `query/run_query.py` — populates data and executes `query.txt` for the query job
* `query.txt` — sample queries/purchases used to exercise the query job
## Prerequisites
* Python 3.11+ (for local execution)
* Docker and Docker Compose (for containerized execution)
* A GitHub repository with Actions enabled (for the CI/CD pipeline itself)
## Local Installation & Setup
To run a service directly with Python (useful for debugging outside of Docker):
1. Create and activate a virtual environment, then install its dependencies:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r pet-store/requirements.txt   # or pet-order/requirements.txt
   ```
2. Start a local MongoDB instance and set the required environment variables (`MONGO_URL`, and for pet-store also `NINJA_API_KEY` and `STORE_COLLECTION`; for pet-order also `PETSTORE1_URL`/`PETSTORE2_URL`).
3. Run the service:
   ```bash
   python app.py
   ```
## Docker Deployment
To build and run the full system (2x pet-store, pet-order, 2x MongoDB) as it's intended to run:
1. Build and start all containers:
   ```bash
   docker compose build
   docker compose up -d
   ```
2. Confirm the services are reachable:
   ```bash
   curl http://localhost:5001/pet-types    # pet-store #1
   curl http://localhost:5002/pet-types    # pet-store #2
   curl http://localhost:5003/transactions # pet-order
   ```
## Testing
With the containers running, execute the pytest suite:
```bash
pip install pytest requests
pytest -v tests/assn4_tests.py
```
## CI/CD Pipeline
On every push, the `assignment4` workflow (`.github/workflows/assignment4.yml`) runs 3 jobs in sequence:
1. **build** — builds both Docker images and uploads them as the `docker-images` artifact.
2. **test** — loads those images, starts the app with Docker Compose, and runs the pytest suite; uploads `assn4_test_results.txt` (as `test-results`) and the completed `log.txt` (as `log`).
3. **query** — re-populates the app, executes `query.txt`, and uploads the resulting `response.txt` (as `query-responses`).

All artifacts are available from the workflow run's summary page on GitHub once it finishes.

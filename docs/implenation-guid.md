# Logistics Optimizer — Implementation Guide

This document describes how to implement the Logistics Optimizer project from an empty repository to a production-ready application.

The implementation should be completed incrementally.

At the end of every step:

1. Run the relevant tests.
2. Verify the application works.
3. Commit the changes.
4. Only then move to the next step.

---

# 1. Project Implementation Roadmap

The project will be implemented in the following stages:

```text
Step 01 — Initialize repository
        ↓
Step 02 — Python environment and dependencies
        ↓
Step 03 — Project structure
        ↓
Step 04 — Configuration management
        ↓
Step 05 — Docker development environment
        ↓
Step 06 — FastAPI application
        ↓
Step 07 — MySQL + SQLAlchemy
        ↓
Step 08 — Alembic migrations
        ↓
Step 09 — Authentication and JWT
        ↓
Step 10 — Database models
        ↓
Step 11 — Pydantic schemas
        ↓
Step 12 — Repositories
        ↓
Step 13 — Services
        ↓
Step 14 — CRUD APIs
        ↓
Step 15 — Optimization domain
        ↓
Step 16 — Objective functions
        ↓
Step 17 — Optimization algorithms
        ↓
Step 18 — Optimization engine
        ↓
Step 19 — Optimization API
        ↓
Step 20 — Background workers
        ↓
Step 21 — Testing
        ↓
Step 22 — Logging and error handling
        ↓
Step 23 — Nginx + production Docker
        ↓
Step 24 — CI
        ↓
Step 25 — AWS deployment
        ↓
Step 26 — CD
        ↓
Step 27 — Monitoring and backups
```

---

# 2. Step 01 — Initialize the Repository

## Goal

Create the Git repository and basic project files.

## Create

```text
logistics-optimizer/
├── README.md
├── LICENSE
├── .gitignore
└── .dockerignore
```

## Commands

```bash
mkdir logistics-optimizer
cd logistics-optimizer

git init

touch README.md
touch LICENSE
touch .gitignore
touch .dockerignore
```

## What to do

Add Python, Docker, environment files, IDE files, caches, secrets and generated files to `.gitignore`.

Example categories:

```text
.env
__pycache__/
.pytest_cache/
.mypy_cache/
.ruff_cache/
.venv/
*.pyc
coverage.xml
htmlcov/
```

## Verify

```bash
git status
```

## Commit

```bash
git add .
git commit -m "chore: initialize repository"
```

---

# 3. Step 02 — Set Up Python and Dependencies

## Goal

Create the Python project configuration and dependency lock file.

Use `uv` for dependency management.

## Create

```text
pyproject.toml
uv.lock
```

## Main dependencies

The project will eventually need packages for:

* FastAPI
* Uvicorn
* SQLAlchemy
* MySQL driver
* Alembic
* Pydantic
* Pydantic Settings
* JWT
* Password hashing
* Pytest
* HTTP testing
* Ruff
* MyPy

## What to do

Initialize the project:

```bash
uv init
```

Add runtime dependencies.

For example:

```bash
uv add fastapi
uv add uvicorn
uv add sqlalchemy
uv add alembic
uv add pydantic-settings
uv add pyjwt
uv add pwdlib
```

Add development dependencies:

```bash
uv add --dev pytest
uv add --dev pytest-asyncio
uv add --dev httpx
uv add --dev ruff
uv add --dev mypy
```

The exact dependency versions should be controlled by `uv.lock`.

## Verify

```bash
uv run python --version
uv run pytest
```

At this point there may not be any tests yet.

## Commit

```bash
git add .
git commit -m "chore: configure Python project"
```

---

# 4. Step 03 — Create the Project Structure

## Goal

Create the application's architecture before implementing business logic.

Create:

```text
app/
├── main.py
├── api/
│   └── v1/
│       ├── router.py
│       └── endpoints/
├── core/
├── db/
├── models/
├── schemas/
├── repositories/
├── services/
├── optimization/
└── workers/

tests/
├── unit/
├── integration/
├── api/
├── factories/
└── conftest.py

scripts/
docker/
compose/
migrations/
```

## What to do

Do not implement everything yet.

The purpose of this step is to establish the architectural boundaries.

## Important rule

Keep the layers separate:

```text
API
 ↓
Service
 ↓
Repository
 ↓
Database
```

Optimization should be separated:

```text
Service
 ↓
Optimization Engine
 ↓
Algorithm
 ↓
Solution
```

## Verify

Check that imports and directories are valid.

## Commit

```bash
git add .
git commit -m "chore: create application architecture"
```

---

# 5. Step 04 — Configuration Management

## Goal

Make configuration environment-based.

## Create

```text
app/core/config.py
.env.example
```

## Configuration categories

Include settings such as:

```text
APP_NAME
APP_ENV
DEBUG

DATABASE_URL

JWT_SECRET_KEY
JWT_ALGORITHM
ACCESS_TOKEN_EXPIRE_MINUTES

LOG_LEVEL
```

Do not put real secrets into Git.

## `.env.example`

Provide safe example values:

```text
APP_NAME=logistics-optimizer
APP_ENV=development
DEBUG=true

DATABASE_URL=mysql+pymysql://app:password@mysql:3306/logistics

JWT_SECRET_KEY=change-me
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

## Verify

The application should be able to load configuration from environment variables.

## Commit

```bash
git add .
git commit -m "feat: add application configuration"
```

---

# 6. Step 05 — Build the Docker Development Environment

## Goal

Run the application and MySQL locally using Docker Compose.

## Create

```text
docker/
├── Dockerfile
└── entrypoint.sh

compose/
└── docker-compose.dev.yml
```

## Services

Initially use:

```text
FastAPI
MySQL
```

Architecture:

```text
Docker Compose
    │
    ├── api
    │
    └── mysql
```

## What to implement

The API container should:

1. Install dependencies.
2. Mount the application during development.
3. Start Uvicorn.
4. Connect to MySQL.

MySQL should:

1. Use a persistent volume.
2. Create the application database.
3. Expose the database internally to the API.

## Verify

```bash
docker compose -f compose/docker-compose.dev.yml up --build
```

Then check:

```text
http://localhost:8000
```

And:

```text
http://localhost:8000/docs
```

## Commit

```bash
git add .
git commit -m "feat: add local Docker environment"
```

---

# 7. Step 06 — Create the FastAPI Application

## Goal

Create the application entry point.

## Create

```text
app/main.py
app/api/v1/router.py
```

## `main.py`

The main responsibility should be application initialization:

```text
Create FastAPI application
        ↓
Register middleware
        ↓
Register exception handlers
        ↓
Register API routers
        ↓
Return application
```

Do not put database queries or optimization algorithms here.

## Create a health endpoint

Example:

```text
GET /health
```

Expected response:

```json
{
  "status": "ok"
}
```

## Verify

```bash
curl http://localhost:8000/health
```

## Commit

```bash
git add .
git commit -m "feat: create FastAPI application"
```

---

# 8. Step 07 — Connect MySQL and SQLAlchemy

## Goal

Create the database layer.

## Create

```text
app/db/base.py
app/db/session.py
```

## Implement

Configure:

```text
SQLAlchemy Engine
        ↓
Connection Pool
        ↓
Session Factory
        ↓
FastAPI Database Dependency
```

The application should be able to obtain a database session inside an API request.

## Verify

Create a simple database connectivity test.

The application should successfully connect to MySQL.

## Commit

```bash
git add .
git commit -m "feat: configure SQLAlchemy database"
```

---

# 9. Step 08 — Configure Alembic

## Goal

Manage database schema changes using migrations.

## Create

```text
alembic.ini
migrations/
├── versions/
└── ...
```

Configure Alembic to use the application's SQLAlchemy metadata.

## Important

Do not manually modify production database tables.

Use migrations:

```text
Model change
    ↓
Alembic migration
    ↓
Migration tested
    ↓
Migration deployed
    ↓
Database updated
```

## Example

Create the first migration:

```bash
uv run alembic revision --autogenerate -m "initial schema"
```

Run:

```bash
uv run alembic upgrade head
```

## Verify

Check the database tables.

## Commit

```bash
git add .
git commit -m "feat: configure database migrations"
```

---

# 10. Step 09 — Implement Authentication

## Goal

Add secure user authentication.

## Create

```text
app/core/security.py
```

Later add authentication endpoints.

## Implement

The security layer should handle:

```text
Password hashing
        ↓
Password verification

User login
        ↓
JWT access token
        ↓
Authenticated API request
```

Implement:

* password hashing
* password verification
* JWT creation
* JWT validation
* authenticated-user dependency

## Important

Never store plaintext passwords.

Never commit:

```text
JWT_SECRET_KEY
database passwords
AWS credentials
```

## Verify

Test:

```text
Register user
      ↓
Login
      ↓
Receive token
      ↓
Access protected endpoint
```

## Commit

```bash
git add .
git commit -m "feat: implement JWT authentication"
```

---

# 11. Step 10 — Implement Database Models

## Goal

Create the application's database representation.

Potential initial models:

```text
User
Vehicle
Order
Route
OptimizationJob
OptimizationResult
```

The exact model design should follow the actual logistics domain requirements.

## Example relationships

```text
User
 │
 ├── Orders
 ├── Vehicles
 └── Optimization Jobs

Optimization Job
 │
 └── Optimization Result
```

## What to do

For each model define:

* primary key
* columns
* data types
* constraints
* indexes
* timestamps
* relationships

## Verify

Generate and run Alembic migrations.

## Commit

```bash
git add .
git commit -m "feat: add core database models"
```

---

# 12. Step 11 — Create Pydantic Schemas

## Goal

Separate API data structures from database models.

Create files such as:

```text
app/schemas/
├── user.py
├── vehicle.py
├── order.py
├── route.py
└── optimization.py
```

## Example structure

For a resource, consider:

```text
CreateSchema
UpdateSchema
ResponseSchema
```

Example:

```text
VehicleCreate
VehicleUpdate
VehicleResponse
```

## Important distinction

```text
SQLAlchemy Model
=
Database representation

Pydantic Schema
=
API representation
```

Do not use database models directly as your API contract.

## Commit

```bash
git add .
git commit -m "feat: add API schemas"
```

---

# 13. Step 12 — Implement Repositories

## Goal

Move database access out of services and endpoints.

Example:

```text
app/repositories/
├── user.py
├── vehicle.py
├── order.py
└── optimization.py
```

Repositories should handle operations such as:

```text
get()
get_by_id()
list()
create()
update()
delete()
```

## Architecture

```text
Endpoint
   ↓
Service
   ↓
Repository
   ↓
SQLAlchemy
   ↓
MySQL
```

## Important

Do not put optimization logic inside repositories.

## Commit

```bash
git add .
git commit -m "feat: implement repository layer"
```

---

# 14. Step 13 — Implement Services

## Goal

Put business/application logic into services.

Example:

```text
app/services/
├── auth.py
├── users.py
├── vehicles.py
├── orders.py
└── optimization.py
```

## Example

Creating an order:

```text
POST /orders
      ↓
OrderEndpoint
      ↓
OrderService
      ↓
OrderRepository
      ↓
MySQL
```

The endpoint should not contain the complete business logic.

## Commit

```bash
git add .
git commit -m "feat: implement service layer"
```

---

# 15. Step 14 — Implement CRUD APIs

## Goal

Create the basic logistics APIs before implementing optimization.

Potential endpoints:

```text
POST   /api/v1/auth/register
POST   /api/v1/auth/login

GET    /api/v1/vehicles
POST   /api/v1/vehicles
GET    /api/v1/vehicles/{id}
PATCH  /api/v1/vehicles/{id}
DELETE /api/v1/vehicles/{id}

GET    /api/v1/orders
POST   /api/v1/orders
GET    /api/v1/orders/{id}
PATCH  /api/v1/orders/{id}
DELETE /api/v1/orders/{id}
```

## Verify

Use:

```text
Swagger UI
Postman
curl
pytest
```

At this point the basic logistics application should work without optimization.

## Commit

```bash
git add .
git commit -m "feat: implement logistics CRUD APIs"
```

---

# 16. Step 15 — Design the Optimization Domain

## Goal

Before writing genetic algorithms or other metaheuristics, define the optimization problem.

Create:

```text
app/optimization/domain/
├── problem.py
├── solution.py
└── constraints.py
```

## Define the problem

For example:

```text
Orders
Vehicles
Depots
Vehicle capacities
Delivery constraints
Time windows
Distances
Costs
```

## Define a solution

A solution represents a candidate delivery plan.

Conceptually:

```text
Vehicle 1
    Depot
      ↓
    Order A
      ↓
    Order C
      ↓
    Depot

Vehicle 2
    Depot
      ↓
    Order B
      ↓
    Order D
      ↓
    Depot
```

## Define constraints

Examples:

```text
Vehicle capacity
Delivery time window
Vehicle availability
Route feasibility
Maximum route duration
Order assignment
```

## Important

The algorithm should not decide what a valid logistics solution means.

The domain model should define validity.

## Commit

```bash
git add .
git commit -m "feat: define optimization domain"
```

---

# 17. Step 16 — Implement Objective Functions

## Goal

Define how a solution is evaluated.

Create:

```text
app/optimization/objectives/
├── distance.py
├── cost.py
├── time.py
└── penalties.py
```

Possible objective components:

```text
Total distance
Total fuel cost
Vehicle operating cost
Delivery time
Late delivery penalty
Capacity violation penalty
Constraint violation penalty
```

A simplified objective might look conceptually like:

```text
Total Score =
    distance cost
  + vehicle cost
  + time cost
  + constraint penalties
```

## Important

Keep objective calculation separate from the algorithm.

That allows the same objective to be used by:

```text
Genetic Algorithm
Simulated Annealing
Tabu Search
Ant Colony Optimization
```

## Commit

```bash
git add .
git commit -m "feat: implement optimization objectives"
```

---

# 18. Step 17 — Implement Optimization Operators

## Goal

Implement reusable operators.

Create:

```text
app/optimization/operators/
```

Possible operators:

```text
selection.py
crossover.py
mutation.py
repair.py
initialization.py
```

## Genetic Algorithm example

```text
Population
    ↓
Evaluate
    ↓
Selection
    ↓
Crossover
    ↓
Mutation
    ↓
Repair
    ↓
New Population
```

## Important

The operators should work with the domain's `Solution` representation.

They should not directly query MySQL.

## Commit

```bash
git add .
git commit -m "feat: implement optimization operators"
```

---

# 19. Step 18 — Implement Optimization Algorithms

## Goal

Implement the actual metaheuristics.

Create:

```text
app/optimization/algorithms/
├── base.py
├── genetic.py
├── simulated_annealing.py
└── ...
```

Start with one algorithm.

A sensible implementation order is:

```text
1. Base algorithm interface
2. Genetic Algorithm
3. Simulated Annealing
4. Additional algorithms
```

## Base interface

Conceptually:

```text
OptimizationAlgorithm
        │
        ├── solve(problem)
        │
        └── returns solution
```

Then:

```text
GeneticAlgorithm
SimulatedAnnealing
TabuSearch
...
```

can implement the same interface.

## Important

The algorithms should not know about:

```text
FastAPI
HTTP requests
JWT
SQLAlchemy
MySQL
```

They should operate on optimization-domain objects.

## Commit

```bash
git add .
git commit -m "feat: implement optimization algorithms"
```

---

# 20. Step 19 — Implement the Optimization Engine

## Goal

Create a common orchestration layer.

Create:

```text
app/optimization/engine.py
```

The engine should:

```text
Receive optimization problem
        ↓
Choose algorithm
        ↓
Configure algorithm
        ↓
Execute algorithm
        ↓
Validate solution
        ↓
Return result
```

Example:

```text
algorithm = "genetic"
```

could select:

```text
GeneticAlgorithm
```

while:

```text
algorithm = "simulated_annealing"
```

selects:

```text
SimulatedAnnealing
```

## Commit

```bash
git add .
git commit -m "feat: add optimization engine"
```

---

# 21. Step 20 — Create Optimization Jobs

## Goal

Connect the business layer to the optimization engine.

Create or implement:

```text
app/services/optimization.py
```

Flow:

```text
API request
    ↓
OptimizationService
    ↓
Load orders
    ↓
Load vehicles
    ↓
Build OptimizationProblem
    ↓
OptimizationEngine
    ↓
Algorithm
    ↓
OptimizationSolution
    ↓
Save result
```

## Optimization job model

A job can contain:

```text
id
status
algorithm
parameters
created_at
started_at
completed_at
error
```

Possible statuses:

```text
PENDING
RUNNING
COMPLETED
FAILED
```

## Commit

```bash
git add .
git commit -m "feat: implement optimization jobs"
```

---

# 22. Step 21 — Implement the Optimization API

## Goal

Expose optimization functionality through the API.

Potential endpoints:

```text
POST /api/v1/optimization/jobs
GET  /api/v1/optimization/jobs
GET  /api/v1/optimization/jobs/{id}
GET  /api/v1/optimization/jobs/{id}/result
```

## Example workflow

```text
POST /optimization/jobs
        ↓
Create job
        ↓
Return job ID
        ↓
Worker executes optimization
        ↓
GET /optimization/jobs/{id}
        ↓
Check status
        ↓
GET /optimization/jobs/{id}/result
```

## Important

Do not make a long-running optimization algorithm block the HTTP request if the optimization can take significant time.

## Commit

```bash
git add .
git commit -m "feat: add optimization API"
```

---

# 23. Step 22 — Add Background Workers

## Goal

Move long-running optimization jobs out of the API request process.

Create:

```text
app/workers/
```

The eventual architecture becomes:

```text
Client
   ↓
FastAPI
   ↓
Create Optimization Job
   ↓
Queue / Worker
   ↓
Optimization Engine
   ↓
Database
```

The worker should:

1. Pick up a job.
2. Mark it `RUNNING`.
3. Load required data.
4. Execute optimization.
5. Save the result.
6. Mark it `COMPLETED`.
7. Mark it `FAILED` if an error occurs.

A queue system can be introduced when the workload requires it.

## Commit

```bash
git add .
git commit -m "feat: add optimization workers"
```

---

# 24. Step 23 — Add Automated Tests

## Goal

Build confidence before deployment.

Tests should exist at several levels.

## Unit tests

```text
tests/unit/
```

Test:

```text
Objective functions
Constraints
Mutation
Crossover
Selection
Algorithms
Services
```

These should be fast and isolated.

## Integration tests

```text
tests/integration/
```

Test:

```text
SQLAlchemy
MySQL
Repositories
Database transactions
```

## API tests

```text
tests/api/
```

Test:

```text
Authentication
Authorization
CRUD endpoints
Optimization endpoints
Error responses
```

## Test pyramid

```text
        API tests
       /         \
 Integration     \
     tests        \
       /            \
   Unit tests        \
```

Most tests should be unit tests.

## Run

```bash
uv run pytest
```

## Commit

```bash
git add .
git commit -m "test: add application test suite"
```

---

# 25. Step 24 — Add Logging and Exception Handling

## Goal

Make failures understandable and production-safe.

Create:

```text
app/core/logging.py
app/core/exceptions.py
```

## Logging should capture

```text
Request information
Authentication events
Optimization job lifecycle
Database errors
Unexpected exceptions
Application startup/shutdown
```

Do not log:

```text
Passwords
JWT secrets
Database passwords
Sensitive credentials
```

## Exception architecture

```text
Domain Exception
        ↓
Application Exception
        ↓
FastAPI Exception Handler
        ↓
HTTP Response
```

## Commit

```bash
git add .
git commit -m "feat: add logging and exception handling"
```

---

# 26. Step 25 — Production Docker Image

## Goal

Create a production-ready container.

Create:

```text
docker/Dockerfile.prod
```

Production image should ideally:

* use a small base image
* install only required dependencies
* run as a non-root user
* avoid development tools
* use environment variables
* have predictable startup behavior

## Important

Do not automatically run dangerous/destructive operations during every container startup.

Database migrations should be an explicit deployment step or controlled initialization step.

## Commit

```bash
git add .
git commit -m "build: add production Docker image"
```

---

# 27. Step 26 — Add Nginx

## Goal

Put Nginx in front of FastAPI in production.

Create:

```text
docker/nginx/nginx.conf
```

Architecture:

```text
Internet
    ↓
Nginx
    ↓
FastAPI
    ↓
Worker
    ↓
MySQL
```

Nginx can handle:

* reverse proxy
* connection handling
* request limits
* headers
* HTTPS termination
* timeouts

## Commit

```bash
git add .
git commit -m "build: add production reverse proxy"
```

---

# 28. Step 27 — Production Docker Compose

## Goal

Create the production service definition.

Create:

```text
compose/docker-compose.prod.yml
```

Potential services:

```text
nginx
api
worker
mysql
```

For AWS, MySQL can later be moved from the EC2 host to RDS.

Initial architecture:

```text
EC2
 │
 ├── Nginx
 ├── FastAPI
 ├── Worker
 └── MySQL
```

Later:

```text
EC2
 │
 ├── Nginx
 ├── FastAPI
 └── Worker

AWS RDS
 └── MySQL
```

## Commit

```bash
git add .
git commit -m "build: add production compose configuration"
```

---

# 29. Step 28 — Create Operational Scripts

## Goal

Make common operations easy to execute.

Create:

```text
scripts/
├── dev.sh
├── test.sh
├── lint.sh
├── migrate.sh
├── deploy.sh
├── healthcheck.sh
└── backup.sh
```

## Responsibilities

### `dev.sh`

Start local development environment.

```bash
./scripts/dev.sh
```

### `test.sh`

Run tests.

```bash
./scripts/test.sh
```

### `lint.sh`

Run formatting/linting/type checking.

```bash
./scripts/lint.sh
```

### `migrate.sh`

Run database migrations.

```bash
./scripts/migrate.sh
```

### `healthcheck.sh`

Check the running application.

```bash
./scripts/healthcheck.sh
```

### `backup.sh`

Create a database backup.

```bash
./scripts/backup.sh
```

### `deploy.sh`

Perform deployment operations.

Keep deployment logic explicit and safe.

## Commit

```bash
git add .
git commit -m "chore: add operational scripts"
```

---

# 30. Step 29 — Create the Makefile

## Goal

Provide a simple developer interface.

Example commands:

```text
make install
make dev
make test
make lint
make format
make migrate
make migration
make build
make up
make down
make logs
make health
```

Instead of remembering long commands:

```bash
docker compose -f compose/docker-compose.dev.yml up --build
```

developers can use:

```bash
make up
```

## Commit

```bash
git add .
git commit -m "chore: add Makefile commands"
```

---

# 31. Step 30 — Implement CI

## Goal

Automatically verify every change pushed to GitHub.

Create:

```text
.github/workflows/ci.yml
```

The CI pipeline should approximately perform:

```text
Push / Pull Request
        ↓
Checkout
        ↓
Install Python
        ↓
Install dependencies
        ↓
Lint
        ↓
Type check
        ↓
Unit tests
        ↓
Integration tests
        ↓
Build Docker image
```

## Example pipeline stages

```text
lint
test
build
```

A pull request should not be merged if required checks fail.

## Commit

```bash
git add .
git commit -m "ci: add continuous integration"
```

---

# 32. Step 31 — Prepare AWS EC2

## Goal

Create the production host.

Initial deployment target:

```text
AWS EC2
```

The EC2 server should have:

* Docker
* Docker Compose
* Git
* appropriate firewall/security-group configuration

The server should not contain application secrets in Git.

Use environment variables or an appropriate secrets mechanism.

## Security groups

Only expose required ports.

Typically:

```text
80
443
```

SSH access should be restricted appropriately.

Do not expose MySQL publicly unless there is a specific requirement.

---

# 33. Step 32 — Deploy Manually First

## Goal

Before creating automatic deployment, prove that the application can be deployed manually.

Deployment flow:

```text
GitHub
   ↓
Build image
   ↓
Push image
   ↓
EC2
   ↓
Pull image
   ↓
Run migration
   ↓
Restart application
   ↓
Health check
```

## Verify

Check:

```text
https://your-domain/health
```

and:

```text
https://your-domain/docs
```

Do the manual deployment before implementing CD.

This makes debugging much easier.

---

# 34. Step 33 — Container Registry

## Goal

Store production Docker images.

For AWS deployment, use a container registry such as Amazon ECR.

Architecture:

```text
GitHub Actions
      ↓
Build Docker Image
      ↓
Push Image
      ↓
Amazon ECR
      ↓
EC2 pulls image
```

Use immutable image tags where practical.

For example:

```text
commit SHA
```

rather than relying only on:

```text
latest
```

---

# 35. Step 34 — Implement CD

## Goal

Automatically deploy successful builds.

Create:

```text
.github/workflows/cd.yml
```

Possible workflow:

```text
Push to main
     ↓
Run CI
     ↓
Build production image
     ↓
Push image to ECR
     ↓
Connect/deploy to EC2
     ↓
Pull new image
     ↓
Run migrations
     ↓
Restart services
     ↓
Health check
```

## Important

Deployment should fail if:

```text
Image build fails
Migration fails
Container fails
Health check fails
```

Do not consider the deployment successful merely because the Docker container started.

## Commit

```bash
git add .
git commit -m "ci: add continuous deployment"
```

---

# 36. Step 35 — Database Backups

## Goal

Protect application data.

Create:

```text
scripts/backup.sh
```

The backup strategy should define:

```text
What is backed up?
Where is it stored?
How often?
How long is it retained?
How is restoration tested?
```

If MySQL is moved to RDS, use the database service's backup capabilities where appropriate.

## Important

A backup is only useful if restoration works.

Test restoration periodically.

---

# 37. Step 36 — Health Checks

## Goal

Make deployment and operations observable.

The API should expose:

```text
GET /health
```

You may also create a deeper readiness check that verifies required dependencies.

For example:

```text
Application running
        +
Database reachable
        +
Required services available
```

Be careful not to expose sensitive diagnostic information publicly.

---

# 38. Step 37 — Final End-to-End Test

Before considering version 1 complete, test:

```text
User registration
       ↓
Login
       ↓
JWT authentication
       ↓
Create vehicle
       ↓
Create orders
       ↓
Create optimization job
       ↓
Worker executes job
       ↓
Algorithm generates solution
       ↓
Solution saved
       ↓
Retrieve result
```

Also test failure scenarios:

```text
Invalid JWT
Invalid request
Missing vehicle
Invalid order
Database failure
Optimization failure
Worker failure
Migration failure
Container failure
```

---

# 39. Final Production Architecture

After completing the implementation, the architecture should look approximately like:

```text
                        Internet
                           │
                           ▼
                        Nginx
                           │
                           ▼
                    ┌─────────────┐
                    │   FastAPI   │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        Application                 Optimization
          Services                    Worker
              │                         │
              │                         ▼
              │                  Optimization
              │                     Engine
              │                         │
              │                  ┌──────┴──────┐
              │                  │             │
              │                  ▼             ▼
              │              Genetic       Other
              │              Algorithm    Algorithms
              │
              ▼
        Repositories
              │
              ▼
        MySQL / RDS
```

---

# 40. Recommended Implementation Order

Do not implement the project in random order.

Use this sequence:

```text
1.  Repository
2.  Python environment
3.  Project structure
4.  Configuration
5.  Docker development environment
6.  FastAPI
7.  MySQL / SQLAlchemy
8.  Alembic
9.  Authentication
10. Database models
11. Pydantic schemas
12. Repositories
13. Services
14. CRUD APIs
15. Optimization domain
16. Objective functions
17. Operators
18. Algorithm
19. Optimization engine
20. Optimization jobs
21. Optimization API
22. Workers
23. Tests
24. Logging
25. Production Docker
26. Nginx
27. Production Compose
28. Bash scripts
29. Makefile
30. CI
31. AWS EC2
32. Manual deployment
33. ECR
34. CD
35. Backups
36. Monitoring/health checks
37. End-to-end testing
```

---

# 41. Definition of Done

The project should not be considered complete just because the API starts.

The project is complete when:

### Application

* [ ] FastAPI starts successfully.
* [ ] API documentation works.
* [ ] Authentication works.
* [ ] JWT protection works.
* [ ] CRUD operations work.
* [ ] Optimization jobs can be created.
* [ ] Optimization algorithms produce valid solutions.
* [ ] Results are persisted.

### Database

* [ ] MySQL works.
* [ ] SQLAlchemy works.
* [ ] Alembic migrations work.
* [ ] Database indexes/constraints are implemented.
* [ ] Backup process exists.
* [ ] Restore process has been tested.

### Optimization

* [ ] Problem representation exists.
* [ ] Solution representation exists.
* [ ] Constraints exist.
* [ ] Objective functions exist.
* [ ] Operators exist.
* [ ] At least one metaheuristic works.
* [ ] Optimization engine exists.
* [ ] Optimization jobs can run asynchronously.

### Testing

* [ ] Unit tests exist.
* [ ] Integration tests exist.
* [ ] API tests exist.
* [ ] Failure cases are tested.
* [ ] CI runs tests automatically.

### Docker

* [ ] Development image works.
* [ ] Production image works.
* [ ] Development Compose works.
* [ ] Production Compose works.

### Deployment

* [ ] EC2 deployment works.
* [ ] Production environment variables are configured securely.
* [ ] ECR image publishing works.
* [ ] CD workflow works.
* [ ] Database migration process works.
* [ ] Health checks work.
* [ ] Backups exist.

---

# 42. Development Rule

The most important rule for this project is:

> **Build one layer at a time and make every layer testable before connecting it to the next layer.**

Do not start by writing the genetic algorithm and then try to build the application around it.

Instead:

```text
Infrastructure
      ↓
API
      ↓
Database
      ↓
Business logic
      ↓
Optimization domain
      ↓
Optimization algorithm
      ↓
Workers
      ↓
Testing
      ↓
Deployment
```

This keeps the logistics optimization algorithm independent from FastAPI, MySQL and Docker, which makes the system easier to test, extend and maintain.

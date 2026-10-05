# Project Structure

This document explains the structure, responsibilities, and architectural purpose of the files and directories in the **Logistics Optimizer** project.

The goal is to keep the codebase maintainable as the system grows from a local development project into a production-oriented logistics optimization platform.

---

# 1. Repository Overview

The project is organized into several major areas:

```text
logistics-optimizer/
│
├── .github/          # CI/CD automation
├── app/              # Main application
├── migrations/       # Database migrations
├── tests/            # Automated tests
├── scripts/          # Operational Bash scripts
├── docker/           # Docker configuration
├── compose/          # Docker Compose configuration
│
├── .env.example      # Environment variable template
├── Makefile          # Developer commands
├── pyproject.toml    # Python project configuration
├── uv.lock           # Locked Python dependencies
├── alembic.ini       # Alembic configuration
├── README.md         # Project documentation
└── LICENSE           # Project license
```

The main application architecture is:

```text
Client
   │
   ▼
API Endpoint
   │
   ▼
Service
   │
   ├───────────────┐
   ▼               ▼
Repository     Optimization
   │              Engine
   ▼               │
 MySQL             ▼
              Algorithm
```

---

# 2. `.github/`

```text
.github/
└── workflows/
    ├── ci.yml
    └── cd.yml
```

Contains GitHub Actions workflows.

The directory is responsible for automating development and deployment processes.

---

## `ci.yml`

### Purpose

Continuous Integration.

It verifies that new code is working before it is merged.

Typical pipeline:

```text
Push / Pull Request
        │
        ▼
Install dependencies
        │
        ▼
Lint
        │
        ▼
Type check
        │
        ▼
Run tests
        │
        ▼
Build Docker image
```

Typical checks include:

* Ruff
* MyPy
* Pytest
* Docker build
* Dependency/security checks

---

## `cd.yml`

### Purpose

Continuous Deployment.

It is responsible for deploying the application to the production environment.

Typical flow:

```text
main
 │
 ▼
GitHub Actions
 │
 ▼
Build Docker image
 │
 ▼
Push image to registry
 │
 ▼
Deploy to AWS EC2
 │
 ▼
Run migrations
 │
 ▼
Health check
```

---

# 3. `app/`

```text
app/
├── main.py
├── api/
├── core/
├── db/
├── models/
├── schemas/
├── repositories/
├── services/
├── optimization/
└── workers/
```

This is the main Python application.

It contains:

* FastAPI application
* API endpoints
* configuration
* security
* database integration
* database models
* API schemas
* repositories
* business services
* optimization engine
* background workers

---

# 4. `app/main.py`

### Purpose

Application entry point.

Responsibilities include:

* Create FastAPI application
* Configure middleware
* Configure logging
* Register API routers
* Register exception handlers
* Configure application startup/shutdown behavior

The file should remain relatively small.

It should **not** contain:

* SQL queries
* business logic
* JWT implementation
* optimization algorithms

---

# 5. `app/api/`

```text
app/api/
└── v1/
    ├── router.py
    └── endpoints/
```

Contains the HTTP/API layer.

This layer is responsible for communicating with clients.

It should primarily deal with:

* HTTP requests
* HTTP responses
* request validation
* authentication dependencies
* API routing
* status codes

Business logic belongs in `services/`.

---

# 6. `app/api/v1/`

The API is versioned so that future API changes can be introduced without immediately breaking existing clients.

Current version:

```text
/api/v1/
```

Future versions could potentially be:

```text
/api/v2/
```

---

## `app/api/v1/router.py`

### Purpose

Central API router.

It combines the individual endpoint routers.

Conceptually:

```text
v1 router
│
├── auth
├── users
├── vehicles
├── orders
├── routes
└── optimization
```

---

# 7. `app/api/v1/endpoints/`

Contains individual API endpoint modules.

Potential structure:

```text
endpoints/
├── auth.py
├── users.py
├── vehicles.py
├── orders.py
├── routes.py
└── optimization.py
```

Each file should focus on a particular API resource.

---

## `auth.py`

Authentication endpoints.

Examples:

```text
POST /auth/register
POST /auth/login
POST /auth/refresh
```

The endpoint calls the authentication service rather than implementing authentication logic itself.

---

## `users.py`

User-related endpoints.

Examples:

```text
GET /users/me
GET /users/{id}
PATCH /users/{id}
```

---

## `vehicles.py`

Vehicle management endpoints.

Examples:

```text
POST /vehicles
GET /vehicles
GET /vehicles/{id}
PATCH /vehicles/{id}
DELETE /vehicles/{id}
```

---

## `orders.py`

Logistics order endpoints.

Examples:

```text
POST /orders
GET /orders
GET /orders/{id}
PATCH /orders/{id}
DELETE /orders/{id}
```

---

## `routes.py`

Route-related endpoints.

Examples:

```text
GET /routes
GET /routes/{id}
POST /routes
```

---

## `optimization.py`

Optimization API endpoints.

Examples:

```text
POST /optimization/jobs
GET /optimization/jobs/{job_id}
GET /optimization/jobs/{job_id}/result
```

This module should **not** contain the actual optimization algorithm.

It communicates with:

```text
OptimizationService
        ↓
OptimizationEngine
```

---

# 8. `app/core/`

```text
core/
├── config.py
├── security.py
├── logging.py
└── exceptions.py
```

Contains application-wide infrastructure.

---

## `config.py`

### Purpose

Centralized configuration.

It manages values such as:

```text
APP_ENV
DATABASE_URL
JWT_SECRET_KEY
JWT_ALGORITHM
JWT expiration
CORS settings
logging configuration
```

Configuration should come from environment variables rather than being hardcoded.

Conceptually:

```text
.env
 │
 ▼
config.py
 │
 ▼
Application
```

---

## `security.py`

### Purpose

Security primitives.

Contains functionality such as:

```text
password hashing
password verification
JWT creation
JWT decoding
JWT validation
```

Examples:

```text
hash_password()
verify_password()
create_access_token()
decode_token()
```

Authentication business logic belongs in `services/auth.py`.

---

## `logging.py`

### Purpose

Centralized application logging.

Responsible for configuring:

* log level
* log format
* console logging
* production logging
* structured logging

Application code should use the configured logger rather than relying on `print()`.

---

## `exceptions.py`

### Purpose

Application-specific exceptions.

Examples:

```text
UserNotFoundError
OrderNotFoundError
VehicleNotFoundError
OptimizationFailedError
InvalidOptimizationProblemError
ConstraintViolationError
```

Central exception handlers can convert these into appropriate HTTP responses.

---

# 9. `app/db/`

```text
db/
├── base.py
└── session.py
```

Contains database infrastructure.

---

## `session.py`

### Purpose

SQLAlchemy database configuration.

Responsible for:

* database engine
* connection pool
* session factory
* database session dependency

Conceptually:

```text
FastAPI request
       │
       ▼
Database Session
       │
       ▼
Repository
       │
       ▼
MySQL
```

---

## `base.py`

### Purpose

Defines the SQLAlchemy declarative base/metadata.

Database models inherit from this base.

Example:

```text
Base
 │
 ├── User
 ├── Vehicle
 ├── Order
 └── Route
```

Alembic uses this metadata when generating migrations.

---

# 10. `app/models/`

Contains SQLAlchemy database models.

Potential models:

```text
models/
├── user.py
├── vehicle.py
├── order.py
├── route.py
└── optimization.py
```

Models represent database tables and relationships.

For example:

```text
User
Vehicle
Order
Route
OptimizationJob
OptimizationResult
```

Important distinction:

```text
Model
    =
database representation
```

while:

```text
Schema
    =
API data contract
```

---

# 11. `app/schemas/`

Contains Pydantic schemas.

Potential structure:

```text
schemas/
├── auth.py
├── user.py
├── vehicle.py
├── order.py
├── route.py
└── optimization.py
```

Schemas validate API requests and define API responses.

Example:

```text
POST /vehicles

Request:

{
    "name": "Truck 01",
    "capacity": 5000
}
```

A Pydantic schema validates this data before it reaches the service layer.

---

# 12. `app/repositories/`

Contains database access logic.

Potential structure:

```text
repositories/
├── user.py
├── vehicle.py
├── order.py
└── optimization.py
```

Repositories answer:

> How do we retrieve or persist data?

Examples:

```text
UserRepository
    create()
    get_by_id()
    get_by_email()

OrderRepository
    create()
    get_by_id()
    list()
```

Repositories should focus on persistence rather than business decisions.

---

# 13. `app/services/`

Contains application/business logic.

Potential structure:

```text
services/
├── auth.py
├── user.py
├── order.py
├── route.py
└── optimization.py
```

Services answer:

> What should the application do?

Typical flow:

```text
Endpoint
   │
   ▼
Service
   │
   ▼
Repository
   │
   ▼
Database
```

Services may also communicate with the optimization engine.

---

## `services/auth.py`

Responsible for authentication business logic.

Examples:

```text
register user
validate credentials
login
create token
refresh token
```

---

## `services/user.py`

Responsible for user business operations.

Examples:

```text
create user
update user
deactivate user
get profile
```

---

## `services/order.py`

Responsible for order-related business rules.

Examples:

```text
create order
validate order
update order
cancel order
validate delivery constraints
```

---

## `services/route.py`

Responsible for route business logic.

Examples:

```text
create route
validate route
assign route
calculate route information
update route status
```

---

## `services/optimization.py`

Acts as the bridge between the API/business layer and the optimization engine.

Typical flow:

```text
API
 │
 ▼
OptimizationService
 │
 ├── load orders
 ├── load vehicles
 ├── validate problem
 ├── create optimization problem
 │
 ▼
OptimizationEngine
 │
 ▼
OptimizationSolution
 │
 ▼
save result
```

---

# 14. `app/optimization/`

This is the core logistics optimization module.

```text
optimization/
├── domain/
├── algorithms/
├── operators/
├── objectives/
└── engine.py
```

The optimization layer should have minimal dependency on FastAPI.

This allows algorithms to be tested independently.

---

# 15. `optimization/domain/`

```text
domain/
├── problem.py
├── solution.py
└── constraints.py
```

Contains the core concepts of the optimization problem.

---

## `problem.py`

Defines the optimization problem.

Possible data:

```text
vehicles
orders
locations
demands
time windows
vehicle capacities
depots
optimization configuration
```

Conceptually:

```text
OptimizationProblem
├── vehicles
├── orders
├── locations
├── constraints
└── configuration
```

---

## `solution.py`

Defines the result produced by an optimization algorithm.

Possible information:

```text
routes
objective value
total distance
total cost
execution time
vehicles used
constraint violations
```

Conceptually:

```text
OptimizationSolution
├── routes
├── objective_value
├── total_distance
├── total_cost
├── execution_time
└── violations
```

---

## `constraints.py`

Contains optimization constraints.

Examples:

```text
vehicle capacity
time windows
maximum route duration
vehicle availability
depot constraints
delivery requirements
```

Constraints should ideally be reusable across algorithms.

---

# 16. `optimization/algorithms/`

Contains actual metaheuristic implementations.

```text
algorithms/
├── genetic_algorithm.py
├── simulated_annealing.py
├── tabu_search.py
└── ant_colony.py
```

Each algorithm should implement a clear interface.

For example:

```text
solve(problem) -> solution
```

The exact interface can evolve as the optimization architecture becomes clearer.

---

## `genetic_algorithm.py`

Contains the Genetic Algorithm implementation.

Main concepts:

```text
population
fitness
selection
crossover
mutation
generations
termination
```

---

## `simulated_annealing.py`

Contains Simulated Annealing.

Main concepts:

```text
current solution
neighbor generation
temperature
cooling schedule
acceptance probability
termination
```

---

## `tabu_search.py`

Contains Tabu Search.

Main concepts:

```text
current solution
neighborhood
tabu list
candidate selection
aspiration criteria
termination
```

---

## `ant_colony.py`

Contains Ant Colony Optimization.

Main concepts:

```text
pheromone
heuristic information
solution construction
pheromone evaporation
pheromone update
```

---

# 17. `optimization/operators/`

Contains reusable algorithm operations.

```text
operators/
├── mutation.py
├── crossover.py
└── selection.py
```

These are particularly useful for Genetic Algorithms.

Examples:

```text
mutation()
crossover()
selection()
```

Keeping them separate makes them independently testable and reusable.

---

# 18. `optimization/objectives/`

Contains objective functions.

```text
objectives/
├── distance.py
├── cost.py
├── time.py
└── penalties.py
```

Possible objectives:

```text
minimize total distance
minimize transportation cost
minimize delivery time
minimize number of vehicles
minimize constraint violations
```

This structure also leaves room for future multi-objective optimization.

---

# 19. `optimization/engine.py`

The optimization engine is the main coordinator.

Conceptually:

```text
OptimizationProblem
        │
        ▼
OptimizationEngine
        │
        ├── select algorithm
        │
        ├── configure algorithm
        │
        ├── execute
        │
        └── validate result
        │
        ▼
OptimizationSolution
```

The service layer should not need to know how Genetic Algorithm or Tabu Search works internally.

---

# 20. `app/workers/`

Contains background processing.

Potential file:

```text
workers/
└── optimization_worker.py
```

Useful when optimization jobs become long-running.

Instead of:

```text
HTTP request
     ↓
wait 2 minutes
     ↓
HTTP response
```

the application can use:

```text
HTTP request
     ↓
create optimization job
     ↓
return job ID
```

Then:

```text
Worker
   ↓
retrieve job
   ↓
run optimization
   ↓
store result
```

A queue/background-processing technology can be introduced later.

---

# 21. `migrations/`

Contains database schema migration history.

```text
migrations/
└── versions/
```

Migration history might eventually look like:

```text
001_create_users.py
002_create_vehicles.py
003_create_orders.py
004_create_routes.py
005_create_optimization_jobs.py
```

Alembic manages these migrations.

---

# 22. `migrations/versions/`

Contains individual migration files.

Each migration represents a database schema change.

Example:

```text
create users
      ↓
create vehicles
      ↓
create orders
      ↓
create routes
      ↓
create optimization jobs
```

Production databases should be updated through controlled migrations.

---

# 23. `tests/`

```text
tests/
├── unit/
├── integration/
├── api/
├── factories/
└── conftest.py
```

Contains automated tests.

Testing is separated into different levels so that fast tests and infrastructure-dependent tests can be handled appropriately.

---

# 24. `tests/unit/`

Tests individual components in isolation.

Examples:

```text
test_password_hashing.py
test_route_cost.py
test_capacity_constraint.py
test_genetic_algorithm.py
test_mutation.py
```

Unit tests should generally avoid external dependencies such as:

```text
MySQL
AWS
real network services
```

They should be fast.

---

# 25. `tests/integration/`

Tests interactions between multiple application components.

Examples:

```text
API
 ↓
Service
 ↓
Repository
 ↓
MySQL
```

Potential tests:

```text
user registration
login
order creation
optimization job creation
database persistence
```

---

# 26. `tests/api/`

Tests the HTTP API contract.

Examples:

```text
POST /auth/login
GET /users/me
POST /vehicles
POST /orders
POST /optimization/jobs
```

These tests verify that the API behaves correctly from a client's perspective.

---

# 27. `tests/factories/`

Contains reusable test data factories.

Examples:

```text
UserFactory
VehicleFactory
OrderFactory
RouteFactory
OptimizationProblemFactory
```

This avoids duplicating large amounts of test setup code.

---

# 28. `tests/conftest.py`

Contains shared Pytest fixtures.

Potential fixtures:

```text
test database
database session
FastAPI test client
authenticated user
JWT token
sample vehicle
sample order
```

This becomes especially useful as the test suite grows.

---

# 29. `scripts/`

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

Contains Bash scripts for common operational tasks.

The scripts should primarily orchestrate existing tools rather than contain application business logic.

---

# 30. `scripts/dev.sh`

Starts or prepares the local development environment.

Possible responsibilities:

```text
validate environment
start Docker services
wait for MySQL
run migrations
start application
```

---

# 31. `scripts/test.sh`

Runs the test suite.

For example:

```text
start test environment
        ↓
pytest
        ↓
return success/failure
```

The same script can potentially be used locally and by CI.

---

# 32. `scripts/lint.sh`

Runs code-quality tools.

Potential commands:

```text
ruff
mypy
format checking
```

---

# 33. `scripts/migrate.sh`

Runs database migrations.

Typical command:

```bash
alembic upgrade head
```

The script can also validate that required environment variables exist.

---

# 34. `scripts/deploy.sh`

Deployment helper.

Potential responsibilities:

```text
pull Docker image
run database migrations
restart containers
wait for application
run health check
```

Production deployment should be designed so that failures are detected rather than silently ignored.

---

# 35. `scripts/healthcheck.sh`

Checks application health.

Potentially:

```text
GET /health
```

and verifies that the application responds correctly.

Useful for:

* Docker
* deployment
* CI/CD
* monitoring

---

# 36. `scripts/backup.sh`

Database backup functionality.

Potentially:

```text
MySQL
 ↓
mysqldump
 ↓
compressed backup
 ↓
backup storage
```

The final implementation will depend on whether MySQL is hosted directly on EC2 or moved to AWS RDS.

---

# 37. `docker/`

```text
docker/
├── Dockerfile
├── Dockerfile.prod
├── entrypoint.sh
└── nginx/
    └── nginx.conf
```

Contains Docker-specific configuration.

---

# 38. `docker/Dockerfile`

Defines the application container image.

Typical contents:

```text
Python base image
system dependencies
Python dependencies
application source
startup command
```

Used primarily for development/general builds.

---

# 39. `docker/Dockerfile.prod`

Production-oriented Docker image.

Potential improvements over the normal Dockerfile:

```text
multi-stage build
smaller image
non-root user
fewer packages
production-only dependencies
security hardening
```

---

# 40. `docker/entrypoint.sh`

Executed when the container starts.

Potential responsibilities:

```text
validate configuration
perform initialization
start application
```

Database migrations should be handled carefully and may be executed explicitly by the deployment process instead of automatically on every container startup.

---

# 41. `docker/nginx/nginx.conf`

Nginx configuration.

Potential responsibilities:

```text
reverse proxy
HTTP/HTTPS
headers
timeouts
request forwarding
```

Typical architecture:

```text
Internet
   │
   ▼
Nginx :443
   │
   ▼
FastAPI :8000
```

---

# 42. `compose/`

```text
compose/
├── docker-compose.yml
├── docker-compose.dev.yml
└── docker-compose.prod.yml
```

Contains Docker Compose configurations.

Compose defines how multiple containers work together.

---

# 43. `docker-compose.yml`

Contains common/shared Compose configuration.

Possible components:

```text
networks
volumes
common environment
shared service configuration
```

---

# 44. `docker-compose.dev.yml`

Development environment.

Potential services:

```text
FastAPI
MySQL
```

Development-specific features can include:

```text
hot reload
source-code volumes
development ports
debug configuration
```

---

# 45. `docker-compose.prod.yml`

Production environment.

Potential services:

```text
Nginx
FastAPI
Optimization Worker
MySQL
```

If MySQL is later moved to AWS RDS, the production Compose configuration can remove the MySQL container and connect FastAPI to RDS.

---

# 46. `.env.example`

Template for environment variables.

Example:

```env
APP_ENV=development

DATABASE_URL=mysql+pymysql://user:password@mysql:3306/logistics

JWT_SECRET_KEY=change-me
JWT_ALGORITHM=HS256

MYSQL_DATABASE=logistics
MYSQL_USER=logistics
MYSQL_PASSWORD=change-me
```

This file **should be committed**.

It must contain example values rather than production secrets.

---

# 47. `.gitignore`

Defines files that Git should ignore.

Typical entries:

```text
.env
.venv/
__pycache__/
.pytest_cache/
.mypy_cache/
.ruff_cache/
*.log
```

Secrets and generated files should not be committed.

---

# 48. `.dockerignore`

Defines files that should not be sent to the Docker build context.

Potential entries:

```text
.git
.github
.venv
__pycache__
.pytest_cache
.env
```

This can reduce build context size and prevent accidental inclusion of unnecessary files.

---

# 49. `Makefile`

Provides convenient developer commands.

Instead of:

```bash
docker compose -f compose/docker-compose.dev.yml up -d
```

developers can use:

```bash
make up
```

Potential commands:

```text
make up
make down
make restart
make logs

make test
make test-unit
make test-integration

make lint
make format
make typecheck

make migrate
make migration

make shell
make db-shell

make build
make deploy
```

The Makefile becomes the main command interface for developers.

---

# 50. `pyproject.toml`

Main Python project configuration.

Potentially contains:

```text
project metadata
Python version
runtime dependencies
development dependencies
Pytest configuration
Ruff configuration
MyPy configuration
build configuration
```

It becomes one of the central configuration files for the Python project.

---

# 51. `uv.lock`

Locks the exact versions of Python dependencies.

The goal is reproducibility:

```text
Developer machine
       =
CI
       =
Docker
       =
Production
```

as far as dependency versions are concerned.

This file should normally be committed to Git.

---

# 52. `alembic.ini`

Configuration file for Alembic.

Defines things such as:

```text
migration location
Alembic configuration
logging
```

Database credentials should not be hardcoded into this file.

---

# 53. `README.md`

The main project documentation.

It answers:

```text
What is the project?
What problem does it solve?
How do I install it?
How do I run it?
How do I test it?
How is it deployed?
What is the current roadmap?
```

The main README should remain focused on **using and understanding the project**.

Detailed internal architecture belongs in the `docs/` directory.

---

# 54. `LICENSE`

Defines the legal terms under which the source code can be used, modified, or distributed.

The appropriate license depends on how the project will be owned and distributed.

---

# 55. Architectural Dependency Direction

A useful rule for this project is to keep dependencies flowing inward.

```text
                 API
                  │
                  ▼
               Service
              /       \
             ▼         ▼
       Repository   Optimization
             │          │
             ▼          ▼
           MySQL      Domain
```

More specifically:

```text
Endpoint
   │
   ▼
Service
   │
   ├──────────────► Repository
   │
   └──────────────► Optimization Engine
```

Avoid this:

```text
Repository
    ↓
FastAPI endpoint
```

or:

```text
Optimization Algorithm
    ↓
FastAPI Request
```

The optimization engine should ideally be usable without running FastAPI.

---

# 56. Data Flow Example

Consider:

```text
POST /api/v1/optimization/jobs
```

The intended flow is:

```text
Client
  │
  ▼
optimization.py
  │
  ▼
OptimizationService
  │
  ├── OrderRepository
  │       │
  │       ▼
  │      MySQL
  │
  ├── VehicleRepository
  │       │
  │       ▼
  │      MySQL
  │
  ▼
OptimizationProblem
  │
  ▼
OptimizationEngine
  │
  ▼
Genetic Algorithm
  │
  ▼
OptimizationSolution
  │
  ▼
OptimizationRepository
  │
  ▼
MySQL
```

The API layer therefore remains relatively thin.

---

# 57. Development Philosophy

The project follows several architectural principles.

## Separation of Concerns

Each layer should have a clear responsibility.

```text
API
    HTTP

Service
    Business logic

Repository
    Database access

Model
    Database structure

Schema
    API data contract

Optimization
    Optimization logic

Worker
    Background execution
```

---

## Testability

The optimization engine should be testable without FastAPI.

For example:

```text
OptimizationProblem
        │
        ▼
GeneticAlgorithm
        │
        ▼
OptimizationSolution
```

should be testable independently.

---

## Configuration

Configuration should come from environment variables rather than being hardcoded.

```text
.env
production environment
CI/CD secrets
AWS configuration
```

should provide configuration to the application.

---

## Reproducibility

The same commands should work across:

```text
Developer machine
CI
Docker
Production
```

where practical.

Docker, `uv.lock`, Makefile, and automated tests help achieve this.

---

# 58. Planned Production Architecture

Initial production architecture:

```text
                   Internet
                       │
                       ▼
                 ┌───────────┐
                 │   Nginx   │
                 └─────┬─────┘
                       │
                       ▼
                 ┌───────────┐
                 │  FastAPI  │
                 └─────┬─────┘
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
         Optimization       Repository
            Worker              │
              │                 │
              └────────┬────────┘
                       │
                       ▼
                    MySQL
```

A later production architecture may use AWS RDS:

```text
                   Internet
                       │
                       ▼
                 ┌───────────┐
                 │   Nginx   │
                 └─────┬─────┘
                       │
                       ▼
                 ┌───────────┐
                 │    EC2    │
                 │           │
                 │ FastAPI   │
                 │ Worker    │
                 └─────┬─────┘
                       │
                       ▼
                  ┌─────────┐
                  │ RDS     │
                  │ MySQL   │
                  └─────────┘
```

---

# 59. Important Rule

When adding a new feature, ask:

> "Which layer does this responsibility belong to?"

For example, adding vehicle creation:

```text
HTTP request
     │
     ▼
api/v1/endpoints/vehicles.py
     │
     ▼
services/vehicle.py
     │
     ▼
repositories/vehicle.py
     │
     ▼
models/vehicle.py
     │
     ▼
MySQL
```

If the feature involves optimization:

```text
services/optimization.py
          │
          ▼
optimization/domain/
          │
          ▼
optimization/algorithms/
          │
          ▼
optimization/solution
```

This keeps the project understandable as the codebase grows.

---

# 60. Summary

The repository can be understood as seven major areas:

```text
┌───────────────────────────────────────────┐
│                 Application               │
│                                           │
│  API → Services → Repositories → MySQL    │
│          │                                │
│          └──────→ Optimization Engine     │
└───────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│                  Testing                  │
│       Unit / Integration / API            │
└───────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│               Optimization                │
│ Domain / Algorithms / Operators / Goals   │
└───────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│                Operations                 │
│       Bash / Makefile / Docker            │
└───────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│                 Delivery                  │
│          GitHub Actions / AWS             │
└───────────────────────────────────────────┘
```

The central architectural idea is:

```text
                    FastAPI
                       │
                       ▼
                    Service
                   /       \
                  ▼         ▼
           Repository    Optimization
                │            │
                ▼            ▼
              MySQL       Algorithms
```

Each layer should have one primary responsibility, making the system easier to test, maintain, deploy, and extend.

# Logistics Optimizer

A production-oriented logistics optimization platform built with **FastAPI, MySQL, JWT authentication, Docker, and metaheuristic optimization algorithms**.

The system is designed to solve logistics and vehicle-routing optimization problems while providing a REST API for managing users, vehicles, orders, routes, and optimization jobs.

> **Status:** 🚧 Under active development

---

## Table of Contents

* [Overview](#overview)
* [Goals](#goals)
* [Key Features](#key-features)
* [Technology Stack](#technology-stack)
* [System Architecture](#system-architecture)
* [Project Structure](#project-structure)
* [Optimization Engine](#optimization-engine)
* [Authentication](#authentication)
* [Database](#database)
* [Local Development](#local-development)
* [Docker](#docker)
* [Environment Variables](#environment-variables)
* [Testing](#testing)
* [Code Quality](#code-quality)
* [API Documentation](#api-documentation)
* [CI/CD](#cicd)
* [AWS Deployment](#aws-deployment)
* [Configuration](#configuration)
* [Development Workflow](#development-workflow)
* [Future Improvements](#future-improvements)
* [License](#license)

---

# Overview

**Logistics Optimizer** is a backend platform for solving logistics optimization problems using mathematical optimization and metaheuristic algorithms.

The platform exposes a REST API through FastAPI and provides functionality for:

* User authentication and authorization
* Vehicle management
* Order management
* Route management
* Logistics constraints
* Optimization job creation
* Optimization algorithm execution
* Optimization result storage
* Asynchronous optimization jobs
* Database persistence
* Containerized deployment

The project is designed to run in two primary environments:

```text
Local Development
    Ubuntu
       ↓
    Docker
       ↓
FastAPI + MySQL
```

and:

```text
Production
    AWS EC2
       ↓
    Docker
       ↓
Nginx + FastAPI + Worker
       ↓
    MySQL / RDS
```

---

# Goals

The main goals of this project are:

1. Build a maintainable logistics optimization backend.
2. Separate API, business logic, persistence, and optimization logic.
3. Provide a reusable optimization engine.
4. Support multiple metaheuristic algorithms.
5. Provide secure JWT-based authentication.
6. Make the application reproducible through Docker.
7. Automate testing and deployment through GitHub Actions.
8. Make the system suitable for future production deployment.
9. Make optimization algorithms independently testable and benchmarkable.

---

# Key Features

## API

* REST API
* API versioning
* OpenAPI documentation
* Request validation
* Response schemas
* Centralized exception handling
* Health-check endpoints

## Authentication

* User registration
* User login
* JWT access tokens
* Password hashing
* Protected API endpoints
* Role-based authorization

## Logistics

* Vehicle management
* Order management
* Route management
* Delivery constraints
* Vehicle capacity constraints
* Time-window constraints
* Cost calculations
* Distance calculations

## Optimization

Planned/implemented algorithms include:

* Genetic Algorithm
* Simulated Annealing
* Tabu Search
* Ant Colony Optimization

The optimization layer is designed to remain independent from FastAPI so that algorithms can be tested and benchmarked separately.

## Infrastructure

* Docker
* Docker Compose
* MySQL
* Nginx
* GitHub Actions
* AWS EC2
* Container registry
* Database migrations with Alembic

---

# Technology Stack

| Component           | Technology      |
| ------------------- | --------------- |
| Language            | Python          |
| API                 | FastAPI         |
| Validation          | Pydantic        |
| ORM                 | SQLAlchemy      |
| Database            | MySQL           |
| Database Migration  | Alembic         |
| Authentication      | JWT             |
| Password Hashing    | Argon2 / bcrypt |
| Testing             | Pytest          |
| Linting             | Ruff            |
| Type Checking       | MyPy            |
| Package Management  | uv              |
| Containers          | Docker          |
| Local Orchestration | Docker Compose  |
| Reverse Proxy       | Nginx           |
| CI/CD               | GitHub Actions  |
| Cloud               | AWS EC2         |
| Container Registry  | AWS ECR         |

> Some technologies above may change during development.

---

# System Architecture

High-level architecture:

```text
                         ┌──────────────────┐
                         │      Client      │
                         │ Web / Mobile/API │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │      Nginx       │
                         │ Reverse Proxy    │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │     FastAPI      │
                         │      REST API    │
                         └────────┬─────────┘
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
             ▼                    ▼                    ▼
       ┌───────────┐       ┌─────────────┐      ┌─────────────┐
       │ Services  │       │ Optimization│      │Repositories │
       └─────┬─────┘       │   Engine    │      └──────┬──────┘
             │             └──────┬──────┘             │
             │                    │                    │
             │             ┌──────┴──────┐             │
             │             │             │             │
             │             ▼             ▼             │
             │            GA            SA             │
             │             │             │             │
             │             └──────┬──────┘             │
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │      MySQL       │
                         └──────────────────┘
```

For long-running optimization jobs, the architecture can be extended with workers:

```text
Client
  │
  ▼
FastAPI
  │
  │ create optimization job
  ▼
Job Queue
  │
  ▼
Optimization Worker
  │
  ▼
Metaheuristic Engine
  │
  ▼
MySQL
```

---

# Project Structure

```text
logistics-optimizer/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
│
├── app/
│   ├── main.py
│   │
│   ├── api/
│   │   └── v1/
│   │       ├── router.py
│   │       └── endpoints/
│   │
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   ├── logging.py
│   │   └── exceptions.py
│   │
│   ├── db/
│   │   ├── base.py
│   │   └── session.py
│   │
│   ├── models/
│   │
│   ├── schemas/
│   │
│   ├── repositories/
│   │
│   ├── services/
│   │
│   ├── optimization/
│   │   ├── domain/
│   │   ├── algorithms/
│   │   ├── operators/
│   │   ├── objectives/
│   │   └── engine.py
│   │
│   └── workers/
│
├── migrations/
│   └── versions/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── api/
│   ├── factories/
│   └── conftest.py
│
├── scripts/
│   ├── dev.sh
│   ├── test.sh
│   ├── lint.sh
│   ├── migrate.sh
│   ├── deploy.sh
│   ├── healthcheck.sh
│   └── backup.sh
│
├── docker/
│   ├── Dockerfile
│   ├── Dockerfile.prod
│   ├── entrypoint.sh
│   └── nginx/
│       └── nginx.conf
│
├── compose/
│   ├── docker-compose.yml
│   ├── docker-compose.dev.yml
│   └── docker-compose.prod.yml
│
├── .env.example
├── .gitignore
├── .dockerignore
├── Makefile
├── pyproject.toml
├── uv.lock
├── alembic.ini
├── README.md
└── LICENSE
```

---

# Optimization Engine

The optimization engine is one of the core components of the system.

It is intentionally separated from the FastAPI application layer.

```text
app/optimization/
│
├── domain/
│   ├── problem.py
│   ├── solution.py
│   └── constraints.py
│
├── algorithms/
│   ├── genetic_algorithm.py
│   ├── simulated_annealing.py
│   ├── tabu_search.py
│   └── ant_colony.py
│
├── operators/
│   ├── mutation.py
│   ├── crossover.py
│   └── selection.py
│
├── objectives/
│   ├── distance.py
│   ├── cost.py
│   ├── time.py
│   └── penalties.py
│
└── engine.py
```

The engine should receive a logistics problem and return an optimization solution.

Conceptually:

```text
Logistics Problem
       │
       ▼
Optimization Engine
       │
       ├── Algorithm
       │
       ├── Constraints
       │
       ├── Operators
       │
       └── Objective Functions
       │
       ▼
Optimization Solution
```

Detailed algorithm documentation will be added here as the project develops.

---

# Authentication

Authentication is implemented using JWT.

Typical authentication flow:

```text
POST /api/v1/auth/register
              │
              ▼
           User
              │
              ▼
POST /api/v1/auth/login
              │
              ▼
         JWT Token
              │
              ▼
Authorization: Bearer <token>
              │
              ▼
        Protected API
```

Security-related functionality is located primarily under:

```text
app/core/security.py
app/services/auth.py
app/api/deps.py
```

Authentication details will be documented as the API stabilizes.

---

# Database

The application uses MySQL as its relational database.

SQLAlchemy is used as the ORM layer and Alembic is used for schema migrations.

Database architecture:

```text
FastAPI
   │
   ▼
Service
   │
   ▼
Repository
   │
   ▼
SQLAlchemy
   │
   ▼
MySQL
```

Database migrations are stored in:

```text
migrations/
└── versions/
```

Run migrations with:

```bash
make migrate
```

Create a new migration with:

```bash
make migration MESSAGE="add optimization jobs"
```

> Exact commands may change during implementation.

---

# Local Development

## Requirements

Development is currently targeted at Ubuntu.

Required tools:

* Git
* Python
* Docker
* Docker Compose
* Make
* uv

Check the installations:

```bash
git --version
python3 --version
docker --version
docker compose version
make --version
uv --version
```

---

# Clone the Repository

```bash
git clone <repository-url>

cd logistics-optimizer
```

---

# Environment Configuration

Copy the example environment file:

```bash
cp .env.example .env
```

Edit the configuration:

```bash
nano .env
```

Never commit the real `.env` file.

---

# Run Locally

The recommended development workflow uses Docker Compose.

Start the development environment:

```bash
make up
```

Check running containers:

```bash
docker compose ps
```

View application logs:

```bash
make logs
```

Stop the environment:

```bash
make down
```

---

# Database Migrations

Run:

```bash
make migrate
```

Create a new migration:

```bash
make migration MESSAGE="describe your change"
```

Example:

```bash
make migration MESSAGE="create optimization jobs table"
```

---

# Testing

Run all tests:

```bash
make test
```

Unit tests:

```bash
make test-unit
```

Integration tests:

```bash
make test-integration
```

API tests:

```bash
make test-api
```

The test suite will be expanded as the system grows.

---

# Code Quality

Run linting:

```bash
make lint
```

Format the code:

```bash
make format
```

Run type checking:

```bash
make typecheck
```

The CI pipeline will execute these checks automatically.

---

# API Documentation

When the application is running locally:

```text
Swagger UI:
http://localhost:8000/docs

ReDoc:
http://localhost:8000/redoc

OpenAPI:
http://localhost:8000/openapi.json
```

> These URLs may change depending on the deployment configuration.

---

# Health Checks

The application provides health endpoints for monitoring and deployment.

Example:

```text
GET /health
GET /health/live
GET /health/ready
```

Example response:

```json
{
  "status": "ok"
}
```

The exact health-check contract will be documented when implemented.

---

# Docker

The project supports Docker-based development and deployment.

Development:

```bash
make up
```

Production build:

```bash
make build
```

Production environment:

```bash
docker compose \
  -f compose/docker-compose.prod.yml \
  up -d
```

Docker configuration is located under:

```text
docker/
compose/
```

---

# CI/CD

GitHub Actions is used for continuous integration and deployment.

## Continuous Integration

The CI workflow is located at:

```text
.github/workflows/ci.yml
```

The pipeline is expected to perform:

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

---

## Continuous Deployment

The CD workflow is located at:

```text
.github/workflows/cd.yml
```

The intended deployment flow is:

```text
main branch
     │
     ▼
GitHub Actions
     │
     ▼
Build Docker image
     │
     ▼
Push image to AWS ECR
     │
     ▼
Deploy to AWS EC2
     │
     ▼
Run migrations
     │
     ▼
Health check
     │
     ▼
Deployment complete
```

Deployment configuration will be documented once the AWS infrastructure is implemented.

---

# AWS Deployment

The production environment is planned around AWS EC2.

Initial architecture:

```text
Internet
   │
   ▼
AWS EC2
   │
   ├── Nginx
   │
   ├── FastAPI
   │
   └── Optimization Worker
   │
   ▼
MySQL
```

A future production architecture may use:

```text
Internet
   │
   ▼
AWS
   │
   ├── EC2
   │    ├── Nginx
   │    ├── FastAPI
   │    └── Worker
   │
   └── RDS MySQL
```

AWS-specific deployment documentation will be added when the infrastructure is configured.

---

# Environment Variables

Example configuration:

```env
APP_ENV=development

DATABASE_URL=mysql+pymysql://user:password@mysql:3306/logistics

JWT_SECRET_KEY=change-me
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=30

MYSQL_DATABASE=logistics
MYSQL_USER=logistics
MYSQL_PASSWORD=change-me
MYSQL_ROOT_PASSWORD=change-me
```

> Never use these example values in production.

Production secrets must not be committed to Git.

---

# Make Commands

The project provides a Makefile as the primary developer interface.

| Command                 | Description                  |
| ----------------------- | ---------------------------- |
| `make up`               | Start development containers |
| `make down`             | Stop containers              |
| `make restart`          | Restart containers           |
| `make logs`             | Show application logs        |
| `make test`             | Run all tests                |
| `make test-unit`        | Run unit tests               |
| `make test-integration` | Run integration tests        |
| `make lint`             | Run linting                  |
| `make format`           | Format source code           |
| `make typecheck`        | Run type checking            |
| `make migrate`          | Apply database migrations    |
| `make migration`        | Create migration             |
| `make shell`            | Open application shell       |
| `make db-shell`         | Open MySQL shell             |
| `make build`            | Build Docker image           |
| `make deploy`           | Deploy application           |

> The Makefile is still under development and commands may change.

---

# Development Workflow

Recommended development workflow:

```text
Create branch
     │
     ▼
Implement feature
     │
     ▼
Write tests
     │
     ▼
Run local checks
     │
     ├── make test
     ├── make lint
     └── make typecheck
     │
     ▼
Commit
     │
     ▼
Push branch
     │
     ▼
Open Pull Request
     │
     ▼
GitHub Actions CI
     │
     ▼
Code review
     │
     ▼
Merge to main
     │
     ▼
CD deployment
```

Example:

```bash
git checkout -b feature/optimization-job

# implement changes

make test
make lint
make typecheck

git add .
git commit -m "feat: add optimization job"

git push origin feature/optimization-job
```

---

# Branching Strategy

The project currently follows a feature-branch workflow.

Example branches:

```text
main
│
├── feature/jwt-auth
├── feature/vehicle-management
├── feature/order-management
├── feature/genetic-algorithm
├── feature/optimization-jobs
└── fix/route-validation
```

Production deployment is triggered from the configured production branch.

---

# Logging and Monitoring

Application logging is centralized through:

```text
app/core/logging.py
```

Future monitoring may include:

* Application logs
* Container logs
* Health checks
* Error tracking
* Metrics
* Optimization execution time
* Algorithm convergence metrics
* Job success/failure rates

Monitoring infrastructure will be documented as it is introduced.

---

# Optimization Metrics

The optimization system is expected to track metrics such as:

```text
Objective value
Total distance
Total cost
Travel time
Number of vehicles
Constraint violations
Execution time
Iterations
Convergence
```

Example future optimization result:

```json
{
  "status": "completed",
  "algorithm": "genetic_algorithm",
  "objective_value": 1234.56,
  "total_distance": 850.2,
  "total_cost": 1234.56,
  "vehicles_used": 12,
  "execution_time_seconds": 18.42
}
```

The exact result schema will be defined during implementation.

---

# Performance and Benchmarking

Optimization algorithms should be benchmarked independently from the API layer.

Future benchmarks will compare:

* Execution time
* Objective value
* Solution quality
* Convergence
* Memory usage
* Number of iterations
* Constraint violations

Benchmark datasets will be documented separately.

---

# Security Considerations

Security requirements include:

* Password hashing
* JWT authentication
* Authorization
* Environment-based secrets
* Input validation
* SQL injection protection through ORM/query parameterization
* Secure HTTP configuration
* Container security
* Dependency vulnerability scanning
* Production secret management

Secrets must never be committed to the repository.

---

# Backups

Database backup functionality will be provided through:

```text
scripts/backup.sh
```

Production backup strategy will depend on the final database architecture.

If AWS RDS is used, managed database backup functionality will be preferred where appropriate.

---

# Roadmap

## Phase 1 — Foundation

* [ ] Project structure
* [ ] FastAPI application
* [ ] Configuration system
* [ ] Docker development environment
* [ ] MySQL
* [ ] SQLAlchemy
* [ ] Alembic

## Phase 2 — Authentication

* [ ] User model
* [ ] Registration
* [ ] Login
* [ ] Password hashing
* [ ] JWT authentication
* [ ] Authorization
* [ ] Roles/permissions

## Phase 3 — Logistics Domain

* [ ] Vehicles
* [ ] Orders
* [ ] Locations
* [ ] Routes
* [ ] Constraints
* [ ] Cost calculation

## Phase 4 — Optimization

* [ ] Optimization domain model
* [ ] Objective functions
* [ ] Constraints
* [ ] Genetic Algorithm
* [ ] Simulated Annealing
* [ ] Tabu Search
* [ ] Ant Colony Optimization
* [ ] Optimization benchmarking

## Phase 5 — Async Processing

* [ ] Optimization jobs
* [ ] Job status
* [ ] Worker architecture
* [ ] Queue
* [ ] Job persistence
* [ ] Retry handling

## Phase 6 — Testing

* [ ] Unit tests
* [ ] Integration tests
* [ ] API tests
* [ ] Optimization tests
* [ ] Performance tests
* [ ] Test coverage

## Phase 7 — CI/CD

* [ ] GitHub Actions CI
* [ ] Docker image build
* [ ] Container registry
* [ ] GitHub Actions CD
* [ ] EC2 deployment
* [ ] Database migration during deployment
* [ ] Deployment health check

## Phase 8 — Production

* [ ] HTTPS
* [ ] Domain
* [ ] Production monitoring
* [ ] Centralized logging
* [ ] Database backups
* [ ] Security hardening
* [ ] Performance optimization

---

# Contributing

Development guidelines will be added as the project evolves.

Before creating a pull request, run:

```bash
make test
make lint
make typecheck
```

All new functionality should include appropriate tests.

---

# License

This project is licensed under the terms described in [`LICENSE`](LICENSE).

---

# Author

**Your Name**

* GitHub: `<your-github-profile>`
* Email: `<your-email>`
* LinkedIn: `<your-linkedin-profile>`

---

# Project Status

🚧 **Active Development**

This README describes the planned architecture and may change as implementation progresses.

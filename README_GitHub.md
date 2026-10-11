# Resource Sharing System

A resource-sharing web application built with **Node.js, Express, Neo4j, and Docker**. The project reimplements a relational SQL/PL-SQL resource-sharing workflow using a **NoSQL graph database**, a REST API, and a lightweight web interface.

It supports resource discovery, borrowing and returning physical resources, fine calculation, ratings, favorites, and report management. Neo4j stores the entities and their relationships, while the API enforces validation and transaction rules.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Application URLs](#application-urls)
- [API Reference](#api-reference)
- [Data Model](#data-model)
- [Demo Walkthrough](#demo-walkthrough)
- [Cypher Queries and Project Documentation](#cypher-queries-and-project-documentation)
- [Troubleshooting](#troubleshooting)
- [Design Notes](#design-notes)

## Overview

The system manages resources shared among users, including resource ownership, subjects, departments, borrowing and returns, ratings, favorites, and reports.

Neo4j was selected because the application's data is highly connected. Graph relationships represent associations that would otherwise be expressed through foreign keys and joins in a relational database. Neo4j transactions help keep multi-step operations such as borrowing and returning consistent.

> **Database:** Neo4j (NoSQL graph database)  
> **Backend:** Node.js + Express REST API  
> **Frontend:** HTML, CSS, and JavaScript  
> **Deployment:** Docker Compose

## Features

- **Resource catalogue:** Browse and filter resources by type, status, subject, or search query.
- **Resource management:** Add resources and associate them with an owner and subject.
- **Borrowing workflow:** Borrow available physical resources with a seven-day due period.
- **Returns and fines:** Record returns, resource condition, and late-return fines at ₹10 per late day.
- **Ratings:** Submit ratings from 1 to 5 and retrieve average ratings.
- **Favorites:** Add, list, and remove favorite resources.
- **Reports:** Submit reports and update their status.
- **Statistics:** View graph node and relationship counts.
- **Demo data:** Load sample data for testing and demonstrations.
- **Graph inspection:** Explore the stored data directly in Neo4j Browser.
- **Containerized setup:** Run the API and Neo4j using Docker Compose.

## Technology Stack

| Component | Technology |
|---|---|
| Runtime | Node.js |
| Backend framework | Express |
| Database | Neo4j |
| Database query language | Cypher |
| Neo4j integration | `neo4j-driver` |
| Frontend | HTML, CSS, JavaScript |
| Containerization | Docker and Docker Compose |

## Architecture

```text
Browser
   |
   v
Web frontend (served by Express)
   |
   v
REST API / Business logic
   |
   v
Neo4j Driver
   |
   v
Neo4j Graph Database
```

The backend handles request validation, database operations, and transaction-sensitive business rules. Neo4j stores the application entities as nodes and their connections as relationships.

## Project Structure

```text
resource-sharing-system/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── server.js
│   │   ├── db.js
│   │   ├── seed.js
│   │   └── util.js
│   └── public/
│       └── index.html
├── cypher/
│   └── resource_sharing.cypher
└── scripts/
    └── smoke-test.sh
```

- `backend/src/server.js` — REST endpoints, validation, and business logic.
- `backend/src/db.js` — Neo4j driver, database helpers, and constraints.
- `backend/src/seed.js` — sample data setup.
- `backend/src/util.js` — shared utilities, including HTTP errors and password hashing.
- `backend/public/index.html` — frontend served by the backend.
- `cypher/resource_sharing.cypher` — Cypher statements corresponding to the project’s database exercises.
- `scripts/smoke-test.sh` — shell-based end-to-end API smoke test.

## Prerequisites

Install the following before starting:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) with Docker Compose enabled.
- A web browser.

For running the API outside Docker, install [Node.js](https://nodejs.org/) and npm as well.

## Getting Started

### 1. Clone the repository

```bash
git clone <repository-url>
cd resource-sharing-system
```

Replace `<repository-url>` with the URL of your GitHub repository. If you have already downloaded the project, open a terminal in the project root instead.

### 2. Start the application

```bash
docker compose up --build
```

The first startup may take around a minute while Neo4j initializes. Wait for the backend log indicating that the Resource Sharing System is running.

### 3. Open the application

Visit the frontend URL listed in [Application URLs](#application-urls).

### 4. Stop the application

Press `Ctrl+C` in the terminal, then run:

```bash
docker compose down
```

To remove the Docker volumes as well as the containers (this deletes the persisted Neo4j data for this setup), run:

```bash
docker compose down -v
```

**Warning:** Use `docker compose down -v` only when you intend to reset the database and remove its persisted data.

### Run the API without containerizing it

To run only Neo4j in Docker and the Node.js API on your machine:

```bash
docker compose up neo4j
```

In a second terminal:

```bash
cd backend
npm install
npm start
```

## Application URLs

When the default ports are available:

| URL | Purpose |
|---|---|
| `http://localhost:3000` | Web application |
| `http://localhost:3000/api/health` | API health check |
| `http://localhost:7474` | Neo4j Browser |
| `bolt://localhost:7687` | Neo4j Bolt connection endpoint |

The supplied Docker configuration uses the Neo4j credentials `neo4j` / `resource123` for local development. Use these only for the configured demo environment; change credentials before any non-local deployment.

## API Reference

The API is served from `http://localhost:3000`. The following endpoints are available in the project.

| Method | Endpoint | Description |
|---|---|---|
| `GET` / `POST` | `/api/departments` | List or create departments |
| `GET` / `POST` | `/api/subjects` | List subjects (including department information) or create a subject |
| `GET` / `POST` | `/api/users` | List users or register a user |
| `GET` | `/api/resources` | List and filter resources |
| `POST` | `/api/resources` | Create a resource |
| `POST` | `/api/resources/:id/download` | Increment the download count for a digital resource |
| `POST` | `/api/borrow` | Borrow a physical resource |
| `POST` | `/api/return` | Return a borrowed resource and calculate any fine |
| `GET` | `/api/borrows` | List borrowing records |
| `GET` / `POST` | `/api/ratings` | List ratings or submit a rating |
| `GET` | `/api/ratings/average` | Retrieve average ratings by resource |
| `GET` / `POST` | `/api/favorites` | List or add favorites |
| `DELETE` | `/api/favorites/:id` | Remove a favorite |
| `GET` / `POST` | `/api/reports` | List reports or submit a report |
| `PATCH` | `/api/reports/:id` | Update a report status |
| `GET` | `/api/stats` | Retrieve node and relationship counts |
| `POST` | `/api/reset` | Reload the sample data |

### Resource filters

`GET /api/resources` supports the following query parameters:

- `type` — resource type.
- `status` — resource status.
- `subject_id` — filter by subject.
- `q` — search query.

### Example: Borrow a resource

```bash
curl -X POST http://localhost:3000/api/borrow \
  -H "Content-Type: application/json" \
  -d '{"user_id":2,"resource_id":1003}'
```

### Example: Return a resource

```bash
curl -X POST http://localhost:3000/api/return \
  -H "Content-Type: application/json" \
  -d '{"borrow_id":1,"resource_condition":"good"}'
```

Use IDs that exist in your current database. The exact request fields and validation responses are implemented in `backend/src/server.js`.

## Data Model

The application represents its core entities as graph nodes and their connections as relationships.

```text
(User)-[:MEMBER_OF]->(Department)
(Subject)-[:IN_DEPARTMENT]->(Department)
(Resource)-[:BELONGS_TO]->(Subject)
(User)-[:OWNS]->(Resource)

(User)-[:BORROWED]->(Borrow)-[:FOR_RESOURCE]->(Resource)
(Borrow)-[:HAS_RETURN]->(ReturnLog)

(User)-[:RATED {rating_id, rating_value, review, rating_date}]->(Resource)
(User)-[:FAVORITED {favorite_id, added_date}]->(Resource)

(User)-[:REPORTED]->(Report)-[:ABOUT]->(Resource)
```

| Relational concept | Neo4j representation |
|---|---|
| Department, subject, user, resource | Nodes with corresponding labels |
| Borrow and return log | Nodes with their own identifiers and lifecycle data |
| Rating and favorite | Relationships with properties linking a user and a resource |
| Report | Report node connected to the reporting user and reported resource |
| Primary keys and unique values | Neo4j uniqueness constraints |
| Foreign keys | Relationships between existing nodes |
| SQL joins | Cypher `MATCH` patterns |
| Stored procedures and transaction logic | REST API operations and Neo4j transactions |

## Demo Walkthrough

The following sequence demonstrates the main application workflows:

1. Open the frontend and review the dashboard statistics and average ratings.
2. Open **Resources**, filter for physical and available resources, and borrow an available item.
3. Try borrowing the same item as another user to verify that unavailable resources cannot be borrowed.
4. Attempt to borrow a digital resource and verify that the API rejects the operation.
5. Open **Borrow / Return**, return a resource with a date beyond its due date, and review the calculated fine.
6. Submit a valid rating, then try an out-of-range rating or a duplicate rating for the same user and resource.
7. Add and remove a favorite, then resolve or reject a report.
8. Open Neo4j Browser and inspect the graph.

For a graph overview, run this Cypher query in Neo4j Browser:

```cypher
MATCH p=(n)-[]->(m)
WHERE NOT n:Counter AND NOT m:Counter
RETURN p
```

## Cypher Queries and Project Documentation

The `cypher/resource_sharing.cypher` file organizes Cypher statements to correspond to the original project exercises. Use Neo4j Browser to run the statements and capture query results when preparing project documentation.

The file’s initial reset section clears existing data. Review the statements before running them, especially if you have added data that you want to keep.

For workflows originally represented by PL/SQL procedures, demonstrate both the relevant Cypher/database state and the corresponding REST API request or frontend action.

## Troubleshooting

| Issue | Suggested fix |
|---|---|
| Ports `3000`, `7474`, or `7687` are already in use | Stop the conflicting process or change the host-side port mapping in `docker-compose.yml`. |
| The API reports that it is waiting for Neo4j | Allow time for Neo4j to start; the backend retries during startup. |
| Neo4j authentication fails after changing credentials | Neo4j credentials may persist in the existing volume. Check the configured password and volume state. |
| Backend changes are not reflected | Rebuild and restart with `docker compose up --build`. |
| You need to reload sample data | Use the application's reset option or `POST /api/reset`. |
| You need a completely fresh local database | Run `docker compose down -v`, understanding that this deletes persisted volume data. |

### Run the smoke test

From the project root, run:

```bash
bash scripts/smoke-test.sh
```

This script requires a shell with `bash` and `curl` available. On Windows, you can use Git Bash or WSL.

## Design Notes

- **Graph-oriented data model:** Relationships are stored explicitly rather than reconstructed through relational joins.
- **Transaction safety:** Borrowing and returning use database transactions so multi-step changes can succeed or roll back together.
- **Validation in the API:** Input constraints and business rules are checked in the service layer where Neo4j Community Edition does not provide equivalent SQL-style constraints.
- **Concurrency handling:** The borrowing flow uses a write to the resource node to help serialize competing borrow requests.
- **Password handling:** User passwords are stored as salted scrypt hashes rather than plaintext.
- **Trigger behavior:** Neo4j Community Edition does not provide native database triggers; relevant checks are implemented in the application logic.

---

This README describes the project’s configured local development setup. Review environment variables, credentials, and deployment settings before exposing the application outside your local machine.

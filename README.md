# postgres-db

PostgreSQL database for the Exodus Rush game. Primary relational database for persistent data.

## Overview

**Service Configuration:**
- **Port:** 5432
- **Replicas:** 1 (primary instance)
- **Access URL:** `postgres-db:5432`
- **Database:** exodus_rush
- **User:** exodus
- **Password:** passover2026 (for development environment)

## Databases

The PostgreSQL instance contains multiple schemas:

### users
User accounts and authentication data
- id, username, email, password_hash
- created_at, last_login

### characters
Character state and positions
- id, user_id, name, position_x, position_y
- state, has_crossed, timestamps

### events
Analytics events and gameplay metrics
- id, user_id, event_type, event_data (JSONB)
- timestamp (indexed for queries)

### game_state
General game world state
- id, key, value (JSONB), updated_at

## Features

- PostgreSQL 15 with UTF-8 encoding
- JSONB support for flexible data structures
- Indexes on frequently queried columns
- Health checks via pg_isready

## Deployment

```bash
# Create namespace
kubectl create namespace passover

# Deploy PostgreSQL
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Initialize database (run once)
kubectl exec -n passover deployment/postgres-db -- psql -U exodus -d exodus_rush -f /docker-entrypoint-initdb.d/init.sql

# Verify deployment
kubectl get pods -n passover -l app=postgres-db
kubectl get svc -n passover postgres-db
```

## Usage

Services connect to PostgreSQL at:

```
postgres://exodus:passover2026@postgres-db:5432/exodus_rush
```

Example connection strings:
- Node.js (pg): `postgresql://exodus:passover2026@postgres-db:5432/exodus_rush`
- Python (psycopg2): `host=postgres-db port=5432 dbname=exodus_rush user=exodus password=passover2026`
- Go (pgx): `postgres://exodus:passover2026@postgres-db:5432/exodus_rush`

## Health Check

```bash
# Test PostgreSQL connection
kubectl exec -n passover deployment/postgres-db -- pg_isready -U exodus

# Connect to psql
kubectl exec -it -n passover deployment/postgres-db -- psql -U exodus -d exodus_rush

# List tables
kubectl exec -n passover deployment/postgres-db -- psql -U exodus -d exodus_rush -c "\dt"
```

## Services Using PostgreSQL

- **auth-service:** User accounts and authentication
- **character-service:** Character positions and state
- **analytics-service:** Event logging and metrics
- (game_state table could be used, but sea state uses etcd for distributed consensus)


## Development

For local development, the init.sql script creates all necessary tables and a test user.

**Note:** In production, use proper secrets management. The hardcoded password is for development environment only.

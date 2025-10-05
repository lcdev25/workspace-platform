# workspace-platform
A Platform For Building Your Own Personal Workspace

Of course. Here is the project plan converted into a Markdown-friendly format, perfect for a PLAN.md or README.md file in your GitHub repository.

Project Plan: Workbench Nucleus
Project Goal: To create a fully containerized, multi-service application that demonstrates the end-to-end event-driven CQRS architecture. A user should be able to git clone the repository and run docker-compose up to have the entire system running on their local machine for development and demonstration.

Phase 0: Foundation & Setup (The Blueprint)
Estimated Time: ~1-2 Days

[ ] Task 0.1: Initialize Git Repository
Create a new repository on GitHub.

Initialize with a main branch.

Create a comprehensive .gitignore file for your chosen tech stack (e.g., Node.js, Java) and OS files.

[ ] Task 0.2: Choose Core Tech Stack
Backend Language: Java (Spring Boot) or Node.js (TypeScript with Fastify/NestJS).

Database: PostgreSQL.

Frontend: React (with Vite) or Vue.

[ ] Task 0.3: Create Master docker-compose.yml File
This file will define the entire application stack. Start with stubs for all services.

YAML

version: '3.8'
services:
  postgres:
    # ...
  zookeeper:
    # ...
  kafka:
    # ...
  debezium-connect:
    # ...
  elasticsearch:
    # ...
  command-api:
    # Your core transactional service
  query-view-updater:
    # Kafka consumer for Elasticsearch
  rule-engine-service:
    # Kafka consumer for rules
  query-api:
    # API for the frontend
  frontend:
    # The UI service
[ ] Task 0.4: Set Up a Monorepo Project Structure
Organize the repository to handle multiple services cleanly.

Bash

/workbench-nucleus
|-- docker/
|   |-- debezium/
|   |   |-- register-postgres.json # Debezium connector config
|-- services/
|   |-- command-api/
|   |-- query-view-updater/
|   |-- rule-engine-service/
|   |-- query-api/
|   |-- frontend/
|-- .gitignore
|-- docker-compose.yml
|-- README.md
Phase 1: The Core Transactional Service (The "Write Side")
Estimated Time: ~5-7 Days

[ ] Task 1.1: Design & Implement Database Schema
Use a migration tool (e.g., Flyway, Liquibase, Knex).

Create the core tables: entities, attributes, attribute_types, items.

Create the crucial outbox table.

outbox Table Schema:
| Column | Type | Constraints | Notes |
|---|---|---|---|
| id | uuid | PRIMARY KEY | Unique event ID |
| aggregate_type | varchar(255)| NOT NULL | e.g., "Item" |
| aggregate_id | varchar(255)| NOT NULL | The ID of the item that changed |
| event_type | varchar(255)| NOT NULL | e.g., "ItemCreated" |
| payload | jsonb | | The full event data |
| created_at | timestamp | DEFAULT NOW()| |

[ ] Task 1.2: Build the "Command API" Service
Scaffold the service in /services/command-api.

Implement Repository/Data Access layers.

Implement business logic ensuring that writes to items and outbox happen in a single database transaction.

[ ] Task 1.3: Implement the REST API
Create CRUD endpoints for /entities, /attributes, and /items.

Integrate OpenAPI/Swagger for automated API documentation.

[ ] Task 1.4: Containerize the Service
Write a Dockerfile for the Command API.

Add the service definition to docker-compose.yml and configure its connection to PostgreSQL.

Phase 2: The Eventing Pipeline (The "Plumbing")
Estimated Time: ~2-3 Days

[ ] Task 2.1: Configure the Debezium Connector
Create the JSON configuration in /docker/debezium/register-postgres.json.

This config will tell Debezium to:

Connect to the postgres container.

Watch the public.outbox table for changes.

Publish records to a Kafka topic named outbox.events.

Use Kafka Connect's outbox event router SMT to clean up the event payload.

[ ] Task 2.2: Test the Full Pipeline
Milestone: Verify that making a POST request to the Command API results in a corresponding event appearing on the outbox.events Kafka topic.

Use a CLI tool like kafkacat or the console consumer script to check the topic.

Phase 3: The Asynchronous Consumers (The "Workers")
Estimated Time: ~5-7 Days

[ ] Task 3.1: Build the "Query View Updater" Service
Scaffold the service in /services/query-view-updater.

Implement a Kafka consumer listening to outbox.events.

Implement an Elasticsearch client.

Core Logic: On receiving an event, transform its payload into the desired Elasticsearch document structure and upsert it into an items index.

Containerize the service and add to docker-compose.yml.

[ ] Task 3.2: Build the "Rule Engine" Service
Scaffold the service in /services/rule-engine-service.

Rule Storage:

Create a rules table in PostgreSQL to store rule definitions.

Fields: id, name, entity_name, trigger_on, conditions (jsonb), action (jsonb).

Rule Caching: Fetch and cache active rules in memory on service startup.

Processing Logic:

The Kafka consumer listens to outbox.events.

For each event, find matching cached rules based on entity and trigger type.

Use a JSON Logic library to evaluate the rule's conditions against the event payload.

If conditions pass, execute the defined action.

Implement Actions:

Start with simple actions like CONSOLE_LOG and WEBHOOK.

Containerize the service and add to docker-compose.yml.

Phase 4: The Query Service & Frontend (The "Read Side")
Estimated Time: ~4-5 Days

[ ] Task 4.1: Build the "Query API" Service
Scaffold a simple service in /services/query-api.

This service abstracts Elasticsearch from the frontend.

Expose a simple API (e.g., POST /items/search) that accepts a filter object.

Core Logic: Translate the simple filter object into the Elasticsearch Query DSL.

Containerize and add to docker-compose.yml.

[ ] Task 4.2: Build a Basic Frontend UI
Scaffold a simple SPA in /services/frontend.

Page 1: Schema Builder: A form to create Entities and Attributes (calls command-api).

Page 2: Data View: A configurable table that displays and filters items by calling the query-api.

Containerize (e.g., with a multi-stage Nginx Dockerfile) and add to docker-compose.yml.

Phase 5: Packaging & Documentation (The "Release")
Estimated Time: ~2-3 Days

[ ] Task 5.1: Write an Excellent README.md
Project Goal: What is this?

Architecture: Include a diagram and explanation of the flow.

Getting Started: Simple, foolproof instructions.

git clone https://github.com/your-repo/workbench-nucleus.git

cd workbench-nucleus

cp .env.example .env

docker-compose up -d --build

Usage Guide: A step-by-step tutorial to demonstrate the full end-to-end functionality.

[ ] Task 5.2: Finalize Configuration
Move all environment-specific variables (ports, passwords, etc.) into a .env file.

Create a .env.example file and commit it to the repository.

[ ] Task 5.3: Test the Full User Journey
Perform a "clean install": clone the repo into a new directory and follow the README instructions precisely.

Ensure the entire system comes up and is fully functional without any manual intervention. Refine scripts and documentation until this is seamless.

# Hospital Management System

DBMS mini project (UE23CS351A) by Anvith Vegi (PES1UG24AM047) and Anirudh N (PES1UG24AM041).

A web application for patients, doctors, appointments, prescriptions, admissions, billing and pharmacy stock. It uses **Next.js**, **PostgreSQL** and **Neo4j**. Access is controlled by four roles, enforced in the database itself.

## Setup

You need Docker Desktop and Node.js.

```bash
# 1. Start the databases. PostgreSQL loads database/postgres/*.sql automatically the first time.
docker compose up -d

# 2. Load the Neo4j graph (wait about 30 seconds after step 1 for Neo4j to start).
docker exec hms-neo4j cypher-shell -u neo4j -p hms_graph_dev -f /cypher/01_create_graph.cypher

# 3. Start the web app.
cd web
cp .env.example .env.local    # then put any long random string in JWT_SECRET
npm install
npm run dev
```

Open <http://localhost:3000>.

To go back to fresh sample data, run `docker compose down -v` and repeat steps 1 and 2.

## Logins

| Username | Password | Role |
|---|---|---|
| `admin` | `admin123` | Admin |
| `reception` | `reception123` | Receptionist |
| `pharmacy` | `pharmacy123` | Pharmacist |
| `dr.rao`, `dr.menon`, `dr.kulkarni`, `dr.nair`, `dr.reddy`, `dr.hegde` | `doctor123` | Doctor |

These are demo passwords, as is the sample data.

## A demo that touches everything

1. **reception**: register a patient, then book them an appointment with Dr. Reddy (only free slots are offered).
2. **dr.reddy**: write a prescription for that appointment. Choose Aspirin and Warfarin to see the warning from Neo4j. Then mark the appointment complete, which creates the bill.
3. **pharmacy**: dispense the prescription. Stock drops and the medicines are added to the bill.
4. **reception**: open the bill and record a payment. Try paying too much to see the trigger reject it.
5. **pharmacy**: on Drug Interactions, add, edit and delete an interaction (Neo4j).
6. **admin**: look at Reports.

To see the database enforcing roles, log in as **dr.reddy** and open `/billing`.

## Project layout

```
database/postgres/   Runs in order on first start
  01_schema.sql              Tables, keys, constraints, indexes
  02_functions_views.sql     Functions (including the recursive slot finder) and views
  03_triggers_procedures.sql Triggers and stored procedures
  04_roles.sql               Database roles and GRANTs
  05_seed.sql                Sample data
database/queries/    DML, advanced query and role demonstrations
database/neo4j/      Graph creation and read/update/delete commands
docs/                Report, diagrams (.drawio and .png), logs, screenshots
web/                 Next.js application
  app/               Pages, and app/api/ for the backend routes
  components/        Shared UI (table, form, sidebar)
  lib/               Database, Neo4j, login and API helpers
```

## Where each rubric item is

| Rubric item | Where |
|---|---|
| ER diagram, relational schema | `docs/diagrams/` |
| DDL with constraints | `database/postgres/01_schema.sql` |
| NoSQL database creation | `database/neo4j/01_create_graph.cypher` |
| Technology justification | Report, User Requirements Specification |
| DML | `database/queries/dml_operations.sql`, and the app's forms |
| Nested, correlated, recursive and join queries | `database/queries/advanced_queries.sql` |
| Procedures, functions, triggers, CTE | `database/postgres/02_*` and `03_*` |
| NoSQL read, update, delete | `database/neo4j/02_read_update_delete.cypher`, Drug Interactions page |
| Next.js interface and API | `web/app/` and `web/app/api/` |
| Authentication | `web/lib/auth.js`, `web/app/api/auth/` |
| Role-based access in the database | `database/postgres/04_roles.sql`, `withRole` in `web/lib/db.js`, `database/queries/rbac_demo.sql` |

## Re-running the demonstration scripts

```bash
docker exec hms-postgres psql -U postgres -d hospital -f /queries/dml_operations.sql
docker exec hms-postgres psql -U postgres -d hospital -f /queries/advanced_queries.sql
docker exec -e PGPASSWORD=hms_app_dev hms-postgres psql -h 127.0.0.1 -U hms_app -d hospital -f /queries/rbac_demo.sql
docker exec hms-neo4j cypher-shell -u neo4j -p hms_graph_dev -f /cypher/02_read_update_delete.cypher
```

The errors in `rbac_demo.sql` are expected: they show the database refusing access. The DML script rolls itself back.

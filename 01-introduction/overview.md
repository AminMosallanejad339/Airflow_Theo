# Overview

# Apache Airflow Learning Table of Contents

## 1. Introduction
- What is Apache Airflow?
- History and evolution
- Use cases (ETL, ML pipelines, data automation)

## 2. Airflow Architecture
- DAG (Directed Acyclic Graph)
- Task, Operator, Sensor, Hook
- Scheduler, Executor, Worker
- Metadata Database
- Web UI & CLI

## 3. Installation & Setup
- Installing via pip
- Initializing Airflow database
- Starting webserver and scheduler
- Directory structure
- Configuring airflow.cfg

## 4. Core Concepts
- DAGs and Task Dependencies
- Operators: PythonOperator, BashOperator, EmailOperator, DummyOperator, etc.
- Sensors
- Hooks
- XCom (for passing data between tasks)
- Variables & Connections
- Pools and Queues

## 5. DAG Design & Scheduling
- DAG structure best practices
- Default arguments
- Schedule intervals (CRON expressions, presets)
- Catchup vs No Catchup
- Task retries and SLA

## 6. Advanced DAG Features
- SubDAGs
- Task groups
- Branching with BranchPythonOperator
- Trigger rules (all_success, all_failed, one_success, etc.)
- Dynamic DAG generation
- Templates & Jinja macros

## 7. Executors & Scaling
- SequentialExecutor
- LocalExecutor
- CeleryExecutor
- KubernetesExecutor
- Choosing the right executor
- Scaling tips

## 8. Monitoring & Logging
- Airflow Web UI overview
- Task logs and debugging
- Email alerts and notifications
- SLA misses and retries

## 9. Integrations
- Connecting to databases (MySQL, PostgreSQL, BigQuery, etc.)
- Cloud integrations (AWS, GCP, Azure)
- APIs and external services
- ML workflows and ETL pipelines

## 10. Best Practices
- DAG versioning and deployment
- Modular DAG design
- Security (Roles, connections, secrets backend)
- Avoiding anti-patterns
- Testing DAGs locally

## 11. Airflow CLI
- Common commands (`airflow dags`, `airflow tasks`, `airflow users`)
- Triggering DAGs manually
- Backfilling
- Pausing and resuming DAGs

## 12. Advanced Topics
- Using XCom for inter-task communication
- Custom Operators, Sensors, Hooks
- Writing Plugins
- Airflow REST API
- Airflow with Docker & Kubernetes

## 13. Troubleshooting & Optimization
- Common errors & solutions
- Performance tuning
- Debugging DAG execution
- Database and scheduler optimizations

## 14. Real-World Use Cases
- Data warehouse ETL pipeline
- ML model training pipeline
- Event-driven automation
- Batch processing and reporting

## 15. Resources & Further Learning
- Official documentation
- Tutorials & YouTube courses
- Books & blogs
- Community forums & GitHub repositories

### **Table of Contents: A Complete Guide to Learning Apache Airflow**

#### **Part I: Foundations & Core Concepts**
1.  **Introduction to Workflow Orchestration**
    *   What is a workflow?
    *   The problem Airflow solves: Scheduling, Dependency Management, Monitoring.
    *   Airflow vs. other tools (e.g., Luigi, Prefect, Dagster).

2.  **Airflow Architecture Overview**
    *   Core Components: Scheduler, Web Server, Executor, Workers, Metadata Database.
    *   How the components interact.
    *   Understanding the role of the message queue (with Celery/Kubernetes Executors).

3.  **The Directed Acyclic Graph (DAG)**
    *   What is a DAG? Nodes and Edges.
    *   Defining a DAG in code: The `DAG` object.
    *   Key DAG parameters: `dag_id`, `schedule_interval`, `start_date`, `catchup`, `default_args`, `tags`.

4.  **Operators: The Building Blocks of Work**
    *   What is an Operator? The principle of idempotency.
    *   Common Core Operators:
        *   `BashOperator`: Executing shell commands.
        *   `PythonOperator`: Executing Python functions.
        *   `EmptyOperator`: For grouping and structuring.
    *   The concept of **Tasks** as instances of Operators.

5.  **Task Dependencies & Relationships**
    *   Defining order: `set_upstream()`, `set_downstream()`.
    *   The bitshift operators: `task1 >> task2` and `task2 << task1`.
    *   Building complex dependencies: branching, fan-in, fan-out.

---

#### **Part II: Development & Functionality**
6.  **Scheduling & Execution**
    *   How the Scheduler parses DAGs.
    *   Understanding execution dates and data intervals.
    *   The `catchup` parameter and backfilling.
    *   Manual triggering of DAGs.

7.  **Sharing Data Between Tasks: XComs**
    *   What is an XCom? (Cross-Communication)
    *   Pushing and pulling data with `xcom_push` and `xcom_pull`.
    *   Limitations and best practices for XComs (small data only).

8.  **Hooks and Connections**
    *   What is a Hook? Abstracting external system connections.
    *   Using the Airflow UI to manage Connections (passwords, logins).
    *   Example: Using `PostgresHook`, `S3Hook`.

9.  **The Airflow UI In-Depth**
    *   **DAGs View**: Enabling/pausing DAGs.
    *   **Tree View**: Monitoring task instances over runs.
    *   **Graph View**: Visualizing dependencies and debugging.
    *   **Task Duration**, **Gantt Chart**, and **Code** views for performance analysis.

10. **Sensors: Waiting for Conditions**
    *   What is a Sensor? Polling for external conditions.
    *   Common Sensors: `FileSensor`, `TimeSensor`.
    *   The `mode` parameter (`poke` vs. `reschedule`).
    *   Implementing timeouts with `timeout` and `soft_fail`.

---

#### **Part III: Intermediate Concepts & Best Practices**
11. **TaskFlow API (Airflow 2.0+)**
    *   Simplifying DAG authoring with the `@task` and `@dag` decorators.
    *   Automatic XCom handling and dependency inference.
    *   Passing data between decorated tasks.

12. **Dynamic DAG Generation**
    *   Generating DAGs and tasks using Python code and loops.
    *   Use cases: Managing similar workflows for multiple tables/clients.

13. **Error Handling & Robustness**
    *   Task Retries: `retries` and `retry_delay` parameters.
    *   Alerting: Configuring `email_on_retry` and `email_on_failure`.
    *   Using `on_failure_callback` for custom alerting (e.g., Slack).

14. **Project Structure & Best Practices**
    *   How to organize your `dags/` folder.
    *   Separating configuration, business logic, and DAG definitions.
    *   Writing idempotent and atomic tasks.
    *   Effective use of DAG and Task IDs.

---

#### **Part IV: Deployment & Operations**
15. **Installation & Setup**
    *   Local installation with `pip` and virtual environments.
    *   **Recommended Path**: Using the official `docker-compose.yaml` for local development.

16. **Executors: Scaling Airflow**
    *   **SequentialExecutor**: For development only.
    *   **LocalExecutor**: Parallelism on a single machine.
    *   **CeleryExecutor**: Distributed execution across multiple workers (classic scaling).
    *   **KubernetesExecutor**: Dynamic, container-based execution on a K8s cluster (modern scaling).

17. **Deployment Strategies**
    *   Deploying with Docker & Docker-Compose (for small teams).
    *   Deploying on Kubernetes with the Official Helm Chart (production standard).
    *   Managed Services: Google Cloud Composer, AWS MWAA, Astronomer.

18. **Monitoring & Logging**
    *   Where to find logs: UI vs. filesystem.
    *   Integrating with external monitoring (Prometheus/Grafana) via metrics.
    *   Setting up centralized logging (e.g., to S3 or Elasticsearch).

19. **Security & Configuration**
    *   Authentication (RBAC - Role-Based Access Control).
    *   Securing Connections and Variables.
    *   Key Airflow configuration tweaks in `airflow.cfg`.

---

#### **Part V: Advanced Topics & Ecosystem**
20. **Creating Custom Operators & Plugins**
    *   When and why to build a custom Operator.
    *   Extending Airflow with custom Hooks, Operators, and Views via Plugins.

21. **The Provider System**
    *   Understanding Airflow's modular provider packages.
    *   Installing providers for AWS, Google Cloud, Azure, Snowflake, etc.
    *   Exploring available Operators and Hooks from providers.

22. **Data-aware Scheduling & Datasets (Airflow 2.4+)**
    *   The modern alternative to complex Sensor setups.
    *   Defining `Datasets` and using them to trigger DAGs based on data updates.

23. **Dynamic Task Mapping (Airflow 2.3+)**
    *   The runtime equivalent of a `for-loop` for tasks.
    *   Mapping tasks over a list of inputs that is only known at runtime.

24. **Performance Tuning & Database Maintenance**
    *   Tuning the Scheduler.
    *   Managing the DAG parsing process.
    *   Purging old metadata from the database.

---

#### **Part VI: Practical Application & Next Steps**
25. **End-to-End Project: Building a Production ETL Pipeline**
    *   Concept: Extract data from an API, process it, load it to a database, and send a report.
    *   Implementing error handling, monitoring, and alerts.

26. **Testing Airflow Workflows**
    *   Unit testing individual tasks and Python functions.
    *   Integration testing DAGs.
    *   Using the `airflow tasks test` command.

27. **CI/CD for Airflow**
    *   Automating DAG deployment and testing with GitHub Actions/GitLab CI.
    *   DAG validation steps.

28. **Staying Updated & Community**
    *   Following the Apache Airflow Blog and release notes.
    *   Joining the Slack community.
    *   Contributing to the project.


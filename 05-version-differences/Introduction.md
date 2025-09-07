## 🚀 Customizing Apache Airflow Docker Image: Adding Python Packages & Timezone Configuration

When working with **Apache Airflow**, sooner or later you’ll need more than the stock Docker image provides.  
Maybe you want to install **extra Python libraries** like `pandas`, `openpyxl`, or `jdatetime`.  
Or perhaps you need Airflow to run in your **local timezone** instead of UTC.

Instead of hacking around with `_PIP_ADDITIONAL_REQUIREMENTS` or manual tweaks, the clean solution is to **extend the official Airflow image with a custom Dockerfile**.

In this post, we’ll go through:

- 🐳 Writing a custom Airflow Dockerfile

- 📦 Installing extra Python dependencies

- 🌍 Configuring timezone (`Asia/Tehran`)

- 🔧 Updating `docker-compose.yaml` to use our custom image

---

## 1. Why extend the base image?

The official image (`apache/airflow:<version>`) is designed to be minimal.  
By default, it does not ship with heavy libraries like `pandas` or `openpyxl`.

If you install packages on container startup using `_PIP_ADDITIONAL_REQUIREMENTS`, you’ll pay the price **every time the container restarts**. That’s slow and error-prone.

Instead, we’ll bake our dependencies into the image once, and reuse it everywhere.

---

## 2. Writing the Dockerfile

Create a file named `Dockerfile` next to your `docker-compose.yaml`:

```dockerfile
# Start from the official Airflow image
FROM docker.arvancloud.ir/apache/airflow:3.0.6

# Switch to root to install system packages
USER root

# Install timezone data and configure Asia/Tehran
RUN apt-get update \
    && apt-get install -y --no-install-recommends tzdata \
    && ln -fs /usr/share/zoneinfo/Asia/Tehran /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install additional Python dependencies
RUN pip install --no-cache-dir pandas jdatetime openpyxl

# Switch back to the airflow user (important!)
USER airflow
```

What this does:

- Installs `tzdata` and sets system timezone to `Asia/Tehran`

- Installs our Python deps (`pandas`, `jdatetime`, `openpyxl`)

- Returns to the `airflow` user so the container behaves like the official image

---

## 3. Updating docker-compose.yaml

In your `docker-compose.yaml`, replace the `image:` line with a `build:` context so Compose builds our custom Dockerfile:

```yaml
x-airflow-common:
  &airflow-common
  build: .   # Use local Dockerfile instead of pulling from Docker Hub
  environment:
    AIRFLOW__CORE__EXECUTOR: CeleryExecutor
    AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: "true"
    AIRFLOW__CORE__LOAD_EXAMPLES: "false"

    # Force Airflow to use Asia/Tehran as default timezone
    TZ: Asia/Tehran
    AIRFLOW__CORE__DEFAULT_TIMEZONE: Asia/Tehran

  volumes:
    - ./dags:/opt/airflow/dags
    - ./logs:/opt/airflow/logs
    - ./plugins:/opt/airflow/plugins
```

That way:

- All Airflow services (`scheduler`, `workers`, `api-server`, etc.) will be built from the same custom image.

- Both system and Airflow are aligned to `Asia/Tehran`.

- Your Python packages are always preinstalled.

---

## 4. Rebuild and run

Now rebuild and start your stack:

```bash
docker compose build
docker compose up -d
```

Check your image:

```bash
docker exec -it <container_id> python -c "import pandas, jdatetime, openpyxl; print('All good!')"
```

Check timezone:

```bash
docker exec -it <container_id> date
```

You should see the system time aligned with `Asia/Tehran`.

---

## 5. Conclusion

By extending the official Apache Airflow image:

- ✅ You keep startup times fast

- ✅ Dependencies are version-controlled and reproducible

- ✅ Timezone configuration is consistent across all services

This approach is production-friendly and avoids hidden surprises when scaling your Airflow cluster.

## 🧩 Understanding Context Variables in Airflow Python Functions (`**kwargs`)

When writing **Apache Airflow DAGs**, you’ll often see Python functions defined like this:

```python
def my_task(**kwargs):
    ...
```

At first glance, it looks like `kwargs` is just an empty dictionary. But in Airflow, this `kwargs` is full of **context variables** that describe the execution environment. These values are automatically injected by Airflow at runtime and are incredibly useful for building dynamic, data-driven workflows.

In this post, we’ll break down what these context variables are, how to access them, and when to use them.

---

## 1. Why does Airflow pass context variables?

Airflow tasks are executed in a specific context:

- They belong to a **DAG**

- They run at a **scheduled time** (`logical_date`)

- They have unique identifiers (`task_id`, `run_id`)

- They may depend on **upstream/downstream tasks**

To make this information accessible inside Python functions, Airflow passes it through the `context` dictionary (a.k.a. `kwargs`).

This is what allows your tasks to “know” when they’re running and adapt dynamically.

---

## 2. The most common context variables

Here are some of the most useful keys you’ll find in `kwargs`:

| Variable              | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `ti`                  | **TaskInstance object** — access XComs, task state, retries, etc. |
| `ds`                  | Execution date as a string in format `YYYY-MM-DD`.           |
| `ds_nodash`           | Same as `ds` but without dashes (e.g. `20250907`).           |
| `logical_date`        | A `pendulum.DateTime` object representing the logical execution date (preferred over `ds`). |
| `ts`                  | Execution timestamp (`YYYY-MM-DDTHH:MM:SS+00:00`).           |
| `prev_ds` / `next_ds` | Execution dates of the previous/next scheduled run.          |
| `dag`                 | The **DAG object** for the current run.                      |
| `task`                | The **Task object** being executed.                          |
| `run_id`              | The unique identifier of the DAG run.                        |
| `params`              | Any parameters passed via `params` in the DAG definition.    |

👉 Example:

```python
def print_context(**kwargs):
    print("Task Instance:", kwargs["ti"])
    print("DAG:", kwargs["dag"])
    print("Task:", kwargs["task"])
    print("Logical date:", kwargs["logical_date"])
    print("Run ID:", kwargs["run_id"])
```

---

## 3. Deep dive: `dag` and `task` objects

### 🔹 `dag` object

The `dag` key points to the **`DAG` object** your task belongs to.  
This gives you access to metadata and configuration of the whole DAG.

```python
def explore_dag(**kwargs):
    dag = kwargs["dag"]
    print("DAG ID:", dag.dag_id)
    print("Default args:", dag.default_args)
    print("Tasks in DAG:", dag.task_ids)
```

**Common attributes:**

- `dag.dag_id`: Unique DAG identifier

- `dag.default_args`: Default parameters for tasks

- `dag.task_ids`: List of all task IDs in the DAG

- `dag.schedule_interval`: The DAG’s schedule

---

### 🔹 `task` object

The `task` key points to the **`BaseOperator` object** representing the task being executed.  
This is useful for introspection or debugging.

```python
def explore_task(**kwargs):
    task = kwargs["task"]
    print("Task ID:", task.task_id)
    print("Operator:", task.__class__.__name__)
    print("Retries:", task.retries)
    print("Depends on past:", task.depends_on_past)
```

**Common attributes:**

- `task.task_id`: Unique task identifier

- `task.owner`: The owner assigned in the DAG definition

- `task.retries`: Retry count for the task

- `task.upstream_task_ids` / `task.downstream_task_ids`: Task dependencies

---

## 4. Using context variables in tasks

### Example 1: Accessing logical date

```python
def process_for_date(**kwargs):
    date = kwargs["logical_date"].strftime("%Y-%m-%d")
    print(f"Processing data for {date}")
```

### Example 2: Reading/writing XComs via `ti`

```python
def push_xcom(**kwargs):
    ti = kwargs["ti"]
    ti.xcom_push(key="my_value", value=42)

def pull_xcom(**kwargs):
    ti = kwargs["ti"]
    value = ti.xcom_pull(key="my_value", task_ids="push_xcom")
    print("Received:", value)
```

### Example 3: Using DAG & Task metadata

```python
def metadata_logger(**kwargs):
    dag = kwargs["dag"]
    task = kwargs["task"]
    print(f"Running task {task.task_id} from DAG {dag.dag_id}")
    print(f"Task retries: {task.retries}, DAG schedule: {dag.schedule_interval}")
```

---

## 5. Explicit context injection (`op_kwargs`)

You don’t always have to rely on `**kwargs`. Airflow can also inject only the variables you need:

```python
def process_with_context(ti, logical_date, ds, dag, task, **_):
    print("TI:", ti)
    print("Logical Date:", logical_date)
    print("DAG:", dag.dag_id)
    print("Task:", task.task_id)
```

Airflow matches argument names with available context keys.

---

## 6. Best practices

- ✅ Prefer `logical_date` (timezone-aware `pendulum.DateTime`) over the older `ds`.

- ✅ Use `ti` for XComs and task state.

- ✅ Use `dag` and `task` when you need DAG-wide or task-specific metadata.

- ✅ Use `params` for configurable DAGs.

- ⚠️ Don’t confuse `execution_date` (deprecated) with `logical_date`.

---

## 7. Summary

Context variables are Airflow’s way of giving tasks awareness about **when and where they are running**.  
By tapping into variables like `ti`, `logical_date`, `dag`, and `task`, you can build tasks that adapt to schedules, fetch the right data partitions, and coordinate state across DAG runs.

Next time you write `def my_task(**kwargs):`, remember: it’s not just boilerplate — it’s your entry point into Airflow’s scheduling brain 🧠.

---


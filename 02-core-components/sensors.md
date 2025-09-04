# Sensors

### 1. What is a Sensor? The Core Concept

A **Sensor** is a special type of **Airflow Operator** whose sole purpose is to **wait for something to happen**.

*   **Analogy:** Think of a sensor as a **watchdog** or a **doorbell**. It doesn't perform the main action (like cooking food or answering the door), but it patiently waits for a specific trigger (the owner arriving or the bell ringing) and then signals that it's okay to proceed.
*   **Technical Definition:** A Sensor polls an external system at a regular interval (`poke_interval`) to check if a certain condition is met. If the condition is `True`, it succeeds, and its downstream tasks can run. If not, it keeps checking until a `timeout` is reached.

### 2. Why Sensors are Essential for Data Engineering

In real-world data pipelines, you can't always assume data is present at exactly the scheduled time. Sensors make your DAGs **resilient** and **event-driven**.

**Common Use Cases:**
*   **Waiting for data arrival:** Is the file in the S3 bucket or SFTP server?
*   **Waiting for data completeness:** Does the file have the expected number of partitions or records? (e.g., `_SUCCESS` file in Hadoop)
*   **Waiting for upstream processes:** Has another DAG or an external system finished its job?
*   **Waiting for a specific time:** Pausing execution until a certain time of day.

Without sensors, you would have to build this waiting and checking logic into your tasks, which is messy and inefficient.

---

### 3. The Two Fundamental Modes: `poke` vs. `reschedule`

This is the most important concept for using Sensors effectively in production. It directly impacts your **worker slot utilization**.

| Feature              | **`mode='poke'` (Default)**                                  | **`mode='reschedule'`**                                      |
| :------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Behavior**         | The Sensor task **holds onto its worker slot** and sleeps between pokes. | The Sensor task **completes and frees its worker slot**. The Scheduler requeues it for the next check. |
| **Resource Usage**   | **Inefficient.** The worker slot is occupied for the entire waiting period, doing nothing most of the time. | **Efficient.** Worker slots are only occupied for the brief moment of checking the condition. |
| **Analogy**          | A security guard staying at their post, checking their watch every few minutes. | A security guard who goes home and sets an alarm to come back and check later. |
| **When to Use**      | **Short waits** (seconds or a few minutes) where the overhead of rescheduling is worse than holding the slot. | **Long waits** (minutes or hours). **THIS IS THE PREFERRED MODE FOR PRODUCTION.** |
| **Timeout Handling** | If `timeout` is reached, the task fails.                     | If `timeout` is reached, the task fails. The timeout is the total time since the **first** execution. |

**Key Takeaway:** Always use `mode='reschedule'` for sensors that might wait more than a minute. It's the single biggest best practice for sensor usage.

```python
from airflow.sensors.filesystem import FileSensor

# GOOD PRACTICE: Efficient sensor for long waits
wait_for_file = FileSensor(
    task_id='wait_for_file',
    filepath='/data/input.csv',
    mode='reschedule',  # <-- This is crucial
    timeout=60 * 60,   # Total timeout of 1 hour
    poke_interval=5 * 60, # Check every 5 minutes
)
```

---

### 4. Common Built-in Sensor Types (From Core & Providers)

Sensors are part of **provider packages**. You must install the relevant provider.

#### A. File & Object Sensors
*   **`FileSensor`** (`airflow.sensors.filesystem`): Waits for a file or directory to appear on a local or networked filesystem.
*   **`S3KeySensor`** (`airflow.providers.amazon.aws.sensors.s3`): Waits for a specific key (file) to appear in an S3 bucket. Can also check for wildcards (e.g., `data/*.csv`).
*   **`S3PrefixSensor`**: Waits for at least one key matching a prefix to exist.
*   **`GCSObjectSensor`** (`airflow.providers.google.cloud.sensors.gcs`): Google Cloud Storage equivalent.

#### B. Database Sensors
*   **`SqlSensor`** (`airflow.sensors.sql`): Runs a SQL query and waits for it to return a truthy value (e.g., `SELECT COUNT(*) FROM table > 0`). Powerful for checking data quality or completeness.

#### C. Cross-DAG & Workflow Sensors
*   **`ExternalTaskSensor`** (`airflow.sensors.external_task`): **Extremely important.** Waits for a task in a *different DAG* to complete for a specific logical date. Essential for orchestrating dependencies between DAGs.
*   **`ExternalTaskMarker` & `TriggerDagRunOperator`**: Often used alongside the sensor for complex cross-DAG workflows.

#### D. Platform-Specific Sensors
*   **`HivePartitionSensor`** (`airflow.providers.apache.hive.sensors.hive`): Waits for a partition to appear in a Hive table.
*   **`SparkJDBCSensor`**, **`EmrStepSensor`**, etc.: Wait for specific events in big data platforms.

#### E. Time & Logic Sensors
*   **`DateTimeSensor`** (`airflow.sensors.date_time`): Waits until a specific absolute datetime is reached. Useful for pausing a pipeline until a specific time.
*   **`TimeDeltaSensor`** (`airflow.sensors.time_delta`): Waits for a fixed relative amount of time (e.g., `timedelta(minutes=15)`). **Warning:** This holds a worker slot for the entire duration! Avoid it for long waits.
*   **`Smart Sensor`** (Legacy in Airflow 2.2+): A feature that consolidated many sensor checks into a single process to reduce load. Largely superseded by `mode='reschedule'`.

---

### 5. Key Parameters & Configuration

Every sensor inherits from `BaseSensorOperator` and has these core parameters:

*   `mode`: (`poke` | `reschedule`) As discussed above.
*   `poke_interval`: (default=60 sec) Time in seconds to wait between checks.
*   `timeout`: (default=60*60*24*7 sec) Total time in seconds the sensor should keep trying before giving up and **failing**.
*   `soft_fail`: If `True`, the sensor will simply be marked as `skipped` instead of `failed` upon timeout.
*   `exponential_backoff`: If `True`, the poke interval will double after each failed poke, up to a maximum. Useful to avoid overwhelming external systems.

---

### 6. Writing a Custom Sensor (Advanced Pattern)

As a Data Engineer, you might need to wait for a condition that isn't covered by a built-in sensor. Writing a custom sensor is straightforward.

You inherit from `BaseSensorOperator` and override the `poke` method.

**Example: A custom sensor to wait for a specific API response.**

```python
from airflow.sensors.base import BaseSensorOperator
from airflow.utils.decorators import apply_defaults
import requests

class MyAPISensor(BaseSensorOperator):
    """
    Custom sensor to wait for a specific status from an API endpoint.
    """
    
    @apply_defaults
    def __init__(self, endpoint: str, expected_status: str, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.endpoint = endpoint
        self.expected_status = expected_status

    def poke(self, context):
        # This function is called repeatedly until it returns True or timeout
        self.log.info(f'Poking API endpoint: {self.endpoint}')
        try:
            response = requests.get(self.endpoint)
            response.raise_for_status()  # Raise an exception for bad status codes
            data = response.json()
            
            # Check our condition
            current_status = data.get('status')
            if current_status == self.expected_status:
                self.log.info(f"API status is '{current_status}'. Condition met!")
                return True  # Success! The sensor will complete.
            else:
                self.log.info(f"API status is '{current_status}'. Still waiting...")
                return False # Condition not met, keep poking.
                
        except requests.exceptions.RequestException as e:
            self.log.warning(f"API request failed: {e}")
            return False  # On error, we can return False to keep trying.

# Usage in a DAG
wait_for_api = MyAPISensor(
    task_id='wait_for_api_ready',
    endpoint='https://my-data-api.com/status',
    expected_status='COMPLETED',
    mode='reschedule', # Use reschedule mode for efficiency
    poke_interval=120, # Check every 2 minutes
    timeout=3600,      # Total timeout after 1 hour
    dag=dag
)
```

---

### 7. Best Practices & Pitfalls for Production

1.  **ALWAYS Use `mode='reschedule'` for Long Waits:** This is non-negotiable for production-grade pipelines. It prevents your workers from being clogged by sleeping tasks.
2.  **Always Set a `timeout`:** A sensor without a timeout might wait forever, creating "zombie" tasks that never clear. This is a common pitfall.
3.  **Use `soft_fail=True` Judiciously:** It can be useful if the condition becoming false is a valid outcome, but often a timeout is a real failure that should alert you.
4.  **Leverage `exponential_backoff` for APIs:** When poking external APIs, use exponential backoff to be a good citizen and avoid getting rate-limited.
5.  **Prefer `ExternalTaskSensor` over Sleep/Delay:** To orchestrate DAGs, use sensors instead of hardcoded delays. They are more reliable and precise.
6.  **Beware of the `TimeDeltaSensor`:** Remember it uses `mode='poke'` by default and will hold a worker slot. Avoid it for anything more than a few seconds.
7.  **Test Sensor Timeouts:** Ensure your DAG behavior is correct when a sensor times out. Does the whole DAG fail? Is that the desired behavior?

### Summary: The Data Engineer's Sensor Checklist

*   **Purpose:** Wait for a condition before proceeding.
*   **Key Parameter:** `mode='reschedule'` (for efficiency).
*   **Mandatory Parameter:** `timeout` (to avoid zombies).
*   **Common Types:** `FileSensor`, `S3KeySensor`, `SqlSensor`, `ExternalTaskSensor`.
*   **Advanced Use:** Write custom sensors by inheriting from `BaseSensorOperator` and overriding the `poke(context)` method.
*   **Production Mindset:** Sensors make your pipelines resilient but must be configured correctly to not waste resources.

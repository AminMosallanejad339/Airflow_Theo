# Apache Airflow Trigger Rules

## Overview

In Apache Airflow, trigger rules determine when a task should be executed based on the state of its upstream (dependent) tasks. Understanding trigger rules is crucial for building robust and flexible data pipelines.

## Default Behavior

By default, Airflow uses the `ALL_SUCCESS` trigger rule, meaning a task will only execute if **all** of its upstream tasks have succeeded.

## Available Trigger Rules

### 1. `ALL_SUCCESS` (default)
- **Description**: Task runs only when all upstream tasks have succeeded
- **Use Case**: Standard sequential processing where each step depends on the previous one's success

```python
task1 >> task2  # task2 uses ALL_SUCCESS by default
```

### 2. `ALL_FAILED`
- **Description**: Task runs only when all upstream tasks have failed
- **Use Case**: Error handling, cleanup operations, or notifications when everything fails

```python
task1 = PythonOperator(
    task_id='process_data',
    python_callable=process_function,
    dag=dag
)

cleanup = PythonOperator(
    task_id='cleanup_on_failure',
    python_callable=cleanup_function,
    trigger_rule='all_failed',
    dag=dag
)

task1 >> cleanup
```

### 3. `ALL_DONE`
- **Description**: Task runs when all upstream tasks are completed, regardless of success or failure
- **Use Case**: Aggregation tasks, final reporting, or cleanup that should always run

```python
report = PythonOperator(
    task_id='generate_report',
    python_callable=report_function,
    trigger_rule='all_done',
    dag=dag
)

task1 >> report
task2 >> report
```

### 4. `ONE_SUCCESS`
- **Description**: Task runs when at least one upstream task has succeeded
- **Use Case**: Multiple data sources where you need only one to succeed

```python
source1 = PythonOperator(task_id='source1', ...)
source2 = PythonOperator(task_id='source2', ...)

process = PythonOperator(
    task_id='process_data',
    python_callable=process_function,
    trigger_rule='one_success',
    dag=dag
)

[source1, source2] >> process
```

### 5. `ONE_FAILED`
- **Description**: Task runs when at least one upstream task has failed
- **Use Case**: Alerting or handling partial failures

```python
task1 = PythonOperator(task_id='task1', ...)
task2 = PythonOperator(task_id='task2', ...)

alert = PythonOperator(
    task_id='send_alert',
    python_callable=alert_function,
    trigger_rule='one_failed',
    dag=dag
)

[task1, task2] >> alert
```

### 6. `NONE_FAILED`
- **Description**: Task runs when all upstream tasks have either succeeded or been skipped (no failures)
- **Use Case**: Continue processing when some tasks might be skipped but none failed

```python
task1 = PythonOperator(task_id='task1', ...)
task2 = PythonOperator(task_id='task2', ...)  # This might be skipped

continue_process = PythonOperator(
    task_id='continue_processing',
    python_callable=continue_function,
    trigger_rule='none_failed',
    dag=dag
)

[task1, task2] >> continue_process
```

### 7. `NONE_FAILED_MIN_ONE_SUCCESS`
- **Description**: Task runs when no upstream tasks failed and at least one succeeded
- **Use Case**: Ensure at least one successful path before continuing

```python
branch1 = PythonOperator(task_id='branch1', ...)
branch2 = PythonOperator(task_id='branch2', ...)  # Might be skipped

merge = PythonOperator(
    task_id='merge_results',
    python_callable=merge_function,
    trigger_rule='none_failed_min_one_success',
    dag=dag
)

[branch1, branch2] >> merge
```

### 8. `NONE_SKIPPED`
- **Description**: Task runs when no upstream tasks were skipped
- **Use Case**: Ensure all possible paths were executed before continuing

```python
task1 = PythonOperator(task_id='task1', ...)
task2 = PythonOperator(task_id='task2', ...)  # Should not be skipped

final = PythonOperator(
    task_id='final_task',
    python_callable=final_function,
    trigger_rule='none_skipped',
    dag=dag
)

[task1, task2] >> final
```

### 9. `DUMMY`
- **Description**: Task always runs, regardless of upstream task states
- **Use Case**: Manual intervention points, testing, or tasks that should always execute

```python
manual_check = PythonOperator(
    task_id='manual_verification',
    python_callable=check_function,
    trigger_rule='dummy',
    dag=dag
)
```

## Practical Examples

### Example 1: Error Handling Pipeline
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def process_data():
    # Your data processing logic
    pass

def handle_failure():
    # Cleanup or notification logic
    pass

def final_report():
    # Final reporting logic
    pass

with DAG('error_handling_example', start_date=datetime(2023, 1, 1)) as dag:
    
    process = PythonOperator(
        task_id='process_data',
        python_callable=process_data
    )
    
    cleanup = PythonOperator(
        task_id='cleanup_on_failure',
        python_callable=handle_failure,
        trigger_rule='all_failed'
    )
    
    report = PythonOperator(
        task_id='generate_report',
        python_callable=final_report,
        trigger_rule='all_done'
    )
    
    process >> cleanup
    process >> report
    cleanup >> report
```

### Example 2: Multiple Data Sources
```python
with DAG('multiple_sources', start_date=datetime(2023, 1, 1)) as dag:
    
    source_a = PythonOperator(task_id='source_a', ...)
    source_b = PythonOperator(task_id='source_b', ...)
    source_c = PythonOperator(task_id='source_c', ...)
    
    process = PythonOperator(
        task_id='process_data',
        python_callable=process_function,
        trigger_rule='one_success'  # Process if any source succeeds
    )
    
    alert = PythonOperator(
        task_id='alert_if_all_fail',
        python_callable=alert_function,
        trigger_rule='all_failed'  # Alert only if all sources fail
    )
    
    [source_a, source_b, source_c] >> process
    [source_a, source_b, source_c] >> alert
```

## Best Practices

1. **Use Default When Possible**: Stick with `ALL_SUCCESS` for most cases as it's the most intuitive
2. **Document Complex Rules**: Add comments when using non-default trigger rules
3. **Test Thoroughly**: Complex trigger rule combinations can have unexpected behaviors
4. **Consider Task States**: Understand how SKIPPED, SUCCESS, and FAILED states interact
5. **Use Branch Operators**: Combine with BranchPythonOperator for complex conditional logic

## Common Pitfalls

1. **Deadlocks**: Circular dependencies or impossible trigger conditions
2. **Unexpected Skips**: Tasks might be skipped due to upstream branch operations
3. **Debugging Complexity**: Complex trigger rules can make debugging more difficult
4. **Resource Wastage**: `DUMMY` rule can cause unnecessary resource consumption

## Version Compatibility

- All trigger rules mentioned are available in Airflow 2.x and 3.x
- Some older deprecated rules like `ALL_SUCCESS` (deprecated alias) have been standardized
- Always check the official Airflow documentation for your specific version

## Conclusion

Trigger rules are powerful tools for building flexible and robust data pipelines in Airflow. By understanding and appropriately using different trigger rules, you can create pipelines that handle various scenarios including error conditions, partial successes, and complex dependencies.

Remember to choose the simplest trigger rule that meets your requirements and thoroughly test your DAGs to ensure they behave as expected under different scenarios.

# 📘 Airflow Trigger Rules Demo

## 🎯 Overview

By default, Airflow tasks only run when **all upstream tasks succeed** (`all_success`).  
But sometimes you want different behavior:

- Run a task even if some upstreams fail.

- Run a task if all upstreams fail.

- Run regardless of status.

- Run only if nothing was skipped.

This project contains **two DAGs** that demonstrate these behaviors:

- `sample-12.py` → **failures vs successes**.

- `sample-13.py` → **skips from branching**.

---

## 🗂 DAGs

### 1️⃣ `sample-12.py`

- One task always succeeds, another always fails.

- Downstream tasks demonstrate:
  
  - `all_success`
  
  - `one_failed`
  
  - `all_done`

### 2️⃣ `sample-13.py`

- Uses `BranchPythonOperator` (odd/even day).

- One branch is **skipped** each run.

- Downstream tasks demonstrate:
  
  - `none_failed_min_one_success`
  
  - `all_failed`
  
  - `none_skipped`

---

## 📊 Trigger Rule Reference Table

| Trigger Rule                  | Behavior                                                       |
| ----------------------------- | -------------------------------------------------------------- |
| `all_success` *(default)*     | Run only if **all upstream tasks succeeded**.                  |
| `all_failed`                  | Run only if **all upstream tasks failed**.                     |
| `all_done`                    | Run regardless of upstream state (success, fail, skipped).     |
| `one_success`                 | Run if **at least one upstream succeeded**.                    |
| `one_failed`                  | Run if **at least one upstream failed**.                       |
| `none_failed`                 | Run if **no upstream task failed** (success or skipped is ok). |
| `none_skipped`                | Run only if **no upstream task was skipped**.                  |
| `none_failed_min_one_success` | Run if **no upstream failed AND at least one succeeded**.      |

---

## ▶️ How to Run

1. Start Airflow (you can use **Astronomer**, **Docker Compose**, or your own setup).

2. Copy the two DAG files into your DAGs folder:
   
   ```
   dags/
    ├── sample-12.py
    └── sample-13.py
   ```

3. In the Airflow UI:
   
   - Trigger **`sample-12`**
     
     - One task fails, one succeeds.
     
     - Observe which downstream tasks run.
   
   - Trigger **`sample-13`**
     
     - Depending on the day, one branch is skipped.
     
     - Observe which downstream tasks run.

---

## 🔎 Learning Outcomes

- Understand the **default trigger rule** (`all_success`).

- See how tasks can run on **failure** or **skip** conditions.

- Build mental models of DAG execution when not all upstream tasks succeed.

# Apache Airflow Trigger Rules - Complete Reference Table

## Overview

Here's a comprehensive reference table of all trigger rules available in Apache Airflow 2.x/3.x, including their behavior, use cases, and examples.

## Complete Trigger Rules Reference Table

| Trigger Rule                      | Description                          | When Task Runs                                               | Use Cases                                       | Example Scenario                                |
| --------------------------------- | ------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- | ----------------------------------------------- |
| **`all_success`** (default)       | All upstream tasks succeeded         | All direct upstream tasks are in `SUCCESS` state             | Sequential processing, strict dependencies      | Data validation → Data processing               |
| **`all_failed`**                  | All upstream tasks failed            | All direct upstream tasks are in `FAILED` state              | Error handling, cleanup, failure notifications  | Multiple fallback sources all fail → Send alert |
| **`all_done`**                    | All upstream tasks completed         | All direct upstream tasks are in terminal states (`SUCCESS`, `FAILED`, or `SKIPPED`) | Final reporting, aggregation, always-run tasks  | Multiple processing branches → Final report     |
| **`one_success`**                 | At least one upstream task succeeded | At least one direct upstream task is in `SUCCESS` state      | Multiple data sources, fallback mechanisms      | Source A OR Source B succeeds → Process data    |
| **`one_failed`**                  | At least one upstream task failed    | At least one direct upstream task is in `FAILED` state       | Partial failure handling, early alerts          | Task A OR Task B fails → Trigger investigation  |
| **`none_failed`**                 | No upstream tasks failed             | All direct upstream tasks are in `SUCCESS` or `SKIPPED` states | Continue when some paths are optionally skipped | Optional preprocessing → Main processing        |
| **`none_failed_min_one_success`** | No failures + at least one success   | No upstream tasks failed AND at least one succeeded          | Ensure at least one successful path exists      | Multiple strategies → Continue if any works     |
| **`none_skipped`**                | No upstream tasks were skipped       | All direct upstream tasks are in `SUCCESS` or `FAILED` states (not `SKIPPED`) | Require all possible paths to complete          | Mandatory validation steps → Final approval     |
| **`dummy`**                       | Always run                           | Regardless of upstream task states                           | Manual steps, testing, always-execute tasks     | Manual verification step                        |

## Detailed Behavior Matrix

### Task State Combinations

| Upstream States        | all_success | all_failed | all_done | one_success | one_failed | none_failed | none_failed_min_one_success | none_skipped | dummy  |
| ---------------------- | ----------- | ---------- | -------- | ----------- | ---------- | ----------- | --------------------------- | ------------ | ------ |
| All SUCCESS            | ✅ Runs      | ❌ Skips    | ✅ Runs   | ✅ Runs      | ❌ Skips    | ✅ Runs      | ✅ Runs                      | ✅ Runs       | ✅ Runs |
| All FAILED             | ❌ Skips     | ✅ Runs     | ✅ Runs   | ❌ Skips     | ✅ Runs     | ❌ Skips     | ❌ Skips                     | ✅ Runs       | ✅ Runs |
| All SKIPPED            | ❌ Skips     | ❌ Skips    | ✅ Runs   | ❌ Skips     | ❌ Skips    | ✅ Runs      | ❌ Skips                     | ❌ Skips      | ✅ Runs |
| Mixed SUCCESS/FAILED   | ❌ Skips     | ❌ Skips    | ✅ Runs   | ✅ Runs      | ✅ Runs     | ❌ Skips     | ❌ Skips                     | ✅ Runs       | ✅ Runs |
| Mixed SUCCESS/SKIPPED  | ❌ Skips     | ❌ Skips    | ✅ Runs   | ✅ Runs      | ❌ Skips    | ✅ Runs      | ✅ Runs                      | ❌ Skips      | ✅ Runs |
| Mixed FAILED/SKIPPED   | ❌ Skips     | ❌ Skips    | ✅ Runs   | ❌ Skips     | ✅ Runs     | ❌ Skips     | ❌ Skips                     | ❌ Skips      | ✅ Runs |
| SUCCESS/FAILED/SKIPPED | ❌ Skips     | ❌ Skips    | ✅ Runs   | ✅ Runs      | ✅ Runs     | ❌ Skips     | ❌ Skips                     | ❌ Skips      | ✅ Runs |

## Code Examples for Each Trigger Rule

### 1. all_success (Default)
```python
task1 = PythonOperator(task_id='validate_data', ...)
task2 = PythonOperator(task_id='process_data', ...)  # Uses all_success by default
task1 >> task2
```

### 2. all_failed
```python
source1 = PythonOperator(task_id='source1', ...)
source2 = PythonOperator(task_id='source2', ...)

alert = PythonOperator(
    task_id='critical_alert',
    python_callable=send_alert,
    trigger_rule='all_failed'
)
[source1, source2] >> alert
```

### 3. all_done
```python
task_a = PythonOperator(task_id='task_a', ...)
task_b = PythonOperator(task_id='task_b', ...)

report = PythonOperator(
    task_id='final_report',
    python_callable=generate_report,
    trigger_rule='all_done'
)
[task_a, task_b] >> report
```

### 4. one_success
```python
api_source = PythonOperator(task_id='api_data', ...)
db_source = PythonOperator(task_id='database_data', ...)

processor = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    trigger_rule='one_success'
)
[api_source, db_source] >> processor
```

### 5. one_failed
```python
validation1 = PythonOperator(task_id='validate_1', ...)
validation2 = PythonOperator(task_id='validate_2', ...)

investigate = PythonOperator(
    task_id='investigate_issue',
    python_callable=investigate_function,
    trigger_rule='one_failed'
)
[validation1, validation2] >> investigate
```

### 6. none_failed
```python
preprocess_opt = PythonOperator(task_id='optional_preprocess', ...)  # Might be skipped
main_process = PythonOperator(
    task_id='main_processing',
    python_callable=main_function,
    trigger_rule='none_failed'  # Runs if optional task skipped or succeeded
)
preprocess_opt >> main_process
```

### 7. none_failed_min_one_success
```python
strategy_a = PythonOperator(task_id='strategy_a', ...)
strategy_b = PythonOperator(task_id='strategy_b', ...)  # Might be skipped

continue_process = PythonOperator(
    task_id='continue_workflow',
    python_callable=continue_function,
    trigger_rule='none_failed_min_one_success'
)
[strategy_a, strategy_b] >> continue_process
```

### 8. none_skipped
```python
mandatory_check1 = PythonOperator(task_id='check1', ...)
mandatory_check2 = PythonOperator(task_id='check2', ...)  # Must not be skipped

approval = PythonOperator(
    task_id='final_approval',
    python_callable=approve_function,
    trigger_rule='none_skipped'  # Only if both checks ran
)
[mandatory_check1, mandatory_check2] >> approval
```

### 9. dummy
```python
manual_review = PythonOperator(
    task_id='manual_review',
    python_callable=review_function,
    trigger_rule='dummy'  # Always runs regardless of upstream
)
```

## Advanced Usage Patterns

### Combined Trigger Rules in Complex DAGs

```python
with DAG('complex_workflow', start_date=days_ago(1)) as dag:
    
    # Multiple data sources
    source1 = PythonOperator(task_id='source1', ...)
    source2 = PythonOperator(task_id='source2', ...)
    source3 = PythonOperator(task_id='source3', ...)
    
    # Process if any source succeeds
    process_data = PythonOperator(
        task_id='process_data',
        python_callable=process_function,
        trigger_rule='one_success'
    )
    
    # Alert only if all sources fail
    alert_all_failed = PythonOperator(
        task_id='alert_all_failed',
        python_callable=alert_function,
        trigger_rule='all_failed'
    )
    
    # Final report always runs
    final_report = PythonOperator(
        task_id='final_report',
        python_callable=report_function,
        trigger_rule='all_done'
    )
    
    # Set dependencies
    sources = [source1, source2, source3]
    sources >> process_data
    sources >> alert_all_failed
    sources >> final_report
    process_data >> final_report
    alert_all_failed >> final_report
```

## Version-Specific Notes

- **Airflow 1.x**: Some trigger rules had different names or behaviors
- **Airflow 2.x/3.x**: Consistent trigger rule implementation
- **Deprecated**: `ALL_SUCCESS` (uppercase) is deprecated in favor of `all_success` (lowercase)

## Best Practices Summary

1. **Start Simple**: Use default `all_success` unless you need specific behavior
2. **Test Edge Cases**: Test with various upstream state combinations
3. **Document Clearly**: Comment complex trigger rule usage
4. **Monitor**: Watch for unexpected task skipping or execution
5. **Combine with Branching**: Use with BranchPythonOperator for complex logic

This reference table provides a complete overview of all trigger rules available in Apache Airflow, helping you design robust and flexible data pipelines.

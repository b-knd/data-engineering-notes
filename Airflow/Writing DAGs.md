# Writing DAGs
### Importing Modules
```python
import textwrap
from datetime import datetime, timedelta

from airflow.models.dag import DAG --for DAG instantiation
from airflow.operators.bash import BashOperator
```

### Default Arguments
The arguments to be passed to task's constructor defined as dictionary
```python
# These args will get passed on to each operator
# You can override them on a per-task basis during operator initialization
default_args={
    "owner": 'data_team',
    "depends_on_past": False,
    "email": ["airflow@example.com"],
    "email_on_failure": False,
    "email_on_retry": False,
    "retries": 1,
    "retry_delay": timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
    # 'wait_for_downstream': False,
    # 'sla': timedelta(hours=2),
    # 'execution_timeout': timedelta(seconds=300),
    # 'on_failure_callback': some_function, # or list of functions
    # 'on_success_callback': some_other_function, # or list of functions
    # 'on_retry_callback': another_function, # or list of functions
    # 'sla_miss_callback': yet_another_function, # or list of functions
    # 'on_skipped_callback': another_function, #or list of functions
    # 'trigger_rule': 'all_success'
},
```
Some definition of arguments:
_** `owner`: person responsible for DAG, for logging or alerting purposes_

_** `depends_on_past`: True - will not run until previous instance of task has completed. To ensure sequential task order in DAG runs. (Default is set to False)_

_** `wait_for_downstream`: True - ensure tasks wait for completion of downstream tasks to enforce task dependencies_

_** `on_failure_callback`: list down functions to be executed when task failes (sending alerts...)_

_** `start_date`: date and time for DAG to start running (first schedule), conventionally set to sometime in the past to avoid DAG's being marked as up-to-date_

_** `email_on_failure`: True - an email will be sent on task failure (Prerequisite: email has been set and configured, Default: True)_

_** `email_on_retry': True - an email will be sent for task retries after failure (Default: True)_
_** `retries`: number of times to retry task if it failes before marking as failed (Default: 0)_

_** `retry_delay`: time delay between retries (set with timedelta) (Default: no delay)_

_** `retry_exponential_backoff`: True - retry delay will increase exponentially, eg. first delay 5 minutes, next 10 minutes... (Default: False)_

_** `max_retry_delay`: Usually use with `retry_exponential_backoff` to limit the maximum delay of retries_

_** `email`: list of email addresses to send failure or retry notifications_

_** `execution_timeout`: maximum amount of time a task is allowed to run before it is marked as failed_

_** `catchup`: determine if backfilling is needed, if True will run DAG for all intervals since start_date (Default: True)_

_** `priority_weight`: determine task's priority, task with higher priority wil be run first in queue (Default: 1)_

_** `provide_context`: True - Airflow will pass a set of context variables to Python function that is executed as task (Default: False)_

_** `sla`: Service Level Agreement for task, if task run longer than this, reported as missed SLA_

_** `trigger_rule`: define how task should be triggered based on state of upstream tasks (ex: `all_success`, `all_failed`, `one_failed`, `none_failed`, `dummy` (run without considering upstream states)_

### DAG instantiation
```python
with DAG(
  "task",
  default_args={
    "depends_on_past": False,
    "email": ["example@gmail.com"],
    "email_on_failure": False,
    "email_on_retry": False,
    "retries": 1,
    "retry_delay": timedelta(minutes=5)
  }
  description = "A simple DAG",
  schedule=timedelta(days=1),
  start_date=datetime(2021, 1, 1),
  catchup=False,
  tags=["example"],
) as dag:
```

### Task definition
```python
t1 = BashOperator (
  task_id = "print_date",
  bash_command="date",
)

t2 = BashOperator (
  task_id = "sleep",
  depends_on_past = False,
  bask_command = "sleep 5",
  retries = 3,
```
_** Note: precedence rules for tasks: explicitly passed arguments -> values in default_args dictionary -> operator's default value if exists_

### Full code
```python
import textwrap
from datetime import datetime, timedelta

# The DAG object; we'll need this to instantiate a DAG
from airflow.models.dag import DAG

# Operators; we need this to operate!
from airflow.operators.bash import BashOperator
with DAG(
    "tutorial",
    # These args will get passed on to each operator
    # You can override them on a per-task basis during operator initialization
    default_args={
        "depends_on_past": False,
        "email": ["airflow@example.com"],
        "email_on_failure": False,
        "email_on_retry": False,
        "retries": 1,
        "retry_delay": timedelta(minutes=5),
        # 'queue': 'bash_queue',
        # 'pool': 'backfill',
        # 'priority_weight': 10,
        # 'end_date': datetime(2016, 1, 1),
        # 'wait_for_downstream': False,
        # 'sla': timedelta(hours=2),
        # 'execution_timeout': timedelta(seconds=300),
        # 'on_failure_callback': some_function, # or list of functions
        # 'on_success_callback': some_other_function, # or list of functions
        # 'on_retry_callback': another_function, # or list of functions
        # 'sla_miss_callback': yet_another_function, # or list of functions
        # 'on_skipped_callback': another_function, #or list of functions
        # 'trigger_rule': 'all_success'
    },
    description="A simple tutorial DAG",
    schedule=timedelta(days=1),
    start_date=datetime(2021, 1, 1),
    catchup=False,
    tags=["example"],
) as dag:

    # t1, t2 and t3 are examples of tasks created by instantiating operators
    t1 = BashOperator(
        task_id="print_date",
        bash_command="date",
    )

    t2 = BashOperator(
        task_id="sleep",
        depends_on_past=False,
        bash_command="sleep 5",
        retries=3,
    )
    t1.doc_md = textwrap.dedent(
        """\
    #### Task Documentation
    You can document your task using the attributes `doc_md` (markdown),
    `doc` (plain text), `doc_rst`, `doc_json`, `doc_yaml` which gets
    rendered in the UI's Task Instance Details page.
    ![img](https://imgs.xkcd.com/comics/fixing_problems.png)
    **Image Credit:** Randall Munroe, [XKCD](https://xkcd.com/license.html)
    """
    )

    dag.doc_md = __doc__  # providing that you have a docstring at the beginning of the DAG; OR
    dag.doc_md = """
    This is a documentation placed anywhere
    """  # otherwise, type it like this
    templated_command = textwrap.dedent(
        """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
    {% endfor %}
    """
    )

    t3 = BashOperator(
        task_id="templated",
        depends_on_past=False,
        bash_command=templated_command,
    )

    t1 >> [t2, t3]
```

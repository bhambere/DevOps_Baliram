# 🚀 Apache Airflow - Complete Summary Guide

---

## 🍕 Understand Airflow Like a Pizza Restaurant (Read This First!)

Before diving into technical terms, let's understand Airflow using a simple real-life story.

### The Story:
> Imagine you own a **pizza restaurant** that gets thousands of orders every day.
> You need the right person doing the right job at the right time, in the right order.
> That's exactly what Airflow does - but for **data tasks** instead of pizza!

### The Restaurant = Your Airflow Cluster

| Restaurant | Airflow |
|---|---|
| 📋 Recipe book | DAG file (.py) |
| 👨‍💼 Manager (decides when to cook) | Scheduler |
| 👨‍🍳 Head Chef (assigns work) | Executor |
| 🧑‍🍳 Junior Chefs (actually cook) | Workers |
| 🖥️ Front display board (shows orders) | Webserver (localhost:8080) |
| 📒 Order logbook (remembers everything) | MetaDB (Postgres) |

### The Full Flow - Step by Step:

```
YOU write DAG file (.py)           → You write the recipe
        ↓
SCHEDULER reads it                 → Manager reads the recipe & schedule
        ↓
SCHEDULER tells EXECUTOR           → "Table 5 ordered! Start cooking!"
        ↓
EXECUTOR assigns to WORKER         → "Junior Chef 1, make the pizza!"
        ↓
WORKER runs the task               → Chef cooks the pizza (runs your code)
        ↓
WORKER saves result to MetaDB      → Writes in logbook "pizza done ✅"
        ↓
WEBSERVER shows you the status     → Board shows "Order 5 - Delivered ✅"
```

### One Line Summary for Each Component:

| Component | One Line |
|---|---|
| **Scheduler** | The brain that decides WHEN to run |
| **Executor** | The manager that decides HOW to run |
| **Worker** | The hands that actually DO the work |
| **Webserver** | The screen that SHOWS you everything |
| **MetaDB** | The memory that REMEMBERS everything |

> **Bottom line**: You write the DAG → Scheduler reads it → Executor plans it → Worker runs it → MetaDB remembers it → Webserver shows it. 🎯

---

## 🌍 Real Life Easy Examples of Airflow

### Example 1: Morning News Email (Simple)
Every morning at 7 AM, you want to receive a summary of top news.

```
[Fetch News from API] → [Filter Top 10 Stories] → [Format into Email] → [Send Email]
     (Extract)               (Transform)                (Transform)         (Load)
```
- Runs daily at 7 AM automatically
- If "Fetch News" fails, it retries 3 times before alerting you
- You can see each step's status in the UI

---

### Example 2: E-commerce Daily Sales Report
Every night at midnight, process the day's sales data.

```
[Extract orders from DB] → [Calculate totals] → [Load to warehouse] → [Email report to CEO]
```
- Runs at `00:00` every day
- If DB is slow, the extract task retries automatically
- CEO gets report every morning without anyone touching it manually

---

### Example 3: Machine Learning Pipeline (Advanced)
Every week, retrain your recommendation model with new data.

```
[Download new data] → [Clean data] → [Train model] → [Evaluate accuracy]
                                                              ↓
                                              If accuracy > 90% → [Deploy model]
                                              If accuracy < 90% → [Alert team]
```
- Uses BranchOperator to decide: deploy or alert
- Runs every Sunday automatically
- Fully automated, no human needed unless accuracy drops

---

### Example 4: File Processing (Sensor Example)
Wait for a file to arrive, then process it.

```
[Wait for file to arrive] → [Read file] → [Validate data] → [Load to database]
      (FileSensor)
```
- Sensor checks every 30 seconds: "Is the file there yet?"
- Once file arrives, rest of the pipeline runs automatically
- Used in banks, healthcare for receiving daily data files

---

### Example 5: Your Labs (What You Built!)

**Lab 2 - Hello World:**
```
[greet] → [farewell]
```

**Lab 3 - ETL Pipeline:**
```
[extract from API] → [transform/clean data] → [load/print data]
```

**Lab 4 - Branching:**
```
[start] → [quality check] → Score ≥ 70: [process good data] → [end]
                         → Score < 70: [handle bad data]   → [end]
```

---

## 1️⃣ What is Apache Airflow?

Apache Airflow is an **open-source workflow orchestration platform** used to programmatically **author, schedule, and monitor** data pipelines.

- Created by **Airbnb in 2014**, donated to **Apache Software Foundation in 2019**
- Written in **Python**
- Used by: Airbnb, Twitter, LinkedIn, NASA, Uber, PayPal
- Best for: **ETL pipelines, ML workflows, data engineering, automation**

> **Simple definition**: Airflow is like a **smart alarm clock + task manager** for your data jobs.
> It runs the right task, at the right time, in the right order, and tells you if something goes wrong.

---

## 2️⃣ Architecture & Flow

```
                        ┌─────────────────────────────────────────┐
                        │             AIRFLOW CLUSTER              │
                        │                                          │
  DAG Files (.py)  -->  │  ┌──────────┐     ┌─────────────────┐  │
                        │  │Scheduler │────>│    Executor      │  │
                        │  └──────────┘     │  (Local/Celery/  │  │
                        │       │           │   Kubernetes)    │  │
                        │       │           └────────┬────────┘  │
                        │       ▼                    │            │
                        │  ┌──────────┐              ▼            │
  Browser/API     -->   │  │Webserver │     ┌─────────────────┐  │
                        │  └──────────┘     │    Workers       │  │
                        │       │           │  (Run Tasks)     │  │
                        │       ▼           └─────────────────┘  │
                        │  ┌──────────┐              │            │
                        │  │MetaDB    │◄─────────────┘            │
                        │  │(Postgres)│                           │
                        │  └──────────┘                           │
                        └─────────────────────────────────────────┘
```

### How it flows (Step by Step):

```
1. You write a DAG (Python file)
2. Scheduler reads the DAG file
3. Scheduler decides when to run (based on schedule)
4. Scheduler sends tasks to Executor
5. Executor assigns tasks to Workers
6. Workers run the tasks
7. Results saved in MetaDatabase
8. Webserver shows you the status
```

---

## 3️⃣ Main Components

| Component       | Role                              | Analogy                    |
|-----------------|-----------------------------------|----------------------------|
| **DAG**         | Pipeline blueprint                | Recipe                     |
| **Task**        | Single unit of work               | One cooking step           |
| **Operator**    | Defines what a task does          | The tool (oven, knife)     |
| **Scheduler**   | Decides when to run tasks         | Alarm clock                |
| **Executor**    | Decides how to run tasks          | Kitchen manager            |
| **Worker**      | Actually runs the task            | The chef                   |
| **Webserver**   | UI for monitoring                 | Kitchen display board      |
| **MetaDatabase**| Stores all state/history          | Kitchen logbook            |
| **XCom**        | Data sharing between tasks        | Passing ingredients        |
| **Hook**        | Connection to external systems    | Plug/adapter               |
| **Connection**  | Stored credentials                | Password vault             |
| **Sensor**      | Waits for a condition             | Watchman                   |
| **Pool**        | Limits concurrent tasks           | Traffic controller         |
| **Variable**    | Global config values              | Settings file              |

---

## 4️⃣ Types of Operators

```python
# Run Python code
PythonOperator(python_callable=my_function)

# Run Shell command
BashOperator(bash_command='echo hello')

# Run SQL
PostgresOperator(sql='SELECT * FROM table')

# Send email
EmailOperator(to='team@company.com', subject='Done!')

# Wait for file
FileSensor(filepath='/data/input.csv')

# Wait for API response
HttpSensor(endpoint='/health', http_conn_id='my_api')

# Conditional branching
BranchPythonOperator(python_callable=decide_path)

# Empty placeholder (Airflow 3.x)
EmptyOperator(task_id='start')
```

---

## 5️⃣ Real World Examples

| Industry        | Use Case                  | DAG Tasks                                              |
|-----------------|---------------------------|--------------------------------------------------------|
| **E-commerce**  | Daily sales report        | Extract orders → Aggregate → Send email                |
| **Finance**     | Risk pipeline             | Pull market data → Calculate risk score → Alert        |
| **Healthcare**  | Patient data ETL          | Extract from hospital DB → Clean → Load to warehouse   |
| **ML / AI**     | Model retraining          | Pull new data → Train → Evaluate → Deploy if better    |
| **Social Media**| Content moderation        | Fetch posts → Run AI classifier → Flag bad content     |
| **DevOps**      | Infrastructure check      | Check server health → Auto-scale → Notify if issue     |
| **Gaming**      | Player analytics          | Collect events → Aggregate → Update leaderboard        |

---

## 6️⃣ Schedule Interval Options

| Value          | Meaning                        |
|----------------|--------------------------------|
| `@once`        | Run once only                  |
| `@hourly`      | Every hour                     |
| `@daily`       | Every day at midnight          |
| `@weekly`      | Every Sunday                   |
| `@monthly`     | First day of every month       |
| `'0 6 * * *'`  | Every day at 6 AM (cron)       |
| `None`         | Manual trigger only            |

---

## 7️⃣ Important Commands

### DAG Management
```bash
# List all DAGs
airflow dags list

# Trigger a DAG manually
airflow dags trigger <dag_id>

# Pause a DAG
airflow dags pause <dag_id>

# Unpause a DAG
airflow dags unpause <dag_id>

# Delete a DAG
airflow dags delete <dag_id>

# Check import errors
airflow dags list-import-errors

# Backfill (run for past dates)
airflow dags backfill -s 2024-01-01 -e 2024-01-31 <dag_id>
```

### Task Management
```bash
# Test a single task (without saving to DB)
airflow tasks test <dag_id> <task_id> <date>

# List tasks in a DAG
airflow tasks list <dag_id>

# Clear a failed task (to re-run it)
airflow tasks clear <dag_id> -t <task_id>
```

### Variables & Connections
```bash
# Set a variable
airflow variables set my_key my_value

# Get a variable
airflow variables get my_key

# List connections
airflow connections list

# Add a connection
airflow connections add my_postgres \
  --conn-type postgres \
  --conn-host localhost \
  --conn-login user \
  --conn-password pass \
  --conn-port 5432
```

### Docker Specific
```bash
# Run command inside Airflow container
docker compose exec airflow-scheduler airflow dags list

# View logs
docker compose logs -f airflow-scheduler

# Restart Airflow
docker compose down && docker compose up -d

# Check container resource usage
docker stats --no-stream
```

---

## 8️⃣ Real World Issues & Fixes

| Issue                          | Cause                         | Fix                                          |
|--------------------------------|-------------------------------|----------------------------------------------|
| `schedule_interval` error      | Removed in Airflow 3.x        | Use `schedule=` instead                      |
| `DummyOperator` not found      | Removed in Airflow 3.x        | Use `EmptyOperator`                          |
| `provide_context=True` error   | Removed in Airflow 3.x        | Remove it, use `**context`                   |
| DAG import timeout             | Heavy imports at top level    | Move imports inside functions                |
| Task stuck in `queued`         | No workers available          | Check executor, scale workers                |
| XCom data too large            | XCom has ~48KB limit          | Use S3/GCS, pass only file path via XCom     |
| `DagBag import timeout`        | Slow DAG file parsing         | Optimize top-level code                      |
| High CPU/Memory in Docker      | Too many example DAGs running | Set `LOAD_EXAMPLES=false`                    |
| Task fails randomly            | External API timeout          | Add `retries` and `retry_delay`              |
| Scheduler not picking DAG      | Wrong `dags_folder` path      | Check `airflow.cfg` path                     |

---

## 9️⃣ Questions & Answers

### Basic Level
**Q: What is a DAG?**
A: A Directed Acyclic Graph. A workflow definition with tasks and their dependencies. Directed = one direction, Acyclic = no loops.

**Q: What is XCom?**
A: Cross-Communication. A mechanism for tasks to share small data with each other via push and pull.

**Q: Difference between Operator and Sensor?**
A: Operator performs an action (run code, SQL, bash). Sensor waits for a condition to be true (file exists, API responds).

**Q: What is the role of the Scheduler?**
A: It reads DAG files, decides when to trigger DAG runs based on schedule, and submits tasks to the Executor.

**Q: What is catchup?**
A: If `catchup=True`, Airflow backfills all missed DAG runs since `start_date`. Set `catchup=False` to only run from now onwards.

### Intermediate Level
**Q: What happens if a task fails?**
A: It retries based on `retries` config, then marks as failed. Set `email_on_failure=True` to get notified automatically.

**Q: How do tasks communicate?**
A: Via XCom for small data. For large data, use shared storage like S3, GCS, or a database and pass only the path via XCom.

**Q: What is a Pool?**
A: A way to limit how many tasks run concurrently for a shared resource (e.g., only 5 DB queries at once to avoid overloading the DB).

**Q: What is trigger_rule?**
A: Controls when a task runs based on upstream task states.
- Default: `all_success`
- Others: `all_failed`, `one_success`, `none_failed_min_one_success`

### Advanced Level
**Q: Difference between Executors?**
A:
- **Local**: Single machine, simple setup
- **Celery**: Distributed with message queue (Redis/RabbitMQ), multiple workers
- **Kubernetes**: Each task runs in its own pod, best for isolation and scaling

**Q: How would you handle large data in XCom?**
A: Store data in S3/GCS and pass only the file path via XCom. Never push large datasets through XCom directly.

**Q: What is a TaskGroup vs SubDAG?**
A: SubDAGs were the old way to group tasks (now deprecated). TaskGroup is the modern, efficient way to visually group related tasks in the UI without performance overhead.

**Q: How do you optimize a slow DAG import?**
A: Move heavy imports (like pandas, spark) inside task functions instead of at the top of the DAG file. Top-level code runs every time the scheduler parses the file.

---

## 🔟 DAG Code Template (Airflow 3.x)

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.empty import EmptyOperator
from datetime import datetime, timedelta

# Default arguments applied to all tasks
default_args = {
    'owner': 'your_name',
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
    'email_on_failure': True,
    'email': ['alert@company.com'],
}

# Define the DAG
with DAG(
    dag_id='my_production_dag',
    default_args=default_args,
    description='My production pipeline',
    start_date=datetime(2024, 1, 1),
    schedule='@daily',              # or cron: '0 6 * * *'
    catchup=False,
    tags=['production', 'etl'],
) as dag:

    start  = EmptyOperator(task_id='start')

    process = PythonOperator(
        task_id='process',
        python_callable=my_function,
    )

    end    = EmptyOperator(task_id='end')

    # Define task order
    start >> process >> end
```

---

## 1️⃣1️⃣ Labs Completed

| Lab   | DAG Name        | Concept Learned                        | Status |
|-------|-----------------|----------------------------------------|--------|
| Lab 2 | `hello_world`   | DAG basics, operators, UI navigation   | ✅ Done |
| Lab 3 | `etl_pipeline`  | ETL pattern, XCom data passing         | ✅ Done |
| Lab 4 | `branching_dag` | Conditional logic, BranchOperator      | 🔄 In Progress |
| Lab 5 | `file_sensor`   | Sensors, event-driven workflows        | ⏳ Next |
| Lab 6 | Advanced        | Celery Executor, Kafka integration     | ⏳ Next |

---

## 1️⃣2️⃣ Airflow 2.x vs 3.x Key Differences

| Feature               | Airflow 2.x               | Airflow 3.x              |
|-----------------------|---------------------------|--------------------------|
| Schedule parameter    | `schedule_interval=`      | `schedule=`              |
| Dummy task            | `DummyOperator`           | `EmptyOperator`          |
| Context injection     | `provide_context=True`    | Automatic via `**context`|
| ZooKeeper             | Required for some setups  | KRaft mode (no ZooKeeper)|
| UI                    | Flask-based               | Modernized React UI      |

---

---

## 🧪 Building Airflow Labs (Step by Step)

---

### ⚙️ Setup: Run Airflow Locally with Docker

**Step 1: Download and start Airflow**
```bash
# Download official docker-compose file
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml'

# Create required folders
mkdir -p ./dags ./logs ./plugins ./config

# Set Airflow user ID
echo -e "AIRFLOW_UID=$(id -u)" > .env

# Initialize the database
docker compose up airflow-init

# Start Airflow
docker compose up -d
```

**Step 2: Open the UI**
- URL: `http://localhost:8080`
- Username: `airflow`
- Password: `airflow`

**Step 3: Disable example DAGs (saves CPU/memory)**

Add this to `docker-compose.yaml` under each service's `environment`:
```yaml
AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
```
Then restart:
```bash
docker compose down && docker compose up -d
```

---

### 🧪 Lab 2: Hello World DAG (Beginner)

**What you learn:** DAG structure, operators, UI navigation, task dependencies

**Create file:** `~/dags/hello_world.py`

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def greet():
    print("Hello, Airflow!")

def farewell():
    print("Goodbye, Airflow!")

with DAG(
    dag_id='hello_world',
    start_date=datetime(2024, 1, 1),
    schedule='@once',           # ✅ Airflow 3.x syntax
    catchup=False,
) as dag:

    task_greet   = PythonOperator(task_id='greet',   python_callable=greet)
    task_farewell = PythonOperator(task_id='farewell', python_callable=farewell)

    task_greet >> task_farewell
```

**Flow:**
```
[greet] → [farewell]
```

**How to run:**
```bash
# Check for errors
docker compose exec airflow-scheduler airflow dags list-import-errors

# Test individual task
docker compose exec airflow-scheduler airflow tasks test hello_world greet 2024-01-01
```
Then trigger from UI at `http://localhost:8080`

---

### 🧪 Lab 3: ETL Pipeline (Beginner+)

**What you learn:** ETL pattern, XCom (data passing between tasks)

**Create file:** `~/dags/etl_pipeline.py`

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime
import urllib.request
import json

def extract(**context):
    url = "https://jsonplaceholder.typicode.com/users"
    with urllib.request.urlopen(url) as response:
        data = json.loads(response.read().decode())
    context['ti'].xcom_push(key='raw_data', value=data)
    print(f"✅ Extracted {len(data)} records")

def transform(**context):
    raw = context['ti'].xcom_pull(key='raw_data', task_ids='extract')
    transformed = [
        {'id': u['id'], 'name': u['name'], 'email': u['email'], 'city': u['address']['city']}
        for u in raw
    ]
    context['ti'].xcom_push(key='clean_data', value=transformed)
    print(f"✅ Transformed {len(transformed)} records")

def load(**context):
    data = context['ti'].xcom_pull(key='clean_data', task_ids='transform')
    print(f"✅ Loading {len(data)} records to database...")
    for row in data:
        print(f"  --> ID: {row['id']} | Name: {row['name']} | Email: {row['email']} | City: {row['city']}")
    print("✅ Load complete!")

with DAG(
    dag_id='etl_pipeline',
    start_date=datetime(2024, 1, 1),
    schedule='@daily',          # ✅ Airflow 3.x syntax
    catchup=False,
    tags=['lab3', 'etl'],
) as dag:

    t1 = PythonOperator(task_id='extract',   python_callable=extract)
    t2 = PythonOperator(task_id='transform', python_callable=transform)
    t3 = PythonOperator(task_id='load',      python_callable=load)

    t1 >> t2 >> t3
```

**Flow:**
```
[extract from API] → [transform/clean data] → [load/print results]
```

**Key concept - XCom:**
```
extract  --pushes raw_data-->  XCom
transform --pulls raw_data-->  XCom --pushes clean_data--> XCom
load     --pulls clean_data--> XCom
```

View XCom data in UI: `Admin → XCom`

---

### 🧪 Lab 4: Branching DAG (Intermediate)

**What you learn:** Conditional logic, BranchPythonOperator, EmptyOperator, trigger_rule

**Create file:** `~/dags/Branching.py`

```python
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.empty import EmptyOperator    # ✅ Airflow 3.x (not DummyOperator)
from datetime import datetime
import random

def check_data_quality():
    score = random.randint(0, 100)
    print(f"📊 Data quality score: {score}")
    if score >= 70:
        print("✅ Quality is GOOD - processing data")
        return 'process_good_data'
    else:
        print("❌ Quality is BAD - sending alert")
        return 'handle_bad_data'

def process_good_data():
    print("✅ Processing good quality data...")
    print("✅ Data loaded to warehouse successfully!")

def handle_bad_data():
    print("🚨 Data quality too low!")
    print("🚨 Sending alert to data team...")

with DAG(
    dag_id='branching_dag',
    start_date=datetime(2024, 1, 1),
    schedule='@once',           # ✅ Airflow 3.x syntax
    catchup=False,
    tags=['lab4', 'branching'],
) as dag:

    start = EmptyOperator(task_id='start')

    branch = BranchPythonOperator(
        task_id='quality_check',
        python_callable=check_data_quality,
    )

    good = PythonOperator(task_id='process_good_data', python_callable=process_good_data)
    bad  = PythonOperator(task_id='handle_bad_data',   python_callable=handle_bad_data)

    end = EmptyOperator(
        task_id='end',
        trigger_rule='none_failed_min_one_success',  # runs after either branch
    )

    start >> branch >> [good, bad] >> end
```

**Flow:**
```
[start] → [quality_check] → Score ≥ 70: [process_good_data] → [end]
                          → Score < 70: [handle_bad_data]   → [end]
```

> 💡 Tip: Trigger this DAG multiple times to see both branches run!

---

### 🧪 Lab 5: File Sensor DAG (Intermediate)

**What you learn:** Sensors, event-driven workflows, waiting for conditions

**Create file:** `~/dags/file_sensor_dag.py`

```python
from airflow import DAG
from airflow.sensors.filesystem import FileSensor
from airflow.operators.python import PythonOperator
from datetime import datetime

def process_file():
    print("✅ File found! Starting processing...")
    print("✅ File processing complete!")

with DAG(
    dag_id='file_sensor_dag',
    start_date=datetime(2024, 1, 1),
    schedule='@once',
    catchup=False,
    tags=['lab5', 'sensor'],
) as dag:

    wait_for_file = FileSensor(
        task_id='wait_for_file',
        filepath='/opt/airflow/dags/data/input.csv',
        poke_interval=30,    # check every 30 seconds
        timeout=300,         # fail after 5 minutes
        mode='poke',
    )

    process = PythonOperator(
        task_id='process_file',
        python_callable=process_file,
    )

    wait_for_file >> process
```

**Test it:**
```bash
# Create the file to trigger the sensor
docker compose exec airflow-scheduler mkdir -p /opt/airflow/dags/data
docker compose exec airflow-scheduler touch /opt/airflow/dags/data/input.csv
```

**Flow:**
```
[wait for file] → (file appears!) → [process_file]
```

---

### 🧪 Lab 6: Multi-Broker Kafka + Airflow Integration (Advanced)

**What you learn:** Connecting Airflow with Kafka, triggering producers from a DAG

**Create file:** `~/dags/kafka_producer_dag.py`

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def produce_to_kafka():
    from confluent_kafka import Producer
    import json

    p = Producer({'bootstrap.servers': 'localhost:9092'})

    for i in range(10):
        message = json.dumps({'id': i, 'event': 'user_login', 'timestamp': str(datetime.now())})
        p.produce('user-events', key=str(i), value=message)
        print(f"✅ Sent message {i} to Kafka")

    p.flush()
    print("✅ All messages sent to Kafka!")

with DAG(
    dag_id='kafka_producer_dag',
    start_date=datetime(2024, 1, 1),
    schedule='@hourly',
    catchup=False,
    tags=['lab6', 'kafka'],
) as dag:

    send_events = PythonOperator(
        task_id='send_events_to_kafka',
        python_callable=produce_to_kafka,
    )
```

---

### 📊 Labs Progress Summary

| Lab | DAG Name         | Concepts Learned                              | Difficulty  | Status         |
|-----|------------------|-----------------------------------------------|-------------|----------------|
| 2   | `hello_world`    | DAG basics, operators, UI navigation          | 🟢 Beginner | ✅ Done         |
| 3   | `etl_pipeline`   | ETL pattern, XCom, data passing               | 🟢 Beginner | ✅ Done         |
| 4   | `branching_dag`  | Conditional logic, BranchOperator             | 🟡 Intermediate | 🔄 In Progress |
| 5   | `file_sensor_dag`| Sensors, event-driven workflows               | 🟡 Intermediate | ⏳ Next         |
| 6   | `kafka_producer` | Airflow + Kafka integration                   | 🔴 Advanced | ⏳ Next         |

---

### 🛠️ Common Lab Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `./dag.py: command not found` | Running Python file as shell script | Use `python3 dag.py` or trigger from UI |
| `schedule_interval` error | Removed in Airflow 3.x | Use `schedule=` instead |
| `DummyOperator` not found | Removed in Airflow 3.x | Use `EmptyOperator` |
| `provide_context=True` error | Removed in Airflow 3.x | Remove it, `**context` works automatically |
| DAG not showing in UI | Scheduler hasn't picked it up yet | Wait 30 seconds, refresh UI |
| `DagBag import timeout` | Airflow example DAGs slow to load | Set `LOAD_EXAMPLES=false` |
| High CPU in Docker | Too many containers running | Increase Docker memory or disable examples |

---

---

## 📖 From the Official Airflow 2.11.0 Documentation (Key Points)

> Source: https://airflow.apache.org/docs/apache-airflow/2.11.0/

---

### 🔷 A. Three Ways to Declare a DAG

The official docs show **3 different ways** to write a DAG:

**Way 1: Context Manager (most common)**
```python
with DAG("my_dag", start_date=datetime(2024, 1, 1)) as dag:
    task1 = BashOperator(task_id="task1", bash_command="echo hello")
```

**Way 2: Standard constructor**
```python
my_dag = DAG("my_dag", start_date=datetime(2024, 1, 1))
task1 = BashOperator(task_id="task1", bash_command="echo hello", dag=my_dag)
```

**Way 3: @dag decorator (TaskFlow API - modern style)**
```python
from airflow.decorators import dag, task

@dag(start_date=datetime(2024, 1, 1), schedule='@daily')
def my_dag():

    @task
    def extract():
        return {"data": "value"}

    @task
    def transform(data: dict):
        return data

    transform(extract())

my_dag()
```

> ✅ **Recommended**: Use the `@dag` decorator with TaskFlow API for new projects. It's cleaner and removes XCom boilerplate.

---

### 🔷 B. Task States (Official)

Every task instance goes through these states:

| State | Color | Meaning |
|---|---|---|
| `none` | White | Task not yet queued |
| `scheduled` | Tan | Scheduler has decided task should run |
| `queued` | Gray | Sent to executor, waiting for a worker |
| `running` | Green (lime) | Currently being executed by a worker |
| `success` | Green | Completed successfully |
| `failed` | Red | Task raised an exception |
| `skipped` | Pink/Purple | Task was skipped due to branching |
| `upstream_failed` | Orange | An upstream task failed |
| `up_for_retry` | Yellow | Task failed but will retry |
| `up_for_reschedule` | Light blue | Sensor waiting to re-check condition |
| `deferred` | Purple | Task deferred to a Trigger (async) |
| `removed` | Light gray | Task no longer exists in the DAG |

---

### 🔷 C. All Trigger Rules (Official)

Trigger rules control **when a task runs** based on upstream task states:

| Trigger Rule | When Task Runs |
|---|---|
| `all_success` (default) | All upstream tasks succeeded |
| `all_failed` | All upstream tasks failed |
| `all_done` | All upstream tasks finished (success or fail) |
| `all_skipped` | All upstream tasks were skipped |
| `one_success` | At least one upstream task succeeded |
| `one_failed` | At least one upstream task failed |
| `one_done` | At least one upstream task finished |
| `none_failed` | No upstream task failed (success or skip OK) |
| `none_skipped` | No upstream task was skipped |
| `none_failed_min_one_success` | None failed AND at least one succeeded |

```python
# Example: run even if some upstream tasks fail
end = EmptyOperator(
    task_id='end',
    trigger_rule='all_done'   # runs no matter what
)
```

---

### 🔷 D. Control Flow Features (Official)

#### 1. Depends on Past
A task will only run if the **same task in the previous DAG run** succeeded:
```python
task = PythonOperator(
    task_id='my_task',
    python_callable=my_fn,
    depends_on_past=True,   # won't run if yesterday's run failed
)
```

#### 2. Latest Only
Only runs the DAG for the **most recent scheduled interval**, skips all historical runs (useful to avoid backfilling):
```python
from airflow.operators.latest_only import LatestOnlyOperator

latest_only = LatestOnlyOperator(task_id='latest_only')
latest_only >> my_task   # my_task only runs for the latest interval
```

#### 3. Setup and Teardown (Airflow 2.7+)
Official pattern for infrastructure setup/cleanup:
```python
with DAG(...) as dag:
    create_cluster = create_cluster_task()     # setup
    run_query      = run_query_task()          # main work
    delete_cluster = delete_cluster_task()    # teardown

    create_cluster >> run_query >> delete_cluster
    create_cluster.as_setup()
    delete_cluster.as_teardown(setups=create_cluster)
```

#### 4. TaskGroups (Official way to group tasks)
```python
from airflow.utils.task_group import TaskGroup

with DAG(...) as dag:
    with TaskGroup("extract_group") as extract_group:
        task_a = PythonOperator(task_id='fetch_api', ...)
        task_b = PythonOperator(task_id='fetch_db',  ...)

    with TaskGroup("load_group") as load_group:
        task_c = PythonOperator(task_id='load_warehouse', ...)

    extract_group >> load_group
```

> In the UI, TaskGroups appear as collapsible boxes. Much better than SubDAGs.

---

### 🔷 E. Task Timeouts & SLAs (Official)

#### Task Timeout - fail if task takes too long:
```python
task = PythonOperator(
    task_id='slow_task',
    python_callable=my_fn,
    execution_timeout=timedelta(minutes=30),  # fail after 30 min
)
```

#### SLA (Service Level Agreement) - alert if task is late:
```python
task = PythonOperator(
    task_id='critical_task',
    python_callable=my_fn,
    sla=timedelta(hours=2),   # alert if not done within 2 hours of DAG start
)
```

---

### 🔷 F. Dynamic Task Mapping (Official - Airflow 2.3+)

Run the **same task multiple times with different inputs** dynamically:

```python
from airflow.decorators import dag, task

@dag(schedule='@daily', start_date=datetime(2024, 1, 1))
def dynamic_mapping():

    @task
    def get_files():
        return ['file1.csv', 'file2.csv', 'file3.csv']

    @task
    def process_file(filename: str):
        print(f"Processing {filename}")

    # Automatically creates one task instance per file
    process_file.expand(filename=get_files())

dynamic_mapping()
```

> This creates 3 parallel task instances automatically - one for each file!

---

### 🔷 G. Deferrable Operators (Official - Airflow 2.2+)

Regular sensors **block a worker slot** while waiting. Deferrable operators **free the worker** and use an async Trigger instead:

```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

# Regular sensor - holds a worker slot the whole time
wait = S3KeySensor(task_id='wait', bucket_key='s3://my-bucket/file.csv')

# Deferrable sensor - frees the worker while waiting
wait = S3KeySensor(
    task_id='wait',
    bucket_key='s3://my-bucket/file.csv',
    deferrable=True,    # ✅ uses Triggerer component, saves worker resources
)
```

> Use deferrable operators for long-waiting sensors to save infrastructure costs.

---

### 🔷 H. Data-aware Scheduling / Datasets (Official - Airflow 2.4+)

Trigger a DAG when **another DAG produces data**, not just on a time schedule:

```python
from airflow import Dataset

# Producer DAG - marks dataset as updated
my_dataset = Dataset("s3://my-bucket/data/output.csv")

with DAG("producer_dag", schedule='@daily') as dag:
    PythonOperator(
        task_id='produce_data',
        python_callable=produce_fn,
        outlets=[my_dataset],   # marks this dataset as updated when task completes
    )

# Consumer DAG - triggers when dataset is updated
with DAG("consumer_dag", schedule=[my_dataset]) as dag:  # triggered by dataset!
    PythonOperator(task_id='consume_data', python_callable=consume_fn)
```

> This is the modern alternative to `ExternalTaskSensor` for cross-DAG dependencies.

---

### 🔷 I. Best Practices (Official)

#### ✅ DO: Move imports inside task functions
```python
# ❌ BAD - runs every time scheduler parses the file (slow!)
import pandas as pd
import heavy_library

with DAG(...) as dag:
    task = PythonOperator(task_id='t', python_callable=my_fn)

# ✅ GOOD - import only when task actually runs
def my_fn():
    import pandas as pd       # imported only when task runs
    import heavy_library
    # ... do work
```

#### ✅ DO: Keep tasks idempotent
A task should produce the same result if run multiple times:
```python
# ❌ BAD - appends data every time, causes duplicates on retry
def load():
    db.execute("INSERT INTO table SELECT * FROM staging")

# ✅ GOOD - safe to run multiple times
def load():
    db.execute("INSERT INTO table SELECT * FROM staging WHERE NOT EXISTS ...")
    # OR: truncate then insert
    db.execute("DELETE FROM table WHERE date = '{{ ds }}'")
    db.execute("INSERT INTO table SELECT * FROM staging WHERE date = '{{ ds }}'")
```

#### ✅ DO: Use `execution_date` / `{{ ds }}` for date logic
```python
def extract(**context):
    run_date = context['ds']           # e.g., '2024-01-15'
    print(f"Processing data for {run_date}")
```

#### ✅ DO: Use Airflow Variables for config (not hardcoded values)
```python
from airflow.models import Variable

def my_fn():
    db_url = Variable.get("database_url")       # stored in Airflow UI
    api_key = Variable.get("api_key", default_var="fallback")
```

#### ✅ DO: Limit XCom to small data only
```python
# ❌ BAD - pushing large dataframe through XCom
context['ti'].xcom_push(key='data', value=large_dataframe)

# ✅ GOOD - push only the file path
context['ti'].xcom_push(key='data_path', value='s3://bucket/data.parquet')
```

#### ✅ DO: Use `.airflowignore` to skip files
Create a `.airflowignore` file in your dags folder to tell Airflow to skip certain files:
```
# .airflowignore
helper_functions.py    # utility file, not a DAG
test_*.py              # test files
__pycache__/
```

#### ✅ DO: Test DAGs before deploying
```bash
# Test that DAG loads without errors (DAG Loader Test)
python dags/my_dag.py

# Test a specific task (no DB writes)
airflow tasks test my_dag my_task_id 2024-01-01

# Check for import errors
airflow dags list-import-errors
```

---

### 🔷 J. Handling Complex Python Dependencies (Official)

When different tasks need **conflicting Python packages**:

**Option 1: PythonVirtualenvOperator** - creates a virtual env on the fly:
```python
from airflow.operators.python import PythonVirtualenvOperator

task = PythonVirtualenvOperator(
    task_id='ml_task',
    python_callable=train_model,
    requirements=['scikit-learn==1.3.0', 'numpy==1.24.0'],
    system_site_packages=False,
)
```

**Option 2: DockerOperator** - runs task in its own Docker container:
```python
from airflow.providers.docker.operators.docker import DockerOperator

task = DockerOperator(
    task_id='ml_task',
    image='my-ml-image:latest',
    command='python train.py',
    docker_url='unix://var/run/docker.sock',
)
```

**Option 3: KubernetesPodOperator** - runs task in its own Kubernetes pod:
```python
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

task = KubernetesPodOperator(
    task_id='ml_task',
    name='ml-pod',
    image='my-ml-image:latest',
    namespace='airflow',
)
```

---

### 🔷 K. Useful Jinja Templating Variables (Official)

Airflow passes these variables into task commands automatically:

| Template Variable | Value |
|---|---|
| `{{ ds }}` | Execution date as string: `2024-01-15` |
| `{{ ds_nodash }}` | Execution date no dashes: `20240115` |
| `{{ ts }}` | Timestamp: `2024-01-15T00:00:00+00:00` |
| `{{ dag.dag_id }}` | Name of the DAG |
| `{{ task.task_id }}` | Name of the current task |
| `{{ run_id }}` | Unique ID of the DAG run |
| `{{ prev_ds }}` | Previous execution date |
| `{{ next_ds }}` | Next execution date |
| `{{ params.my_param }}` | Access DAG params |

```python
# Example: use in BashOperator
task = BashOperator(
    task_id='dated_task',
    bash_command='echo "Processing date: {{ ds }}"',
)

# Example: use in SQL
task = PostgresOperator(
    task_id='load_daily',
    sql="SELECT * FROM orders WHERE date = '{{ ds }}'",
)
```

---

### 🔷 L. Official Documentation Sections (Reference Links)

| Section | URL | What it covers |
|---|---|---|
| Quick Start | `.../start.html` | Get Airflow running in minutes |
| Core Concepts | `.../core-concepts/` | DAGs, Tasks, Operators, Sensors, XCom |
| Best Practices | `.../best-practices.html` | How to write good DAGs |
| Authoring & Scheduling | `.../authoring-and-scheduling/` | Dynamic tasks, Datasets, Timetables |
| Administration | `.../administration-and-deployment/` | Scaling, security, config |
| CLI Reference | `.../cli-and-env-variables-ref.html` | All CLI commands |
| REST API | `.../stable-rest-api-ref.html` | Trigger DAGs via API |
| FAQ | `.../faq.html` | Common questions |
| Troubleshooting | `.../troubleshooting.html` | Debugging guide |

---

*Document prepared by: Dust AI Assistant*
*Date: July 2026*
*Based on: Apache Airflow 2.11.0 official documentation + Airflow 3.x*
*Reference: https://airflow.apache.org/docs/apache-airflow/2.11.0/*

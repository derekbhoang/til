# Celery

`Celery` is an open-source Python library used for distributed task queues and background job processing.

It helps you run time-consuming tasks asynchronously instead of blocking your main application.

## Core Idea

Instead of making your application do long-running tasks (sending emails, processing images, generating reports, calling APIs), Celery lets you:

1. Define a task
2. Queue tasks inside a message broker.
3. Have separate worker processes execute it in background.
4. Return results later if needed

## Core Components

### 1. Task

A task is the unit of work, executed asynchronously.

A task is just a Python function decorated with `@app.task`.

```python
from celery import Celery

app = Celery('demo')

@app.task
def add(x, y):
    return x + y
```

- Defined using @app.task
- Can be queued and executed later
- Supports retries, scheduling, chaining, etc.

### 2. Producer / Client

The producer is the application code (below) that sends tasks to the broker.

```python
add.delay(13, 9)
```

The producer:

- Serializes the task
- Sends a message to the message broker
- Does not execute the task itself

```json
{
    "task": "tasks.add",
    "id": "10100011101",
    "args": [13, 9],
    "kwargs": {}
}
```

### 3. Message Broker

The broker transports messages/tasks between producers and workers.

Common brokers:

- Redis
- RabbitMQ
- Amazon SQS

The broker:

- Stores queued tasks
- Delivers tasks to workers
- Handles routing and acknowledgments
- Waits until the workers consume them.

Flow:

```text
Producer -> Broker -> Worker
```

RabbitMQ is often preferred for reliability.

Redis is popular for simplicity.

### 4. Worker

A worker is a process that consumes tasks from queues and executes them.

Start a worker:

```bash
celery -A tasks worker --loglevel=info
```

Workers responsibilities:

- Pull tasks from broker
- Execute task functions
- Handle retries/failures
- Report results

Workers support:

- Concurrency
- Autoscaling
- Prefetching
- Task acknowledgment

### 5. Result Backend

The result backend stores task states and results.

Supported backends include:

- Redis
- Database (PostgreSQL/MySQL)
- Memcached
- RPC

Example:

```python
app = Celery(
    'tasks',
    broker='redis://localhost:6379/0',
    backend='redis://localhost:6379/1'
)
```

Retrieve result:

```python
result = add.delay(2, 3)
print(result.get())
```

Task states include:

- PENDING
- STARTED
- SUCCESS
- FAILURE
- RETRY
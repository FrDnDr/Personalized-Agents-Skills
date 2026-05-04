---
name: background-jobs
description: Celery and Redis setup for background tasks including idempotency rules, retry logic, and Celery Beat scheduling.
---
# background-jobs.md

## Purpose

Define conventions for queue setup, background workers, scheduled tasks,
and retry logic for async operations that run outside the request/response cycle.

---

## Conventions

### When to Use Background Jobs

Use a background job when:

- Task takes longer than 500ms (email sending, file processing, report generation)
- Task should not block the HTTP response
- Task needs to be retried on failure
- Task needs to run on a schedule (cron-like)

Do NOT use background jobs for:

- Simple DB writes that are fast and must be synchronous
- Tasks where the user needs the result immediately in the same request

### Library Stack

- **Celery** — primary task queue for Python (with Redis or RabbitMQ as broker)
- **Redis** — default broker and result backend
- **Celery Beat** — for scheduled/periodic tasks
- **Flower** — for task monitoring dashboard (development)

### Folder Structure

```
/app
  /workers
    celery_app.py       # Celery app instance and config
    /tasks
      email_tasks.py    # Email-related tasks
      report_tasks.py   # Report generation tasks
      sync_tasks.py     # Data sync tasks
    /schedules
      beat_schedule.py  # Periodic task schedule definitions
```

### Task Conventions

- Each task file groups related tasks by domain
- Always use `@celery_app.task(bind=True)` for tasks that need retry logic
- Always set explicit `name` on tasks — never rely on auto-naming
- Always define `max_retries` and `default_retry_delay`
- Tasks must be idempotent — running the same task twice must not cause side effects

```python
# workers/tasks/email_tasks.py
from app.workers.celery_app import celery_app
from app.services.email import send_email_service

@celery_app.task(
    bind=True,
    name="tasks.send_welcome_email",
    max_retries=3,
    default_retry_delay=60,
)
def send_welcome_email(self, user_id: str):
    try:
        send_email_service(user_id)
    except Exception as exc:
        raise self.retry(exc=exc)
```

### Celery App Config

```python
# workers/celery_app.py
from celery import Celery
from app.core.config import settings

celery_app = Celery(
    "worker",
    broker=settings.REDIS_URL,
    backend=settings.REDIS_URL,
    include=["app.workers.tasks.email_tasks", "app.workers.tasks.report_tasks"],
)

celery_app.conf.update(
    task_serializer="json",
    result_serializer="json",
    accept_content=["json"],
    timezone="UTC",
    enable_utc=True,
    task_track_started=True,
)
```

### Retry Logic

- Always retry on transient failures (network errors, timeouts, rate limits)
- Never retry on permanent failures (invalid input, 4xx errors from external APIs)
- Use exponential backoff for retries: `countdown = 2 ** self.request.retries * 30`
- Always log the failure reason before retrying

### Scheduled Tasks (Celery Beat)

```python
# workers/schedules/beat_schedule.py
from celery.schedules import crontab

CELERYBEAT_SCHEDULE = {
    "sync-orders-every-hour": {
        "task": "tasks.sync_orders",
        "schedule": crontab(minute=0),     # Every hour on the hour
    },
    "daily-report": {
        "task": "tasks.generate_daily_report",
        "schedule": crontab(hour=6, minute=0),  # 6 AM UTC daily
    },
}
```

### Observability

- Always log task start, success, and failure with task ID and input summary
- Never log sensitive data (passwords, tokens, PII) in task logs
- Monitor task queue depth — alert if queue exceeds threshold
- Track task execution time — alert on tasks exceeding expected duration

---

## Anti-Patterns

- Never run long operations synchronously in a FastAPI endpoint
- Never make tasks non-idempotent — always design for at-least-once execution
- Never hardcode broker URLs — use config/env vars
- Never skip retry logic for tasks that call external services
- Never log sensitive input data inside task functions
- Never use in-process background tasks (`BackgroundTasks` in FastAPI) for anything that must survive a server restart

---

## Ready-to-Use Prompt

```
Task: Implement background job for [task description]
Skill: backend/background-jobs, backend/rest-api, core/observability

REQUIREMENTS:
- Task type: [one-off triggered / scheduled periodic]
- Trigger: [API endpoint / event / schedule — specify cron if periodic]
- Retry needed: [yes/no — describe failure scenarios]
- External services called: [list APIs, DBs, email providers, etc.]

CONSTRAINTS:
- Use Celery with Redis broker
- Tasks must be idempotent
- Set max_retries and default_retry_delay on all retryable tasks
- No sensitive data in task logs
- Broker URL from config — never hardcoded

DONE WHEN:
- Task executes correctly when triggered
- Retry logic handles transient failures
- Task logs start, success, and failure with task ID
- Scheduled tasks registered in beat_schedule if periodic
- Code review skill applied before commit
```


# celery
## Celery & Task Scheduling in Django  🚀

## Introduction

Celery is an asynchronous task queue that helps Django applications handle background tasks efficiently. It allows you to run long-running or scheduled tasks without blocking the main application.

## Why Use Celery?

✅ Background Task Processing

Offload time-consuming tasks (e.g., sending notifications, processing files) from the main app.

Prevent slow response times by running tasks in the background.

✅ Periodic Task Scheduling

Run scheduled tasks like database cleanup, backups, and notification reminders.

Use django-celery-beat to manage periodic tasks from the Django Admin panel.

## Components

1. Celery

The core task queue framework that allows Django to run asynchronous jobs.

2. Redis (Message Broker)

Acts as a middleman between Django and Celery, queuing tasks until they are processed.

Alternative: RabbitMQ (for more reliability, but harder to set up).

3. Celery Workers

The background processes that execute Celery tasks.

4. Django-Celery-Beat (For Scheduling Tasks)

Enables Django Admin integration to create and manage periodic tasks.

5. Django-Celery-Results (For Tracking Task Execution)

Stores task results in the database for monitoring and debugging.

## Installation

Install the required dependencies:

```bash
pip install celery redis django-celery-beat django-celery-results
```

## Configuration

1. Set Up Celery in Django

 Inside Django project, create a celery.py file next to settings.py:

```bash
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_project.settings')

app = Celery('your_project')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

Add Celery settings to settings.py:

CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'

```

2. Define a Celery Task

Inside  Django app  create a tasks.py file:

```bash
from celery import shared_task

@shared_task
def send_admin_notification(message):
    print(f'Admin Notification: {message}')
```

3. Running Celery Worker

Start the Redis server:
```bash
redis-server
```
Run the Celery worker:
```bash
celery -A your_project worker --loglevel=info

Scheduling Periodic Tasks (Using django-celery-beat)
```

1. Apply Migrations
```bash
python manage.py migrate django_celery_beat
```

2. Add Celery Beat to Installed Apps
```bash
INSTALLED_APPS = [
    ...
    'django_celery_beat',
]
```
3. Start Celery Beat Service
```bash

celery -A your_project beat --loglevel=info
```

4. Create a Periodic Task

Go to Django Admin → Periodic Tasks → Add a new periodic task.

Example: Notification Before Expiry Date

Assume we have a Policy model with expiry_date. We need to notify admins 3 days before a policy expires.

```bash
tasks.py:

from datetime import timedelta
from django.utils import timezone
from celery import shared_task
from your_app.models import Policy

@shared_task
def notify_admin_about_expiring_policies():
    three_days_from_now = timezone.now() + timedelta(days=3)
    expiring_policies = Policy.objects.filter(expiry_date__lte=three_days_from_now)
    for policy in expiring_policies:
        print(f'Policy {policy.name} is expiring soon!')
```

Schedule this task to run daily using django-celery-beat.

## Running Everything Together

Start Redis:
```bash
redis-server
```
Run Celery Worker:
```bash
celery -A your_project worker --loglevel=info
```
Run Celery Beat:
```bash
celery -A your_project beat --loglevel=info
```
Django Server :
```bash
python manage.py runserver
```

Now, Celery will run background tasks, and periodic tasks will execute based on schedule

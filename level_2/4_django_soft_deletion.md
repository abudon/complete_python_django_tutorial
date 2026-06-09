# Django Soft Deletion - Complete Implementation Guide

## What is Soft Deletion?

**Soft deletion** is a database design pattern where records are marked as "deleted" instead of being permanently removed from the database.

### Why Use Soft Deletion?

#### ✅ Benefits
- **Data Recovery**: Deleted items can be restored
- **Audit Trail**: Maintains historical records
- **Data Integrity**: Prevents orphaned references in related tables
- **Analytics**: Deleted data still available for reporting
- **Legal Compliance**: Required data retention in some industries

#### ❌ Hard Deletion Problems
```python
# Hard deletion - data is GONE forever
Task.objects.filter(id=1).delete()  # ❌ Permanent loss
```

#### ✅ Soft Deletion Solution
```python
# Soft deletion - data is marked as deleted
Task.objects.filter(id=1).update(deleted=True)  # ✅ Recoverable
```

---

## Implementation Steps

### Step 1: Add Deleted Field to Model

Add a `deleted` boolean field to your model:

```python
# models.py
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)
    deleted = models.BooleanField(default=False)  # Add this field
    created_date = models.DateTimeField(auto_now_add=True)
    updated_date = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title

    # Soft delete method (optional)
    def soft_delete(self):
        """Mark the task as deleted"""
        self.deleted = True
        self.save()

    # Restore method (optional)
    def restore(self):
        """Restore a soft-deleted task"""
        self.deleted = False
        self.save()
```

### Step 2: Create and Run Migration

Generate migration for the new field:

```bash
python manage.py makemigrations
```

**Output:**
```
Migrations for 'tasks':
  tasks/migrations/0003_task_deleted.py
    - Add field deleted to task
```

Apply the migration:

```bash
python manage.py migrate
```

**Output:**
```
Operations to perform:
  Apply all migrations: tasks
Running migrations:
  Applying tasks.0003_task_deleted... OK
```

---

## Update Views for Soft Deletion

### Modify Delete View

**Before (hard deletion):**
```python
def delete_tasks_view(request, index):
    Task.objects.filter(id=index).delete()  # ❌ Permanent deletion
    return HttpResponseRedirect("/tasks")
```

**After (soft deletion):**
```python
def delete_tasks_view(request, index):
    Task.objects.filter(id=index).update(deleted=True)  # ✅ Soft deletion
    return HttpResponseRedirect("/tasks")
```

**Alternative using model method:**
```python
def delete_tasks_view(request, index):
    try:
        task = Task.objects.get(id=index)
        task.soft_delete()  # Using custom method
        return HttpResponseRedirect("/tasks")
    except Task.DoesNotExist:
        # Handle case where task doesn't exist
        return HttpResponseRedirect("/tasks")
```

### Modify List View

**Before (shows all tasks):**
```python
def tasks_view(request):
    all_tasks = Task.objects.all()  # ❌ Shows deleted tasks
    # ... rest of view
```

**After (excludes deleted tasks):**
```python
def tasks_view(request):
    all_tasks = Task.objects.filter(deleted=False)  # ✅ Excludes deleted
    # ... rest of view
```

### Complete Updated Views

```python
# views.py
from django.shortcuts import render, HttpResponseRedirect, get_object_or_404
from .models import Task

def tasks_view(request):
    # Only show non-deleted tasks
    all_tasks = Task.objects.filter(deleted=False)
    
    # Search functionality
    search_term = request.GET.get("searchterm")
    if search_term:
        all_tasks = all_tasks.filter(title__icontains=search_term)
    
    return render(request, "tasks.html", {"tasks": all_tasks})

def add_tasks_view(request):
    new_task = request.GET.get("task")
    if new_task:
        Task.objects.create(title=new_task)
    return HttpResponseRedirect("/tasks")

def delete_tasks_view(request, index):
    # Soft delete the task
    Task.objects.filter(id=index).update(deleted=True)
    return HttpResponseRedirect("/tasks")

# BONUS: Restore functionality
def restore_task_view(request, index):
    """Restore a soft-deleted task"""
    Task.objects.filter(id=index).update(deleted=False)
    return HttpResponseRedirect("/tasks")
```

---

## Advanced Soft Deletion Features

### Custom Manager for Automatic Filtering

Create a custom manager that automatically excludes deleted records:

```python
# models.py
class TaskManager(models.Manager):
    """Custom manager that excludes soft-deleted tasks by default"""
    
    def get_queryset(self):
        return super().get_queryset().filter(deleted=False)
    
    def all_with_deleted(self):
        """Get all tasks including deleted ones"""
        return super().get_queryset()
    
    def deleted_only(self):
        """Get only deleted tasks"""
        return super().get_queryset().filter(deleted=True)

class Task(models.Model):
    # ... fields ...
    
    objects = TaskManager()  # Use custom manager
    
    # Alternative managers for accessing deleted items
    all_objects = models.Manager()  # Access to all records
    
    def __str__(self):
        return self.title
```

**Usage:**
```python
# Only active tasks
active_tasks = Task.objects.all()

# All tasks including deleted
all_tasks = Task.all_objects.all()

# Only deleted tasks
deleted_tasks = Task.all_objects.filter(deleted=True)
```

### Admin Interface Integration

Make soft deletion work in Django admin:

```python
# admin.py
from django.contrib import admin
from .models import Task

@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    list_display = ['title', 'completed', 'deleted', 'created_date']
    list_filter = ['completed', 'deleted', 'created_date']
    actions = ['soft_delete', 'restore']
    
    def soft_delete(self, request, queryset):
        """Admin action for soft deletion"""
        queryset.update(deleted=True)
        self.message_user(request, f"Soft deleted {queryset.count()} tasks")
    soft_delete.short_description = "Soft delete selected tasks"
    
    def restore(self, request, queryset):
        """Admin action for restoration"""
        queryset.update(deleted=False)
        self.message_user(request, f"Restored {queryset.count()} tasks")
    restore.short_description = "Restore selected tasks"
    
    # Exclude deleted tasks from admin by default
    def get_queryset(self, request):
        return super().get_queryset(request).filter(deleted=False)
```

### Template Updates

Update templates to work with soft deletion:

```html
<!-- tasks.html -->
<h1>My Tasks</h1>

<!-- Search Form -->
<form method="GET">
    <input type="text" name="searchterm" placeholder="Search tasks...">
    <button type="submit">Search</button>
</form>

<!-- Add Task Form -->
<form method="GET" action="/add-task/">
    <input type="text" name="task" placeholder="New task..." required>
    <button type="submit">Add Task</button>
</form>

<!-- Tasks List -->
{% for task in tasks %}
<div class="task-item">
    <h2>
        {{ forloop.counter }}. {{ task.title }}
        {% if task.completed %}
            <span class="completed">✓</span>
        {% endif %}
    </h2>
    <p>{{ task.description|default:"No description" }}</p>
    <small>Created: {{ task.created_date|date:"M d, Y H:i" }}</small>
    
    <div class="actions">
        <a href="/delete-task/{{ task.id }}/" onclick="return confirm('Soft delete this task?')">Delete</a>
        {% if task.completed %}
            <a href="/mark-incomplete/{{ task.id }}/">Mark Incomplete</a>
        {% else %}
            <a href="/mark-complete/{{ task.id }}/">Mark Complete</a>
        {% endif %}
    </div>
</div>
{% endfor %}

<!-- BONUS: Show deleted tasks count -->
<p class="deleted-count">
    {{ deleted_count }} tasks in trash
    <a href="/deleted-tasks/">View Trash</a>
</p>
```

---

## Complete Implementation Example

### models.py
```python
from django.db import models

class TaskManager(models.Manager):
    """Manager that excludes soft-deleted tasks"""
    def get_queryset(self):
        return super().get_queryset().filter(deleted=False)

class Task(models.Model):
    title = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)
    deleted = models.BooleanField(default=False)
    created_date = models.DateTimeField(auto_now_add=True)
    updated_date = models.DateTimeField(auto_now=True)
    
    # Use custom manager
    objects = TaskManager()
    all_objects = models.Manager()  # Access to all records
    
    def __str__(self):
        return self.title
    
    def soft_delete(self):
        """Soft delete this task"""
        self.deleted = True
        self.save()
    
    def restore(self):
        """Restore this task"""
        self.deleted = False
        self.save()
    
    class Meta:
        ordering = ['-created_date']
```

### views.py
```python
from django.shortcuts import render, HttpResponseRedirect, get_object_or_404
from .models import Task

def tasks_view(request):
    """List all active (non-deleted) tasks"""
    all_tasks = Task.objects.all()
    
    # Search functionality
    search_term = request.GET.get("searchterm")
    if search_term:
        all_tasks = all_tasks.filter(title__icontains=search_term)
    
    # Get deleted count for display
    deleted_count = Task.all_objects.filter(deleted=True).count()
    
    return render(request, "tasks.html", {
        "tasks": all_tasks,
        "deleted_count": deleted_count
    })

def deleted_tasks_view(request):
    """List all deleted tasks (trash)"""
    deleted_tasks = Task.all_objects.filter(deleted=True)
    return render(request, "deleted_tasks.html", {"tasks": deleted_tasks})

def add_tasks_view(request):
    """Add a new task"""
    title = request.GET.get("task")
    description = request.GET.get("description", "")
    
    if title:
        Task.objects.create(title=title, description=description)
    
    return HttpResponseRedirect("/tasks")

def delete_tasks_view(request, index):
    """Soft delete a task"""
    Task.all_objects.filter(id=index).update(deleted=True)
    return HttpResponseRedirect("/tasks")

def restore_task_view(request, index):
    """Restore a soft-deleted task"""
    Task.all_objects.filter(id=index).update(deleted=False)
    return HttpResponseRedirect("/deleted-tasks")

def toggle_complete_view(request, index):
    """Toggle task completion status"""
    task = get_object_or_404(Task, id=index)
    task.completed = not task.completed
    task.save()
    return HttpResponseRedirect("/tasks")
```

### urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('tasks/', views.tasks_view, name='tasks'),
    path('deleted-tasks/', views.deleted_tasks_view, name='deleted_tasks'),
    path('add-task/', views.add_tasks_view, name='add_task'),
    path('delete-task/<int:index>/', views.delete_tasks_view, name='delete_task'),
    path('restore-task/<int:index>/', views.restore_task_view, name='restore_task'),
    path('toggle-complete/<int:index>/', views.toggle_complete_view, name='toggle_complete'),
]
```

---

## Best Practices

### 1. Always Use Soft Deletion for User Data
```python
# ✅ Good - recoverable
Task.objects.filter(id=id).update(deleted=True)

# ❌ Bad - permanent loss
Task.objects.filter(id=id).delete()
```

### 2. Add Deleted Field to All User-Facing Models
```python
class Article(models.Model):
    # ... fields ...
    deleted = models.BooleanField(default=False)

class Comment(models.Model):
    # ... fields ...
    deleted = models.BooleanField(default=False)
```

### 3. Use Custom Managers
```python
# Default manager excludes deleted items
active_articles = Article.objects.all()

# Access deleted items when needed
all_articles = Article.all_objects.all()
```

### 4. Implement Restore Functionality
Always provide a way for users to recover accidentally deleted items.

### 5. Clean Up Old Data Periodically
```python
# Delete items deleted more than 30 days ago
from datetime import timedelta
from django.utils import timezone

cutoff_date = timezone.now() - timedelta(days=30)
Task.all_objects.filter(
    deleted=True, 
    updated_date__lt=cutoff_date
).delete()  # Hard delete old soft-deleted items
```

---

## Security Considerations

### Protect Deleted Data Access
```python
# views.py - Only allow owners to see their deleted tasks
@login_required
def deleted_tasks_view(request):
    deleted_tasks = Task.all_objects.filter(
        deleted=True, 
        user=request.user  # Assuming user field exists
    )
    return render(request, "deleted_tasks.html", {"tasks": deleted_tasks})
```

### Prevent Access to Deleted Records
```python
# Middleware or decorator to prevent access to deleted items
def require_not_deleted(view_func):
    def wrapper(request, *args, **kwargs):
        task_id = kwargs.get('index')
        task = get_object_or_404(Task, id=task_id)
        if task.deleted:
            raise Http404("Task not found")
        return view_func(request, *args, **kwargs)
    return wrapper
```

---

## Key Takeaways

1. **Soft deletion preserves data** while appearing deleted to users
2. **Add `deleted` boolean field** to models requiring soft deletion
3. **Filter queries** with `deleted=False` to exclude soft-deleted items
4. **Use `update(deleted=True)`** instead of `delete()` for soft deletion
5. **Custom managers** can automatically exclude deleted records
6. **Provide restore functionality** for better user experience
7. **Clean up old data** periodically to manage database size

---

## Testing Soft Deletion

### Unit Tests
```python
# tests.py
from django.test import TestCase
from .models import Task

class TaskModelTest(TestCase):
    def setUp(self):
        self.task = Task.objects.create(title="Test Task")
    
    def test_soft_delete(self):
        """Test that soft delete works"""
        self.task.soft_delete()
        self.assertTrue(self.task.deleted)
        
        # Should not appear in default queryset
        self.assertEqual(Task.objects.count(), 0)
        
        # Should appear in all_objects
        self.assertEqual(Task.all_objects.count(), 1)
    
    def test_restore(self):
        """Test that restore works"""
        self.task.soft_delete()
        self.task.restore()
        self.assertFalse(self.task.deleted)
        self.assertEqual(Task.objects.count(), 1)
```

---

**Soft deletion makes your application more robust and user-friendly by preserving data integrity and providing recovery options!** 🎉

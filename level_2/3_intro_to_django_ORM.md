# Django ORM - Building a Tasks Management App

## Introduction to Django ORM

Now that we have a **Model** and its **schema updated** in the database, we can start learning the **ORM** (Object-Relational Mapping).

The Django ORM is a powerful feature that lets you interact with the database using Python objects instead of raw SQL. No complex queries or database languages required!

### What We'll Build

We'll enhance our Tasks management app to use the Django ORM for:
- ✅ Creating and saving tasks to the database
- ✅ Retrieving tasks from the database
- ✅ Searching and filtering tasks
- ✅ Deleting tasks using primary keys

---

## Step 1: Organize Your Code

### Move Views to Proper Location

Currently, our views are in `urls.py`. **Best practice**: Keep views in `views.py` within your app.

**Before (urls.py):**
```python
# This is wrong - views don't belong in urls.py
def tasks_view(request):
    # view logic here...
```

**After: Move to views.py**
```python
# views.py - This is the correct location
def tasks_view(request):
    # view logic here...
```

**Then import in urls.py:**
```python
# urls.py
from . import views  # Import views from current app

urlpatterns = [
    path('tasks/', views.tasks_view, name='tasks'),
    # ... other URL patterns
]
```

### Test the Application

Start your server and verify everything works as before:

```bash
python manage.py runserver
```

---

## Step 2: Saving Data with ORM

### Import the Model

At the top of `views.py`, import your Task model:

```python
from .models import Task
```

### Modify Create Task View

**Before (using global variable):**
```python
def add_tasks_view(request):
    new_task = request.GET.get("task")
    tasks.append(new_task)  # Global variable approach
    return HttpResponseRedirect("/tasks")
```

**After (using ORM):**
```python
def add_tasks_view(request):
    new_task = request.GET.get("task")
    Task(title=new_task).save()  # ORM approach
    return HttpResponseRedirect("/tasks")
```

### How It Works

1. **Create object**: `Task(title=new_task)` creates a Task instance
2. **Save to database**: `.save()` method generates SQL and inserts data
3. **Automatic validation**: Django checks data integrity before saving

**That's it!** One line change and you're now using a database.

---

## Step 3: Retrieving Data with ORM

### Modify Tasks List View

**Before (global variable):**
```python
def tasks_view(request):
    return render(request, "tasks.html", {"tasks": tasks})
```

**After (ORM Query):**
```python
def tasks_view(request):
    all_tasks = Task.objects.all()
    return render(request, "tasks.html", {"tasks": all_tasks})
```

### Understanding QuerySets

The `Task.objects.all()` returns a **QuerySet**:

- **QuerySet**: Iterable object containing database results
- **Lazy evaluation**: Query only executes when needed
- **Chainable**: Can add filters, ordering, etc.

### The __str__ Method

When you render objects in templates, Django calls the `__str__` method. Without it, you see `Task object (1)`.

**Add to your Task model:**
```python
# models.py
class Task(models.Model):
    title = models.CharField(max_length=200)
    created_date = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
```

**Now templates display task titles instead of `Task object (1)`.**

---

## Step 4: Template Rendering Options

### Option 1: Using __str__ Method

```html
<!-- tasks.html -->
{% for task in tasks %}
<h2>
  {{ forloop.counter }} {{ task }} - {{ task.created_date }}
  <a href="delete-tasks/{{ forloop.counter }}">delete</a>
</h2>
{% endfor %}
```

### Option 2: Explicit Attribute Access

```html
<!-- tasks.html -->
{% for task in tasks %}
<h2>
  {{ forloop.counter }} {{ task.title }} - {{ task.created_date }}
  <a href="delete-tasks/{{ task.id }}">delete</a>
</h2>
{% endfor %}
```

**Recommendation**: Use explicit attributes for clarity and to access multiple fields.

---

## Step 5: Search Functionality

### Add Search Form to Template

```html
<!-- tasks.html -->
<form method="GET" action="">
    <input type="text" name="searchterm" placeholder="Search tasks...">
    <button type="submit">Search</button>
</form>

<!-- Display results -->
{% for task in tasks %}
<h2>{{ forloop.counter }} {{ task.title }} - {{ task.created_date }}</h2>
{% endfor %}
```

### Implement Search in View

```python
def tasks_view(request):
    all_tasks = Task.objects.all()
    
    # Handle search
    search_term = request.GET.get("searchterm")
    if search_term:
        all_tasks = all_tasks.filter(title__icontains=search_term)
    
    return render(request, "tasks.html", {"tasks": all_tasks})
```

### Search Options

| Method | Description | Case Sensitive |
|--------|-------------|----------------|
| `title=search_term` | Exact match | Yes |
| `title__contains=search_term` | Contains substring | Yes |
| `title__icontains=search_term` | Contains substring | **No** (recommended) |

---

## Step 6: Delete with Primary Keys

### Why Primary Keys?

**Problem with loop indices:**
- Loop counter (1, 2, 3...) changes when tasks are reordered
- Not reliable for database operations
- Can cause wrong deletions

**Solution: Use primary keys (IDs)**
- Every Django model has an auto-generated `id` field
- Unique identifier for each database record
- Never changes

### Update Template Links

```html
{% for task in tasks %}
<h2>
  {{ forloop.counter }} {{ task.title }} - {{ task.created_date }}
  <a href="delete-tasks/{{ task.id }}">delete</a>  <!-- Use task.id -->
</h2>
{% endfor %}
```

### Update Delete View

```python
def delete_tasks_view(request, index):
    Task.objects.filter(id=index).delete()
    return HttpResponseRedirect("/tasks")
```

### How Delete Works

1. **Filter**: `Task.objects.filter(id=index)` finds the specific task
2. **Delete**: `.delete()` removes it from database
3. **Redirect**: Return to tasks list

---

## Complete Code Examples

### models.py
```python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=200)
    created_date = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
```

### views.py
```python
from django.shortcuts import render, HttpResponseRedirect
from .models import Task

def tasks_view(request):
    all_tasks = Task.objects.all()
    
    # Search functionality
    search_term = request.GET.get("searchterm")
    if search_term:
        all_tasks = all_tasks.filter(title__icontains=search_term)
    
    return render(request, "tasks.html", {"tasks": all_tasks})

def add_tasks_view(request):
    new_task = request.GET.get("task")
    if new_task:
        Task(title=new_task).save()
    return HttpResponseRedirect("/tasks")

def delete_tasks_view(request, index):
    Task.objects.filter(id=index).delete()
    return HttpResponseRedirect("/tasks")
```

### urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('tasks/', views.tasks_view, name='tasks'),
    path('add-task/', views.add_tasks_view, name='add_task'),
    path('delete-tasks/<int:index>/', views.delete_tasks_view, name='delete_task'),
]
```

### tasks.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>Tasks</title>
</head>
<body>
    <h1>My Tasks</h1>
    
    <!-- Search Form -->
    <form method="GET" action="">
        <input type="text" name="searchterm" placeholder="Search tasks...">
        <button type="submit">Search</button>
    </form>
    
    <!-- Add Task Form -->
    <form method="GET" action="/add-task/">
        <input type="text" name="task" placeholder="New task...">
        <button type="submit">Add Task</button>
    </form>
    
    <!-- Tasks List -->
    {% for task in tasks %}
    <h2>
        {{ forloop.counter }}. {{ task.title }} 
        - {{ task.created_date|date:"M d, Y H:i" }}
        <a href="/delete-tasks/{{ task.id }}/">[delete]</a>
    </h2>
    {% endfor %}
</body>
</html>
```

---

## Key ORM Concepts Learned

1. **Model Managers**: `Task.objects` - interface to database queries
2. **Creating Objects**: `Task(title="value")` then `.save()`
3. **QuerySets**: Lazy, iterable results from `objects.all()`
4. **Filtering**: `filter(field__lookup=value)`
5. **String Representation**: `__str__` method for display
6. **Primary Keys**: Use `id` field for unique identification
7. **Delete Operations**: `filter().delete()` for safe deletion

---

## Next Steps

Now that you understand basic ORM operations, you can explore:
- **Advanced Queries**: `exclude()`, `order_by()`, `Q` objects
- **Relationships**: Foreign keys, many-to-many fields
- **Aggregations**: `count()`, `sum()`, `avg()`
- **Raw SQL**: When ORM isn't enough

Your tasks app now uses a real database! 🎉

---

**Remember**: The ORM abstracts database complexity while maintaining performance and security.

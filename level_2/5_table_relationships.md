# Django Table Relationships - Complete Guide

## Why Relationships Matter

### The Problem with Single Tables

Imagine building a task management app where users can create tasks. Your first instinct might be to add user information directly to the Task table:

```python
# ❌ Bad approach - data duplication
class Task(models.Model):
    title = models.CharField(max_length=100)
    user_email = models.EmailField()  # Duplicate user data
    user_name = models.CharField(max_length=100)  # More duplication
    completed = models.BooleanField(default=False)
```

**Problems:**
- **Data duplication** - User info repeated in every task
- **Inconsistency** - What if user changes email?
- **Maintenance nightmare** - Update user info in multiple places
- **Deletion issues** - What happens to tasks when user is deleted?

### The Relational Solution

**Separate tables with relationships:**

```python
# ✅ Good approach - normalized design
class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)

class Task(models.Model):
    title = models.CharField(max_length=100)
    user = models.ForeignKey(User, on_delete=models.CASCADE)  # Relationship!
    completed = models.BooleanField(default=False)
```

**Benefits:**
- **Single source of truth** - User data stored once
- **Data integrity** - Relationships maintain consistency
- **Flexibility** - Easy to add user fields without affecting tasks
- **Performance** - Efficient queries with joins

---

## Understanding Database Relationships

### Primary Keys and Foreign Keys

#### Primary Key (PK)
- **Unique identifier** for each record
- Django automatically adds `id` field as primary key
- Ensures no duplicate records

#### Foreign Key (FK)
- **Reference to another table's primary key**
- Creates the "relationship" between tables
- Enables data linking without duplication

### Example Data Structure

**Users Table:**
| id | name          | email               |
|----|---------------|---------------------|
| 1  | Vignesh Hari  | hey@vigneshhari.dev |
| 2  | Gigin C       | hey@gigin.dev       |

**Tasks Table:**
| id | title        | completed | user_id |
|----|--------------|-----------|---------|
| 1  | Buy Milk     | False     | 1       |
| 2  | Learn Django | False     | 1       |
| 3  | Buy Bread    | False     | 2       |

**Joined Result:**
| task_id | title        | completed | user_id | user_name    | user_email          |
|---------|--------------|-----------|---------|--------------|---------------------|
| 1       | Buy Milk     | False     | 1       | Vignesh Hari | hey@vigneshhari.dev |
| 2       | Learn Django | False     | 1       | Vignesh Hari | hey@vigneshhari.dev |
| 3       | Buy Bread    | False     | 2       | Gigin C      | hey@gigin.dev       |

---

## Types of Relationships in Django

### 1. One-to-Many Relationship (ForeignKey)

**Use case:** One user can have many tasks, but each task belongs to only one user.

```python
# models.py
class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)

class Task(models.Model):
    title = models.CharField(max_length=100)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    completed = models.BooleanField(default=False)
```

**Key points:**
- `ForeignKey` field creates the relationship
- `on_delete=models.CASCADE` - delete tasks when user is deleted
- Access: `task.user` (get user) or `user.task_set` (get all user's tasks)

### 2. One-to-One Relationship (OneToOneField)

**Use case:** One user has one profile, and one profile belongs to one user.

```python
class User(models.Model):
    username = models.CharField(max_length=50, unique=True)
    email = models.EmailField(unique=True)

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    birth_date = models.DateField(null=True, blank=True)
```

**Key points:**
- `OneToOneField` ensures 1:1 relationship
- Access: `user.userprofile` or `profile.user`

### 3. Many-to-Many Relationship (ManyToManyField)

**Use case:** Tasks can have multiple tags, and tags can belong to multiple tasks.

```python
class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    color = models.CharField(max_length=7, default='#000000')  # Hex color

class Task(models.Model):
    title = models.CharField(max_length=100)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    tags = models.ManyToManyField(Tag, blank=True)
    completed = models.BooleanField(default=False)
```

**Key points:**
- `ManyToManyField` creates intermediate table automatically
- Access: `task.tags.all()` or `tag.task_set.all()`

---

## Implementing Relationships in Django

### Step 1: Define Models with Relationships

```python
# models.py
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    
    def __str__(self):
        return self.name

class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    
    # Relationships
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, blank=True)
    
    # Status fields
    completed = models.BooleanField(default=False)
    priority = models.IntegerField(choices=[
        (1, 'Low'),
        (2, 'Medium'),
        (3, 'High'),
        (4, 'Urgent')
    ], default=2)
    
    # Timestamps
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    def __str__(self):
        return f"{self.title} ({self.user.username})"
    
    class Meta:
        ordering = ['-created_at']

class Comment(models.Model):
    task = models.ForeignKey(Task, on_delete=models.CASCADE, related_name='comments')
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"Comment by {self.user.username} on {self.task.title}"
```

### Step 2: Create and Run Migrations

```bash
# Generate migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate
```

### Step 3: Working with Related Data

#### Creating Related Objects

```python
# Create user and task together
from django.contrib.auth.models import User
from .models import Task, Category

# Create user
user = User.objects.create_user(
    username='john_doe',
    email='john@example.com',
    password='password123'
)

# Create category
category = Category.objects.create(
    name='Work',
    description='Work-related tasks'
)

# Create task with relationships
task = Task.objects.create(
    title='Complete project report',
    description='Finish the quarterly report',
    user=user,
    category=category,
    priority=3
)

# Add comment to task
comment = Comment.objects.create(
    task=task,
    user=user,
    content='Started working on this today'
)
```

#### Querying Related Data

```python
# Get all tasks for a user
user_tasks = Task.objects.filter(user=user)

# Get tasks with related user data (avoids N+1 queries)
tasks_with_users = Task.objects.select_related('user').all()

# Get tasks with category data
tasks_with_categories = Task.objects.select_related('category').filter(
    category__name='Work'
)

# Get user's tasks with comments
user_tasks_with_comments = Task.objects.filter(user=user).prefetch_related('comments')

# Get all comments for user's tasks
all_comments = Comment.objects.filter(task__user=user)

# Get tasks by category name
work_tasks = Task.objects.filter(category__name='Work')

# Get users who have completed tasks
users_with_completed_tasks = User.objects.filter(task__completed=True).distinct()
```

#### Accessing Related Objects

```python
# Get task's user
task = Task.objects.get(id=1)
user = task.user  # Access related user
print(f"Task '{task.title}' belongs to {user.username}")

# Get user's tasks
user = User.objects.get(username='john_doe')
user_tasks = user.task_set.all()  # Reverse relationship
print(f"User {user.username} has {user_tasks.count()} tasks")

# Get task's comments
task_comments = task.comments.all()  # Using related_name
print(f"Task has {task_comments.count()} comments")

# Get comment's task and user
comment = Comment.objects.get(id=1)
print(f"Comment by {comment.user.username} on task '{comment.task.title}'")
```

---

## Advanced Relationship Features

### Custom Related Names

```python
class Task(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='tasks')
    # Now use: user.tasks instead of user.task_set

class Comment(models.Model):
    task = models.ForeignKey(Task, on_delete=models.CASCADE, related_name='comments')
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='task_comments')
```

### on_delete Options

| Option | Behavior |
|--------|----------|
| `CASCADE` | Delete related objects when parent is deleted |
| `PROTECT` | Prevent deletion if related objects exist |
| `SET_NULL` | Set foreign key to NULL |
| `SET_DEFAULT` | Set foreign key to default value |
| `DO_NOTHING` | Do nothing (dangerous!) |

### Self-Referencing Relationships

```python
class Employee(models.Model):
    name = models.CharField(max_length=100)
    manager = models.ForeignKey(
        'self', 
        on_delete=models.SET_NULL, 
        null=True, 
        blank=True,
        related_name='subordinates'
    )
    
    def __str__(self):
        return self.name
```

### Through Models for Many-to-Many

```python
class TaskAssignment(models.Model):
    task = models.ForeignKey(Task, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    assigned_at = models.DateTimeField(auto_now_add=True)
    role = models.CharField(max_length=50, choices=[
        ('owner', 'Owner'),
        ('assignee', 'Assignee'),
        ('reviewer', 'Reviewer')
    ])

class Task(models.Model):
    title = models.CharField(max_length=200)
    assignments = models.ManyToManyField(
        User, 
        through=TaskAssignment,
        related_name='assigned_tasks'
    )
```

---

## Views and Templates with Relationships

### Views Examples

```python
# views.py
from django.shortcuts import render, get_object_or_404
from django.contrib.auth.decorators import login_required
from .models import Task, Category

@login_required
def task_list(request):
    """List all tasks for current user"""
    tasks = Task.objects.filter(user=request.user).select_related('category')
    
    # Filter by category if provided
    category_id = request.GET.get('category')
    if category_id:
        tasks = tasks.filter(category_id=category_id)
    
    categories = Category.objects.all()
    
    return render(request, 'tasks/task_list.html', {
        'tasks': tasks,
        'categories': categories,
        'selected_category': category_id
    })

@login_required
def task_detail(request, task_id):
    """Show task details with comments"""
    task = get_object_or_404(
        Task, 
        id=task_id, 
        user=request.user  # Security: only user's own tasks
    )
    
    # Get comments with user data
    comments = task.comments.select_related('user').order_by('created_at')
    
    return render(request, 'tasks/task_detail.html', {
        'task': task,
        'comments': comments
    })

@login_required
def create_task(request):
    """Create a new task"""
    if request.method == 'POST':
        title = request.POST.get('title')
        description = request.POST.get('description')
        category_id = request.POST.get('category')
        
        category = None
        if category_id:
            category = get_object_or_404(Category, id=category_id)
        
        Task.objects.create(
            title=title,
            description=description,
            user=request.user,
            category=category
        )
        
        return redirect('task_list')
    
    categories = Category.objects.all()
    return render(request, 'tasks/create_task.html', {'categories': categories})
```

### Template Examples

```html
<!-- task_list.html -->
{% extends 'base.html' %}

{% block content %}
<h1>My Tasks</h1>

<!-- Category Filter -->
<form method="GET" class="mb-3">
    <select name="category" onchange="this.form.submit()">
        <option value="">All Categories</option>
        {% for category in categories %}
        <option value="{{ category.id }}" 
                {% if category.id|stringformat:"s" == selected_category %}selected{% endif %}>
            {{ category.name }}
        </option>
        {% endfor %}
    </select>
</form>

<!-- Tasks List -->
<div class="tasks">
{% for task in tasks %}
<div class="task-card {% if task.completed %}completed{% endif %}">
    <h3>
        <a href="{% url 'task_detail' task.id %}">{{ task.title }}</a>
        {% if task.completed %}✓{% endif %}
    </h3>
    
    {% if task.category %}
    <span class="badge">{{ task.category.name }}</span>
    {% endif %}
    
    <p>{{ task.description|truncatewords:20 }}</p>
    <small>Created {{ task.created_at|date:"M d, Y" }}</small>
    
    <div class="task-actions">
        {% if not task.completed %}
        <a href="{% url 'complete_task' task.id %}">Mark Complete</a>
        {% endif %}
        <a href="{% url 'edit_task' task.id %}">Edit</a>
        <a href="{% url 'delete_task' task.id %}" onclick="return confirm('Delete this task?')">Delete</a>
    </div>
</div>
{% empty %}
<p>No tasks found. <a href="{% url 'create_task' %}">Create your first task</a></p>
{% endfor %}
</div>

<a href="{% url 'create_task' %}" class="btn btn-primary">Add New Task</a>
{% endblock %}
```

```html
<!-- task_detail.html -->
{% extends 'base.html' %}

{% block content %}
<div class="task-detail">
    <h1>{{ task.title }}</h1>
    
    {% if task.category %}
    <p><strong>Category:</strong> {{ task.category.name }}</p>
    {% endif %}
    
    <p><strong>Status:</strong> 
        {% if task.completed %}Completed{% else %}Pending{% endif %}
    </p>
    
    <p><strong>Priority:</strong> {{ task.get_priority_display }}</p>
    
    {% if task.description %}
    <div class="task-description">
        <h3>Description</h3>
        <p>{{ task.description }}</p>
    </div>
    {% endif %}
    
    <div class="task-meta">
        <p>Created by {{ task.user.username }} on {{ task.created_at|date:"M d, Y H:i" }}</p>
        {% if task.updated_at != task.created_at %}
        <p>Last updated {{ task.updated_at|date:"M d, Y H:i" }}</p>
        {% endif %}
    </div>
    
    <!-- Comments Section -->
    <div class="comments">
        <h3>Comments ({{ comments.count }})</h3>
        
        {% for comment in comments %}
        <div class="comment">
            <strong>{{ comment.user.username }}</strong>
            <small>{{ comment.created_at|date:"M d, Y H:i" }}</small>
            <p>{{ comment.content }}</p>
        </div>
        {% endfor %}
        
        <!-- Add Comment Form -->
        <form method="POST" action="{% url 'add_comment' task.id %}">
            {% csrf_token %}
            <textarea name="content" placeholder="Add a comment..." required></textarea>
            <button type="submit">Add Comment</button>
        </form>
    </div>
</div>

<div class="task-actions">
    <a href="{% url 'edit_task' task.id %}">Edit Task</a>
    <a href="{% url 'task_list' %}">Back to Tasks</a>
</div>
{% endblock %}
```

---

## Performance Optimization

### Select Related (One-to-One, Foreign Key)

```python
# ❌ N+1 Query Problem
tasks = Task.objects.all()
for task in tasks:
    print(task.user.username)  # Separate query for each task!

# ✅ Solution: Use select_related
tasks = Task.objects.select_related('user').all()
for task in tasks:
    print(task.user.username)  # No additional queries!
```

### Prefetch Related (Many-to-Many, Reverse Foreign Key)

```python
# ❌ N+1 Query Problem
users = User.objects.all()
for user in users:
    print(f"{user.username}: {user.task_set.count()} tasks")  # Separate query per user!

# ✅ Solution: Use prefetch_related
users = User.objects.prefetch_related('task_set').all()
for user in users:
    print(f"{user.username}: {user.task_set.count()} tasks")  # Single additional query!
```

### When to Use Each

| Method | Use Case | Relationships |
|--------|----------|---------------|
| `select_related` | One-to-One, Foreign Key | Reduces JOINs in single query |
| `prefetch_related` | Many-to-Many, Reverse FK | Batches related queries |

---

## Best Practices

### 1. Plan Relationships Carefully
- Think about data access patterns
- Consider future requirements
- Avoid over-normalization

### 2. Use Appropriate on_delete
```python
# User posts - cascade delete
post = models.ForeignKey(User, on_delete=models.CASCADE)

# User profile - set null
profile = models.OneToOneField(User, on_delete=models.SET_NULL, null=True)

# Important data - protect
invoice = models.ForeignKey(Client, on_delete=models.PROTECT)
```

### 3. Add Database Indexes
```python
class Task(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
```

### 4. Use Constraints for Data Integrity
```python
class Task(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    
    class Meta:
        unique_together = ['user', 'title']  # User can't have duplicate task titles
```

### 5. Handle Circular Dependencies
```python
# Avoid this - circular import
from .models import Task  # In User model file
# Solution: Use string references
task = models.ForeignKey('Task', on_delete=models.CASCADE)
```

---

## Common Patterns

### User-Specific Data
```python
@login_required
def my_view(request):
    # Always filter by current user for security
    my_objects = MyModel.objects.filter(user=request.user)
    return render(request, 'template.html', {'objects': my_objects})
```

### Generic Foreign Keys (Advanced)
```python
from django.contrib.contenttypes.fields import GenericForeignKey
from django.contrib.contenttypes.models import ContentType

class Comment(models.Model):
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey('content_type', 'object_id')
    text = models.TextField()
    # Can comment on any model (Task, Article, etc.)
```

---

## Key Takeaways

1. **Relationships eliminate data duplication** and maintain consistency
2. **ForeignKey** creates one-to-many relationships
3. **OneToOneField** creates one-to-one relationships
4. **ManyToManyField** creates many-to-many relationships
5. **Choose appropriate `on_delete` behavior** for data integrity
6. **Use `select_related`** and `prefetch_related`** for performance
7. **Always filter by user** in multi-user applications for security
8. **Plan relationships carefully** - they affect your entire application

---

## Resources

- [Django Model Relationships Documentation](https://docs.djangoproject.com/en/3.2/topics/db/models/#relationships)
- [Database Normalization](https://docs.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description)
- [Query Optimization](https://docs.djangoproject.com/en/3.2/topics/db/optimization/)

---

**Mastering relationships is key to building scalable Django applications!** 🎯

---

## Practice Exercise

Create a blog application with the following relationships:

1. **User** ↔ **Post** (One-to-Many)
2. **Post** ↔ **Category** (Many-to-Many)
3. **Post** ↔ **Comment** (One-to-Many)
4. **User** ↔ **Comment** (One-to-Many)

Try implementing CRUD operations for each model and experiment with different queries!

# Django Admin - Complete Guide

## Introduction to Django Admin

Django comes with a powerful **automatic admin interface** that lets you manage your application's data through a web-based UI. It's one of Django's most powerful features for rapid application development.

### What is Django Admin?

- **Auto-generated interface** for CRUD operations on your models
- **Production-ready** with authentication and permissions
- **Customizable** - from simple tweaks to completely custom interfaces
- **Secure** - built-in authentication and authorization
- **Extensible** - add custom actions, filters, and displays

### When to Use Django Admin

✅ **Perfect for:**
- Internal tools and content management
- Rapid prototyping and development
- Administrative interfaces
- Data management for small teams

❌ **Not ideal for:**
- Public-facing user interfaces
- Highly customized user experiences
- Complex business logic workflows

---

## Getting Started with Django Admin

### Step 1: Access the Admin Interface

Django admin is automatically included in new Django projects. Access it at:

```
http://localhost:8000/admin/
```

### Step 2: Create a Superuser

Create an admin user to access the interface:

```bash
python manage.py createsuperuser
```

**Follow the prompts:**
```
Username: admin
Email address: admin@example.com
Password: ********
Password (again): ********
Superuser created successfully.
```

### Step 3: Login

Visit `/admin/` and login with your superuser credentials.

---

## Registering Models with Admin

### Basic Model Registration

By default, Django admin doesn't show your custom models. You need to register them.

**In your app's `admin.py`:**

```python
# admin.py
from django.contrib import admin
from .models import Task

# Basic registration
admin.site.register(Task)
```

**That's it!** Your model now appears in the admin interface.

### What You Get Automatically

With basic registration, Django provides:
- ✅ **List view** - See all records
- ✅ **Detail view** - View individual records
- ✅ **Add form** - Create new records
- ✅ **Edit form** - Update existing records
- ✅ **Delete confirmation** - Safe deletion
- ✅ **Search** - Basic search functionality
- ✅ **Pagination** - Handle large datasets

---

## Customizing Admin Interface

### ModelAdmin Class

For more control, use the `ModelAdmin` class:

```python
# admin.py
from django.contrib import admin
from .models import Task

@admin.register(Task)  # Alternative to admin.site.register()
class TaskAdmin(admin.ModelAdmin):
    # Customize list view
    list_display = ['title', 'completed', 'created_at', 'user']
    list_filter = ['completed', 'created_at', 'user']
    search_fields = ['title', 'description']
    ordering = ['-created_at']
    
    # Customize detail/edit view
    fieldsets = (
        ('Basic Information', {
            'fields': ('title', 'description', 'user')
        }),
        ('Status', {
            'fields': ('completed', 'priority'),
            'classes': ('collapse',)  # Collapsible section
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    
    # Make fields read-only
    readonly_fields = ['created_at', 'updated_at']
    
    # Customize form
    formfield_overrides = {
        models.TextField: {'widget': forms.Textarea(attrs={'rows': 4})},
    }
```

### List View Customizations

#### Display Fields
```python
class TaskAdmin(admin.ModelAdmin):
    list_display = ['title', 'user', 'completed', 'priority', 'created_at']
    
    # Custom display methods
    def priority_display(self, obj):
        return obj.get_priority_display()  # For choice fields
    priority_display.short_description = 'Priority Level'
    
    def user_name(self, obj):
        return obj.user.username if obj.user else 'No User'
    user_name.short_description = 'Assigned To'
```

#### List Filters
```python
class TaskAdmin(admin.ModelAdmin):
    list_filter = [
        'completed',
        'priority',
        ('created_at', admin.DateFieldListFilter),  # Date range filter
        'user',
    ]
```

#### Search Fields
```python
class TaskAdmin(admin.ModelAdmin):
    search_fields = ['title', 'description', 'user__username', 'user__email']
    # Use double underscore for related field search
```

#### Ordering and Pagination
```python
class TaskAdmin(admin.ModelAdmin):
    ordering = ['-created_at']  # Default ordering
    list_per_page = 25  # Items per page
    list_max_show_all = 200  # Max items for "show all"
```

---

## Advanced Admin Features

### Custom Actions

Add bulk actions to the list view:

```python
class TaskAdmin(admin.ModelAdmin):
    actions = ['mark_completed', 'mark_pending', 'export_csv']
    
    def mark_completed(self, request, queryset):
        updated = queryset.update(completed=True)
        self.message_user(
            request,
            f'{updated} tasks marked as completed.',
            messages.SUCCESS
        )
    mark_completed.short_description = 'Mark selected tasks as completed'
    
    def mark_pending(self, request, queryset):
        updated = queryset.update(completed=False)
        self.message_user(
            request,
            f'{updated} tasks marked as pending.',
            messages.WARNING
        )
    mark_pending.short_description = 'Mark selected tasks as pending'
    
    def export_csv(self, request, queryset):
        import csv
        from django.http import HttpResponse
        
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment; filename="tasks.csv"'
        
        writer = csv.writer(response)
        writer.writerow(['Title', 'User', 'Completed', 'Created'])
        
        for task in queryset:
            writer.writerow([
                task.title,
                task.user.username if task.user else '',
                task.completed,
                task.created_at
            ])
        
        return response
    export_csv.short_description = 'Export selected tasks to CSV'
```

### Inline Admin

Display related models directly on the parent model's page:

```python
# admin.py
from django.contrib import admin
from .models import Task, Comment

class CommentInline(admin.TabularInline):  # or admin.StackedInline
    model = Comment
    extra = 0  # Number of empty forms to show
    readonly_fields = ['created_at']
    ordering = ['-created_at']

class TaskAdmin(admin.ModelAdmin):
    inlines = [CommentInline]
    # ... other configurations
```

### Custom Forms

Use custom forms for validation and widgets:

```python
from django import forms
from .models import Task

class TaskAdminForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = '__all__'
    
    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 3:
            raise forms.ValidationError('Title must be at least 3 characters long.')
        return title
    
    def clean(self):
        cleaned_data = super().clean()
        user = cleaned_data.get('user')
        priority = cleaned_data.get('priority')
        
        if priority == 4 and not user:  # Urgent tasks must be assigned
            raise forms.ValidationError('Urgent tasks must be assigned to a user.')
        
        return cleaned_data

class TaskAdmin(admin.ModelAdmin):
    form = TaskAdminForm
```

### Conditional Field Display

Show/hide fields based on conditions:

```python
class TaskAdmin(admin.ModelAdmin):
    def get_form(self, request, obj=None, **kwargs):
        form = super().get_form(request, obj, **kwargs)
        if not request.user.is_superuser:
            # Non-superusers can't change priority
            form.base_fields['priority'].disabled = True
        return form
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if not request.user.is_superuser:
            # Non-superusers only see their own tasks
            return qs.filter(user=request.user)
        return qs
```

---

## Admin with Relationships

### Foreign Key Handling

When you have foreign keys, Django admin handles them automatically:

```python
# models.py
from django.contrib.auth.models import User

class Task(models.Model):
    title = models.CharField(max_length=100)
    user = models.ForeignKey(
        User, 
        on_delete=models.CASCADE,
        null=True, 
        blank=True
    )
    completed = models.BooleanField(default=False)
```

**Admin automatically provides:**
- Dropdown for user selection
- Search/lookup for large user tables
- Display of related user information

### Many-to-Many Relationships

```python
class Task(models.Model):
    title = models.CharField(max_length=100)
    tags = models.ManyToManyField('Tag', blank=True)

class Tag(models.Model):
    name = models.CharField(max_length=50)
    color = models.CharField(max_length=7, default='#000000')
```

**Admin provides:**
- Multiple select widget
- Add new tags inline
- Filter by tags

### Custom Foreign Key Widgets

For better UX with large datasets:

```python
class TaskAdmin(admin.ModelAdmin):
    autocomplete_fields = ['user']  # Search as you type
    
    # Or use raw_id_fields for popup selection
    raw_id_fields = ['user']
```

---

## Admin Security and Permissions

### Staff vs Superuser

- **Superuser**: Full access to everything
- **Staff user**: Limited access based on permissions

### Custom Permissions

```python
class TaskAdmin(admin.ModelAdmin):
    def has_add_permission(self, request):
        return request.user.is_superuser
    
    def has_change_permission(self, request, obj=None):
        if obj and obj.user == request.user:
            return True  # Users can edit their own tasks
        return request.user.is_superuser
    
    def has_delete_permission(self, request, obj=None):
        return request.user.is_superuser
```

### Group-Based Permissions

Create groups in admin and assign permissions:

1. Go to `/admin/auth/group/`
2. Create groups like "Task Managers", "Content Editors"
3. Assign specific model permissions to groups
4. Add users to appropriate groups

---

## Customizing Admin Appearance

### Admin Site Customization

```python
# admin.py
from django.contrib import admin

# Customize admin site
admin.site.site_header = "My Project Admin"
admin.site.site_title = "My Project Admin Portal"
admin.site.index_title = "Welcome to My Project Admin"

# Custom admin site
class MyAdminSite(admin.AdminSite):
    site_header = "My Custom Admin"
    site_title = "My Custom Admin Portal"
    index_title = "Dashboard"

# Create instance
my_admin_site = MyAdminSite(name='myadmin')

# Register models with custom admin
my_admin_site.register(Task, TaskAdmin)
```

### Custom CSS/JavaScript

```python
class TaskAdmin(admin.ModelAdmin):
    class Media:
        css = {
            'all': ('css/admin_custom.css',)
        }
        js = ('js/admin_custom.js',)
```

### Custom Admin Templates

Override admin templates by creating:

```
templates/
    admin/
        base.html
        change_list.html
        change_form.html
```

---

## Production Considerations

### Security Settings

```python
# settings.py
# Admin URL customization (change from default 'admin/')
from django.urls import reverse_lazy

# Custom admin URL
ADMIN_URL = 'manage/'  # Instead of 'admin/'

# Admin settings
ADMIN_SITE_HEADER = "My Project Administration"
ADMIN_SITE_TITLE = "My Project Admin Portal"
ADMIN_INDEX_TITLE = "Dashboard"

# Security
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

### Admin Logging

```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'admin_actions.log',
        },
    },
    'loggers': {
        'django.admin': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```

### Performance Optimization

```python
class TaskAdmin(admin.ModelAdmin):
    # Optimize list queries
    list_select_related = ['user']  # For foreign keys
    list_prefetch_related = ['tags']  # For many-to-many
    
    # Optimize detail queries
    autocomplete_fields = ['user']  # Reduces queries
    raw_id_fields = ['category']  # For large foreign key tables
```

---

## Complete Example

### models.py
```python
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    
    def __str__(self):
        return self.name

class Task(models.Model):
    PRIORITY_CHOICES = [
        (1, 'Low'),
        (2, 'Medium'),
        (3, 'High'),
        (4, 'Urgent'),
    ]
    
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)
    priority = models.IntegerField(choices=PRIORITY_CHOICES, default=2)
    
    # Relationships
    user = models.ForeignKey(User, on_delete=models.CASCADE, null=True, blank=True)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, blank=True)
    
    # Timestamps
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    def __str__(self):
        return f"{self.title} ({self.user.username if self.user else 'Unassigned'})"
    
    class Meta:
        ordering = ['-created_at']
```

### admin.py
```python
from django.contrib import admin
from django.contrib import messages
from django.utils.translation import ngettext
from .models import Task, Category

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'description']
    search_fields = ['name', 'description']

@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    # List view
    list_display = ['title', 'user', 'category', 'priority_display', 'completed', 'created_at']
    list_filter = ['completed', 'priority', 'category', 'created_at', 'user']
    search_fields = ['title', 'description', 'user__username', 'user__email']
    ordering = ['-created_at']
    list_per_page = 25
    
    # Detail view
    fieldsets = (
        ('Basic Information', {
            'fields': ('title', 'description', 'category')
        }),
        ('Assignment & Priority', {
            'fields': ('user', 'priority'),
        }),
        ('Status', {
            'fields': ('completed',),
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    
    readonly_fields = ['created_at', 'updated_at']
    
    # Custom methods
    def priority_display(self, obj):
        return obj.get_priority_display()
    priority_display.short_description = 'Priority'
    priority_display.admin_order_field = 'priority'
    
    # Actions
    actions = ['mark_completed', 'mark_pending', 'set_high_priority']
    
    def mark_completed(self, request, queryset):
        updated = queryset.update(completed=True)
        self.message_user(
            request,
            ngettext(
                '%d task was successfully marked as completed.',
                '%d tasks were successfully marked as completed.',
                updated,
            ) % updated,
            messages.SUCCESS,
        )
    mark_completed.short_description = 'Mark selected tasks as completed'
    
    def mark_pending(self, request, queryset):
        updated = queryset.update(completed=False)
        self.message_user(
            request,
            ngettext(
                '%d task was successfully marked as pending.',
                '%d tasks were successfully marked as pending.',
                updated,
            ) % updated,
            messages.WARNING,
        )
    mark_pending.short_description = 'Mark selected tasks as pending'
    
    def set_high_priority(self, request, queryset):
        updated = queryset.update(priority=3)
        self.message_user(
            request,
            ngettext(
                '%d task was set to high priority.',
                '%d tasks were set to high priority.',
                updated,
            ) % updated,
            messages.INFO,
        )
    set_high_priority.short_description = 'Set selected tasks to high priority'
    
    # Performance optimization
    list_select_related = ['user', 'category']
    
    # Permissions
    def has_delete_permission(self, request, obj=None):
        # Only superusers can delete
        return request.user.is_superuser
```

---

## Key Takeaways

1. **Django admin** provides automatic CRUD interfaces for your models
2. **ModelAdmin classes** customize list views, filters, and forms
3. **Custom actions** enable bulk operations on selected items
4. **Inline admin** displays related models on the same page
5. **Permissions** control what users can see and do
6. **Performance optimization** is crucial for large datasets
7. **Security** should be a top priority in production

---

## Resources

- [Django Admin Documentation](https://docs.djangoproject.com/en/3.2/ref/contrib/admin/)
- [ModelAdmin Reference](https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#modeladmin-options)
- [Admin Actions](https://docs.djangoproject.com/en/3.2/ref/contrib/admin/actions/)

---

**Django admin is a powerful tool that can save you hours of development time!** 🚀

---

## Practice Exercise

Enhance the Task admin with the following features:

1. **Custom list display** showing task status with colors
2. **Date hierarchy** for filtering by creation date
3. **Export functionality** for selected tasks
4. **Inline editing** for quick status updates
5. **Custom permissions** based on user roles

Experiment with different admin customizations to understand the full potential!

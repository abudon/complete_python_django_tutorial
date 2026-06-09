# Django Migrations - Complete Guide

## Why Migrations Matter

Database schemas are **critical** to application stability. Even small changes can break your entire application. Django doesn't automatically sync schema changes to protect your data and maintain stability.

### The Problem Without Migrations

Imagine you have `first_name` and `last_name` fields that you want to merge into a single `name` field:

1. **Remove** `first_name` and `last_name` fields
2. **Create** new `name` field
3. **Sync** changes → **ALL EXISTING DATA IS LOST!**

**Migrations solve this problem** by tracking schema changes like Git tracks code changes.

---

## What Are Migrations?

**Migrations** are Django's way of:
- **Recording** database schema changes
- **Maintaining** change history
- **Enabling** schema versioning
- **Allowing** forward/backward schema movement

Think of migrations as **Git for your database schema**.

---

## Migration Commands

Django provides two primary commands for migration management:

### `makemigrations`
**Purpose**: Creates new migration files based on model changes

```bash
python manage.py makemigrations
```

- **Scans** your models for changes
- **Generates** migration files in `app/migrations/`
- **Safe to run** - only creates files, doesn't modify database

### `migrate`
**Purpose**: Applies migrations to actually change the database schema

```bash
python manage.py migrate
```

- **Executes** pending migrations
- **Updates** database schema
- **Can target** specific migration versions

### Migration Files Location

All migrations are stored in your app's `migrations/` folder:

```
myapp/
    migrations/
        __init__.py
        0001_initial.py
        0002_add_field.py
        0003_remove_field.py
```

**⚠️ Warning**: Never manually edit migration files unless you know exactly what you're doing!

---

## Creating Your First Migration

### Step 1: Modify Your Model

```python
# models.py
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    # Add new field
    age = models.IntegerField(default=0)
```

### Step 2: Generate Migration

```bash
python manage.py makemigrations
```

**Output:**
```
Migrations for 'myapp':
  myapp/migrations/0002_person_age.py
    - Add field age to person
```

### Step 3: Apply Migration

```bash
python manage.py migrate
```

**Output:**
```
Operations to perform:
  Apply all migrations: myapp
Running migrations:
  Applying myapp.0002_person_age... OK
```

---

## Custom Migrations

Sometimes you need **custom logic** for complex schema changes. Custom migrations allow you to write Python code to handle data transformations.

### Real-World Example: Merging Fields

**Scenario**: Merge `first_name` and `last_name` into a single `name` field without losing data.

### Step-by-Step Process

#### 1. Add New Field (Keep Old Fields)

```python
# models.py
class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    # Add new field
    name = models.CharField(max_length=100, blank=True)
```

#### 2. Create Migration for New Field

```bash
python manage.py makemigrations
python manage.py migrate
```

#### 3. Create Custom Data Migration

```bash
python manage.py makemigrations --empty myapp
```

This creates an empty migration file. Edit it:

```python
# migrations/0003_merge_names.py
from django.db import migrations

def merge_names(apps, schema_editor):
    Person = apps.get_model('myapp', 'Person')
    for person in Person.objects.all():
        person.name = f"{person.first_name} {person.last_name}"
        person.save()

def reverse_merge_names(apps, schema_editor):
    Person = apps.get_model('myapp', 'Person')
    for person in Person.objects.all():
        # Split name back (simplified example)
        parts = person.name.split(' ', 1)
        person.first_name = parts[0]
        person.last_name = parts[1] if len(parts) > 1 else ''
        person.save()

class Migration(migrations.Migration):
    dependencies = [
        ('myapp', '0002_add_name_field'),
    ]

    operations = [
        migrations.RunPython(merge_names, reverse_merge_names),
    ]
```

#### 4. Remove Old Fields

```python
# models.py
class Person(models.Model):
    # Remove old fields
    name = models.CharField(max_length=100)
```

#### 5. Create and Apply Final Migration

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Migration Safety Features

### Atomic Transactions

**Migrations are atomic** - if any part fails, the entire migration is rolled back:

- ✅ **Success**: All changes applied
- ❌ **Failure**: Database returns to pre-migration state

### Migration History

View migration history:

```bash
python manage.py showmigrations
```

**Output:**
```
myapp
 [X] 0001_initial
 [X] 0002_add_age
 [ ] 0003_merge_names
```

### Rolling Back Migrations

Rollback to specific migration:

```bash
python manage.py migrate myapp 0001
```

This undoes migrations `0002` and `0003`.

---

## Best Practices

### 1. Always Backup Before Migrating
```bash
# Create database backup
pg_dump mydb > backup.sql  # PostgreSQL
mysqldump mydb > backup.sql  # MySQL
```

### 2. Test Migrations
- **Run on staging** before production
- **Test rollback** procedures
- **Verify data integrity** after migration

### 3. Commit Migration Files
```bash
git add migrations/
git commit -m "Add age field to Person model"
```

### 4. Never Delete Migration Files
Migration files are part of your project's history.

### 5. Use Descriptive Names
```bash
python manage.py makemigrations --name add_user_profile_fields
```

---

## Common Migration Scenarios

### Adding Fields
```python
# Add field with default
new_field = models.CharField(max_length=100, default='default_value')
```

### Removing Fields
```python
# Django handles data deletion automatically
# But backup first!
```

### Renaming Fields
```python
# Use migrations.RenameField
operations = [
    migrations.RenameField(
        model_name='person',
        old_name='first_name',
        new_name='given_name',
    ),
]
```

### Changing Field Types
```python
# May require data conversion
operations = [
    migrations.AlterField(
        model_name='person',
        name='age',
        field=models.CharField(max_length=3),  # Was IntegerField
    ),
]
```

---

## Troubleshooting

### Migration Errors

**"Table already exists"**
```bash
# Reset migrations (CAUTION: destroys data)
python manage.py migrate --fake-initial
```

**"No migrations to apply"**
```bash
# Force migration creation
python manage.py makemigrations --empty your_app
```

**"Inconsistent migration history"**
```bash
# Fake migrations to fix history
python manage.py migrate --fake
```

---

## Advanced Topics

### Migration Dependencies
Migrations can depend on other migrations:

```python
class Migration(migrations.Migration):
    dependencies = [
        ('myapp', '0001_initial'),
        ('otherapp', '0002_some_migration'),
    ]
```

### Migration Squashing
Combine multiple migrations into one:

```bash
python manage.py squashmigrations myapp 0001 0005
```

### Custom Migration Operations
Create reusable migration operations by extending `migrations.Operation`.

---

## Key Takeaways

1. **Migrations protect your data** - Never modify schema directly
2. **Always backup** before running migrations
3. **Test migrations** on staging environment first
4. **Custom migrations** handle complex data transformations
5. **Migrations are atomic** - All or nothing execution
6. **Never delete** migration files from version control
7. **Commit migration files** with your code changes

---

## Resources

- [Django Migrations Documentation](https://docs.djangoproject.com/en/3.2/topics/migrations/)
- [Data Migrations Guide](https://docs.djangoproject.com/en/3.2/topics/migrations/#data-migrations)
- [Migration Operations Reference](https://docs.djangoproject.com/en/3.2/ref/migration-operations/)

---

**Remember**: Migrations are one of Django's most powerful features. Master them to build robust, maintainable applications!

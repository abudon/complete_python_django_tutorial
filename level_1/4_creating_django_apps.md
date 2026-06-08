
# Lesson 4: Creating Django Apps

## What is an App?
An app is a module that performs a specific function.

## Create App
```bash
python manage.py startapp pages
```

## App Structure
- views.py
- models.py
- admin.py
- apps.py

## Register App
Add to `INSTALLED_APPS` in `settings.py`:
```python
'pages',
```

Why register?
- Enables Django to recognize your app

---
End of Lesson 4

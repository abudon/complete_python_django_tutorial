# Getting Started with Django

## Overview

In this guide, we'll walk through the basic steps to get Django up and running. Django is a powerful Python web framework that makes building web applications faster and easier. We'll cover:
- Installing Django
- Creating a Django project
- Understanding the project structure
- Starting the development server

---

## Step 1: Installing Django

### What is Django?

Django is a Python package that needs to be installed through pip (Python's package manager). It comes with all the tools and libraries you need to build web applications quickly and efficiently.

### Installation Command

```bash
pip install django
```

**What this does:**
- Installs the latest version of Django
- Automatically installs all dependencies (other packages that Django needs to function)
- Adds the `django-admin` command-line utility to your system

> **Note:** Make sure you have Python and pip installed on your system before running this command.

---

## Step 2: Creating a Django Project

### Using django-admin

Once Django is installed, we create a new project using the `django-admin` command (which comes automatically with Django):

```bash
django-admin startproject my_project
```

**What this does:**
- Creates a new folder named `my_project`
- Generates all the essential Django project files
- Sets up the basic structure needed to start building your web application

### Understanding the Project Structure

After running the command above, your project folder will look like this:

```
my_project/
    manage.py
    my_project/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

---

## Step 3: Project Files Explained

Let's understand what each file does and why it's important:

### Outer `my_project/` folder
This is your project's root directory containing all project files.

### Core Files

**`manage.py`**
- Similar to the `tasks.py` file you used in the previous level
- A command-line utility that helps you interact with your Django project
- Used to:
    - Start the development server
    - Run database migrations
    - Execute custom commands
    - Manage your entire project

**`settings.py`**
- The configuration hub of your project
- Contains all settings that control how Django behaves
- Examples of settings you can modify:
    - Database configuration
    - Installed applications
    - Allowed hosts
    - Static files location
    - Timezone and language preferences

**`urls.py`**
- Defines all URL patterns for your project
- Works like the `if` conditions you used before to route different URLs
- As your project grows, URL patterns can become complex, so having them organized in one file makes management much easier
- Routes incoming requests to the appropriate views

### Package Files

**`__init__.py`**
- An empty Python file (can contain code, but usually left empty)
- Tells Python to treat this folder as a package
- Must be present for Python to recognize the directory structure

### Deployment Files (Advanced)

**`asgi.py` and `wsgi.py`**
- Used for running Django in production environments
- `WSGI` (Web Server Gateway Interface) - for traditional web servers
- `ASGI` (Asynchronous Server Gateway Interface) - for modern async servers
- **For now:** Leave these as they are. We'll revisit them once you have a solid grasp of Django basics

---

## Step 4: Starting the Development Server

### What is a Development Server?

A development server is a lightweight, local server used only for testing your application during development. It allows you to:
- Test your code in a real browser environment
- See changes in real-time
- Debug issues easily
- The server runs on your local machine

### Starting the Server

Use the `manage.py` utility we discussed earlier:

```bash
python manage.py runserver
```

**What this does:**
- Starts Django's development server
- Runs your project locally at `http://localhost:8000`

### Testing the Server

1. Run the command above in your terminal
2. Open your web browser
3. Visit: `http://localhost:8000`
4. You should see Django's default welcome page

> **Tip:** To stop the server, press `Ctrl+C` in your terminal

---

## What's Next?

Now that Django is installed and running, you're ready to:
- Create Django apps to organize your project
- Build views to handle requests
- Create templates to display content
- Set up models for your database
- Configure URLs to route traffic

Your next step will be to modify the configuration files and start creating content. Start by creating your first Django app!

---

## Common Issues & Tips

- **Port already in use?** Django defaults to port 8000. If it's in use, run: `python manage.py runserver 8080`
- **Command not found?** Make sure Django is installed: `pip install django`
- **Permission denied?** You may need admin privileges to install packages

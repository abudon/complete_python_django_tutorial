# Django Templates - Complete Lecture Guide

## Table of Contents
1. [Introduction to Templates](#introduction-to-templates)
2. [Template Setup](#template-setup)
3. [Template Syntax](#template-syntax)
4. [Variables and Context](#variables-and-context)
5. [Filters](#filters)
6. [Tags](#tags)
7. [Template Inheritance](#template-inheritance)
8. [Built-in Tags Reference](#built-in-tags-reference)
9. [Practical Examples](#practical-examples)

---

## Introduction to Templates

### What are Django Templates?

Django templates are HTML files that contain **static HTML content** mixed with **dynamic content** generated from Python code. They:
- Allow you to generate HTML dynamically
- Separate presentation (HTML) from business logic (Python)
- Enable code reuse through inheritance
- Make your web pages flexible and maintainable

### Why Use Templates?

1. **Separation of Concerns** - HTML stays in templates, logic stays in Python
2. **Reusability** - Share common HTML structure across pages
3. **Maintainability** - Easy to modify appearance without touching Python code
4. **Security** - Built-in protection against common attacks
5. **Flexibility** - Display different content based on conditions and data

### The Template Engine

Django uses its own **template engine** that processes templates by:
1. Loading the template file
2. Passing a **context** (data dictionary) to it
3. Rendering the template with the context data
4. Returning the final HTML string

---

## Template Setup

### Basic Directory Structure

Templates should be stored in a `templates` folder in your app directory:

```
my_project/
    manage.py
    my_project/
        settings.py
        urls.py
    myapp/
        migrations/
        templates/              <- Templates folder
            myapp/
                home.html
                about.html
                post_detail.html
        views.py
        urls.py
        models.py
```

### Configuration in settings.py

Django automatically finds templates if you configure it correctly:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # Your app must be listed here
]

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            BASE_DIR / 'templates',  # Project-level templates
        ],
        'APP_DIRS': True,  # Look for templates in app folders
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

### Rendering a Template in Views

**views.py**
```python
from django.shortcuts import render

def home(request):
    context = {
        'title': 'Home Page',
        'author': 'John Doe'
    }
    return render(request, 'myapp/home.html', context)
```

---

## Template Syntax

### Variable Display

Display data passed from your view using **double curly braces**:

```html
<h1>{{ title }}</h1>
<p>Author: {{ author }}</p>
```

### Accessing Dictionary and List Items

Access nested data structures:

```html
<!-- Access dictionary items -->
<p>{{ user['name'] }}</p>
<p>{{ user.email }}</p>  <!-- Also works with dot notation -->

<!-- Access list items -->
<p>{{ items.0 }}</p>  <!-- First item -->
<p>{{ items.1 }}</p>  <!-- Second item -->
```

### Attribute and Method Access

```html
<!-- Accessing object attributes -->
<p>{{ post.title }}</p>
<p>{{ post.author.name }}</p>

<!-- Calling methods (without parentheses) -->
<p>{{ comment.text|lower }}</p>
```

---

## Variables and Context

### The Context Dictionary

The **context** is a Python dictionary that contains data passed to the template:

```python
def detail_view(request, id):
    post = Post.objects.get(id=id)
    context = {
        'post': post,
        'title': 'Post Detail',
        'view_count': 150,
        'is_published': True
    }
    return render(request, 'post_detail.html', context)
```

### Template Usage

```html
<h1>{{ title }}</h1>
<h2>{{ post.title }}</h2>
<p>{{ post.content }}</p>
<p>Views: {{ view_count }}</p>
{% if is_published %}
    <span>Published</span>
{% endif %}
```

### Special Variables

Django provides some built-in variables:

```html
<!-- Current request object -->
{{ request.method }}
{{ request.user }}
{{ request.path }}

<!-- Current user -->
{{ user }}
{{ user.username }}
{{ user.is_authenticated }}
```

---

## Filters

### What are Filters?

**Filters** modify the output of variables. They use the **pipe character** `|` and can be chained:

```html
{{ value|filter_name }}
{{ value|filter1|filter2 }}
```

### Common Filters

#### String Filters

```html
<!-- Lowercase -->
{{ text|lower }}
<!-- Input: "Hello World" -->
<!-- Output: "hello world" -->

<!-- Uppercase -->
{{ text|upper }}
<!-- Input: "Hello World" -->
<!-- Output: "HELLO WORLD" -->

<!-- Capitalize first letter -->
{{ text|title }}
<!-- Input: "hello world" -->
<!-- Output: "Hello World" -->

<!-- Replace text -->
{{ text|cut:" " }}
<!-- Input: "Hello World" -->
<!-- Output: "HelloWorld" -->

<!-- Slug format (URL-safe) -->
{{ text|slugify }}
<!-- Input: "Hello World!" -->
<!-- Output: "hello-world" -->

<!-- Word truncate -->
{{ text|truncatewords:3 }}
<!-- Input: "The quick brown fox" -->
<!-- Output: "The quick brown ..." -->

<!-- Character truncate -->
{{ text|truncatechars:10 }}
<!-- Input: "Hello World" -->
<!-- Output: "Hello W..." -->
```

#### Number Filters

```html
<!-- Add number -->
{{ value|add:5 }}
<!-- Input: 10 -->
<!-- Output: 15 -->

<!-- Default value if empty -->
{{ value|default:"No value" }}
<!-- Input: (empty) -->
<!-- Output: "No value" -->

<!-- Format decimal places -->
{{ value|floatformat:2 }}
<!-- Input: 123.456 -->
<!-- Output: "123.46" -->

<!-- List length -->
{{ items|length }}
```

#### Date Filters

```html
<!-- Format date -->
{{ date|date:"Y-m-d" }}
<!-- Input: 2024-12-25 -->
<!-- Output: "2024-12-25" -->

{{ date|date:"F j, Y" }}
<!-- Output: "December 25, 2024" -->

{{ date|time:"H:i" }}
<!-- Output: "14:30" (if time is 2:30 PM) -->
```

#### Other Useful Filters

```html
<!-- Pluralize (add 's' if needed) -->
{{ count }} item{{ items|pluralize }}
<!-- Input: 1 item OR 2 items -->

<!-- Join list items -->
{{ list|join:", " }}
<!-- Input: ['apple', 'banana', 'cherry'] -->
<!-- Output: "apple, banana, cherry" -->

<!-- Escape HTML -->
{{ dangerous_html|escape }}
<!-- Prevents XSS attacks -->

<!-- Safe (mark as safe HTML - use carefully!) -->
{{ trusted_html|safe }}
```

---

## Tags

### What are Tags?

**Tags** perform logic operations in templates. They use **curly percent braces** `{% %}`:

```html
{% tag_name %}
{% tag_name argument %}
{% tag_name argument1 argument2 %}
```

### Conditional Tags

#### if / elif / else

```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}!</p>
{% elif user.is_anonymous %}
    <p>Please log in</p>
{% else %}
    <p>Unknown user</p>
{% endif %}
```

Comparison operators:
```html
{% if age >= 18 %}
    <p>You are an adult</p>
{% endif %}

{% if name == "John" %}
    <p>Hello John!</p>
{% endif %}

{% if items %}
    <p>You have items</p>
{% endif %}
```

### Loop Tags

#### for

Loop through lists and querysets:

```html
<ul>
{% for item in items %}
    <li>{{ item }}</li>
{% endfor %}
</ul>
```

Loop counter variables:
```html
<ul>
{% for item in items %}
    <li>
        Item {{ forloop.counter }}: {{ item }}
        <!-- forloop.counter starts at 1 -->
        <!-- forloop.counter0 starts at 0 -->
        <!-- forloop.revcounter counts backwards from end -->
        <!-- forloop.first True for first iteration -->
        <!-- forloop.last True for last iteration -->
    </li>
{% endfor %}
</ul>
```

Empty fallback:
```html
{% for item in items %}
    <p>{{ item }}</p>
{% empty %}
    <p>No items available</p>
{% endfor %}
```

Nested loops:
```html
<table>
{% for user in users %}
    <tr>
        <td>{{ user.name }}</td>
        <td>
            <ul>
            {% for post in user.posts %}
                <li>{{ post.title }}</li>
            {% endfor %}
            </ul>
        </td>
    </tr>
{% endfor %}
</table>
```

#### for ... cycle

Alternate between values:

```html
<table>
{% for item in items %}
    <tr class="{% cycle 'odd' 'even' %}">
        <td>{{ item }}</td>
    </tr>
{% endfor %}
</table>
```

### Variable Assignment

#### with

Create a variable alias:

```html
{% with total=business.employees.count %}
    <p>The business has {{ total }} employees</p>
{% endwith %}
```

### Comment Tags

```html
<!-- Single line comment (visible in HTML) -->
{# Single line comment (NOT visible in HTML) #}

<!-- Multi-line comment -->
{% comment %}
    This comment can span
    multiple lines
    and won't be visible in the HTML output
{% endcomment %}
```

### Include Tag

Reuse template snippets:

```html
<!-- Include a template -->
{% include 'header.html' %}

<!-- Include with context -->
{% include 'item.html' with item=product %}

<!-- Include without current context -->
{% include 'modal.html' only %}
```

---

## Template Inheritance

### Why Use Inheritance?

Template inheritance allows you to:
- Define a base template with common structure
- Override specific parts in child templates
- Avoid code duplication
- Maintain consistency across pages

### Base Template (base.html)

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
    <style>
        {% block extra_css %}{% endblock %}
    </style>
</head>
<body>
    <header>
        <h1>My Website</h1>
        <nav>
            <a href="/">Home</a>
            <a href="/about/">About</a>
        </nav>
    </header>

    <main>
        {% block content %}
            <p>Default content</p>
        {% endblock %}
    </main>

    <footer>
        <p>&copy; 2024 My Website</p>
    </footer>
</body>
</html>
```

### Child Template (home.html)

```html
{% extends "base.html" %}

{% block title %}Home - My Site{% endblock %}

{% block content %}
    <h2>Welcome!</h2>
    <p>This is the home page</p>
{% endblock %}
```

### Multi-level Inheritance

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
    {% block content %}...{% endblock %}
</html>

<!-- base_posts.html -->
{% extends "base.html" %}
{% block content %}
    <div class="posts">
        {% block posts_content %}{% endblock %}
    </div>
{% endblock %}

<!-- post_list.html -->
{% extends "base_posts.html" %}
{% block posts_content %}
    {% for post in posts %}
        <h3>{{ post.title }}</h3>
    {% endfor %}
{% endblock %}
```

---

## Built-in Tags Reference

### URL Tag

Generate URLs for your views:

```html
<!-- Simple URL -->
{% url 'home' %}
<!-- Output: / -->

<!-- URL with parameters -->
{% url 'post_detail' post.id %}
<!-- Output: /post/1/ -->

<!-- Store in variable -->
{% url 'about' as about_url %}
<a href="{{ about_url }}">About</a>
```

### Static Tag

Reference static files (CSS, JavaScript, images):

```html
<!-- CSS -->
<link rel="stylesheet" href="{% static 'css/style.css' %}">

<!-- JavaScript -->
<script src="{% static 'js/script.js' %}"></script>

<!-- Image -->
<img src="{% static 'images/logo.png' %}" alt="Logo">
```

### Load Tag

Load template tags/filters from apps:

```html
{% load humanize %}
<!-- Now you can use humanize filters -->

{{ number|intcomma }}
<!-- 1000000 becomes "1,000,000" -->
```

### Autoescape Tag

Control HTML escaping:

```html
{% autoescape on %}
    {{ user_input }}  <!-- Will escape HTML -->
{% endautoescape %}

{% autoescape off %}
    {{ trusted_html|safe }}  <!-- Won't escape -->
{% endautoescape %}
```

### Spaceless Tag

Remove whitespace between HTML tags:

```html
{% spaceless %}
    <p>
        Hello World
    </p>
{% endspaceless %}
<!-- Output: <p>Hello World</p> -->
```

### Debug Tag

Output debug information (development only):

```html
{% debug %}
<!-- Displays all context variables -->
```

---

## Practical Examples

### Example 1: Blog Post List

**views.py**
```python
from django.shortcuts import render
from .models import Post

def post_list(request):
    posts = Post.objects.all().order_by('-created_at')
    context = {'posts': posts}
    return render(request, 'blog/post_list.html', context)
```

**post_list.html**
```html
{% extends 'base.html' %}

{% block title %}All Posts{% endblock %}

{% block content %}
    <h1>Blog Posts</h1>
    
    {% if posts %}
        <div class="posts">
        {% for post in posts %}
            <article class="post">
                <h2>{{ post.title }}</h2>
                <p class="meta">
                    By {{ post.author.username }} 
                    on {{ post.created_at|date:"F j, Y" }}
                </p>
                <p>{{ post.content|truncatewords:20 }}</p>
                <a href="{% url 'post_detail' post.id %}">Read More</a>
            </article>
        {% endfor %}
        </div>
    {% else %}
        <p>No posts available yet.</p>
    {% endif %}
{% endblock %}
```

### Example 2: Product Page with Filters

**views.py**
```python
def product_detail(request, id):
    product = Product.objects.get(id=id)
    reviews = product.reviews.all()
    context = {
        'product': product,
        'reviews': reviews,
        'total_reviews': reviews.count()
    }
    return render(request, 'shop/product_detail.html', context)
```

**product_detail.html**
```html
{% extends 'base.html' %}

{% block title %}{{ product.name }}{% endblock %}

{% block content %}
    <div class="product">
        <h1>{{ product.name|upper }}</h1>
        
        <p class="price">${{ product.price|floatformat:2 }}</p>
        
        <p class="description">{{ product.description }}</p>
        
        {% if product.in_stock %}
            <button>Add to Cart</button>
        {% else %}
            <p class="out-of-stock">Out of Stock</p>
        {% endif %}
        
        <div class="reviews">
            <h2>Reviews ({{ total_reviews }})</h2>
            
            {% for review in reviews %}
                <div class="review">
                    <h4>{{ review.title }}</h4>
                    <p>Rating: {{ review.rating }}/5</p>
                    <p>{{ review.content }}</p>
                </div>
            {% empty %}
                <p>No reviews yet.</p>
            {% endfor %}
        </div>
    </div>
{% endblock %}
```

### Example 3: User Profile Page

**views.py**
```python
def user_profile(request, username):
    user = User.objects.get(username=username)
    posts = user.post_set.all()
    context = {
        'profile_user': user,
        'posts': posts,
        'post_count': posts.count()
    }
    return render(request, 'profile.html', context)
```

**profile.html**
```html
{% extends 'base.html' %}

{% block title %}{{ profile_user.first_name }}'s Profile{% endblock %}

{% block content %}
    <div class="profile">
        <h1>{{ profile_user.first_name }} {{ profile_user.last_name }}</h1>
        <p class="username">@{{ profile_user.username }}</p>
        <p class="email">{{ profile_user.email|lower }}</p>
        
        {% if profile_user.is_active %}
            <span class="active-badge">Active</span>
        {% else %}
            <span class="inactive-badge">Inactive</span>
        {% endif %}
        
        <h2>Posts ({{ post_count }})</h2>
        
        {% if posts %}
            <ul class="post-list">
            {% for post in posts %}
                <li>
                    {{ forloop.counter }}. 
                    <a href="{% url 'post_detail' post.id %}">
                        {{ post.title|truncatewords:5 }}
                    </a>
                    <small>{{ post.created_at|date:"Y-m-d" }}</small>
                </li>
            {% endfor %}
            </ul>
        {% else %}
            <p>No posts from this user.</p>
        {% endif %}
    </div>
{% endblock %}
```

---

## Key Takeaways

1. **Templates** separate presentation from business logic
2. **Variables** are displayed with `{{ }}`
3. **Filters** modify output with `|` (e.g., `|lower`, `|upper`)
4. **Tags** perform logic with `{% %}` (e.g., `{% if %}`, `{% for %}`)
5. **Inheritance** reduces code duplication with `{% extends %}` and `{% block %}`
6. **Context** dictionary passes data from views to templates
7. **Built-in** tags and filters handle common tasks
8. Templates are **auto-escaped** by default for security

---

## Additional Resources

- Django Template Documentation: https://docs.djangoproject.com/en/3.2/topics/templates/
- Template Language Reference: https://docs.djangoproject.com/en/3.2/ref/templates/language/
- Built-in Tags and Filters: https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#ref-templates-builtins-tags







## Django Template Language (DTL) & Template Inheritance (Master Level)
---
## 🎯 Lesson Objectives
By the end of this guide, you will:
- ✅ Understand what DTL really is
- ✅ Master template inheritance
- ✅ Use the most important DTL tags
- ✅ Use the most common filters
- ✅ Avoid common template mistakes
- ✅ Structure templates professionally
- ✅ Write clean, scalable template code
---
## 🧠 What Is Django Template Language (DTL)?
DTL is the system Django uses to:
- Display dynamic data
- Loop through objects
- Add logic in templates
- Extend layouts
- Apply formatting
> **📌 Important:** DTL is NOT Python. It is a simplified, secure template language.
---
## 🏗️ How Templates Fit in Django Flow
```
User Request → URL Router → View → Model → Database → View → Template → HTML Response
```
**The template's job:**
- ✅ Present data attractively
- ✅ Handle user interface logic
- ❌ NOT process business logic
- ❌ NOT access databases directly
- ❌ NOT perform complex calculations
---
## 🧩 DTL Syntax Basics
Django templates use **3 core syntax elements**:
| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{ }}` | Output variable | `{{ user.name }}` |
| `{% %}` | Template tag (logic) | `{% if user.is_staff %}` |
| `{# #}` | Comment | `{# TODO: fix this #}` |
---
## 🔹 1. Displaying Variables
### Basic Variable Output
```html
<h1>{{ post.title }}</h1>
<p>{{ post.content }}</p>
```
### Accessing Nested Object Properties
```html
<!-- Accessing related object attributes -->
<p>Author: {{ post.author.username }}</p>
<p>Created: {{ post.created_at }}</p>
```
---
## 🔹 2. For Loops (Very Common)
### Basic For Loop
```html
{% for post in posts %}
    <h3>{{ post.title }}</h3>
    <p>{{ post.excerpt }}</p>
{% endfor %}
```
### For Loop with Empty Fallback
```html
{% for post in posts %}
    <p>{{ post.title }}</p>
{% empty %}
    <p>No posts available.</p>
{% endfor %}
```
### Accessing Loop Variables
```html
{% for post in posts %}
    <div>{{ forloop.counter }}. {{ post.title }}</div>
{% endfor %}
```
**Available loop variables:**
- `forloop.counter` - Current iteration (1-indexed)
- `forloop.counter0` - Current iteration (0-indexed)
- `forloop.first` - True for first item
- `forloop.last` - True for last item
---
## 🔹 3. If Conditions
### Basic If-Else
```html
{% if user.is_authenticated %}
    <p>Welcome back, {{ user.username }}!</p>
{% else %}
    <p>Please <a href="{% url 'login' %}">login</a></p>
{% endif %}
```
### Complex Conditions
```html
{% if post.status == 'published' and post.views > 100 %}
    <p>Popular Post: {{ post.title }}</p>
{% elif post.status == 'draft' %}
    <p>Draft Post (Admin only)</p>
{% else %}
    <p>Archive Post</p>
{% endif %}
```
**Boolean operators:**
- `and` - Both conditions must be true
- `or` - At least one condition true
- `not` - Negate a condition
- `in` - Check if item in collection
---
## 🔹 4. URL Tag (VERY IMPORTANT)
### Basic URL Generation
```html
<!-- Named URL -->
<a href="{% url 'blog:post_list' %}">All Posts</a>
<!-- URL with parameters -->
<a href="{% url 'blog:post_detail' post.id %}">Read</a>
```
### URL with Query Parameters
```html
<a href="{% url 'search' %}?q={{ search_term }}">Search Results</a>
```
---
## 🔹 5. CSRF Token (Security Critical)
### Form Protection
```html
<form method="POST" action="{% url 'submit_form' %}">
    {% csrf_token %}
    <input type="text" name="message" placeholder="Your message">
    <button type="submit">Submit</button>
</form>
```
> **⚠️ Important:** Always include `{% csrf_token %}` in POST forms!
---
## 🔗 TEMPLATE INHERITANCE (Critical Concept)
This is how professional Django apps are structured.
### Why Template Inheritance?
**Without Inheritance (❌ Bad)**
Every page repeats:
- Navigation bar
- Footer
- CSS links
- JavaScript includes
**With Inheritance (✅ Good)**
- One base template
- Other templates extend it
- DRY principle followed
- Easy to maintain
---
## 📁 Recommended Template Structure
```
templates/
├── base.html                 # Main site layout
├── includes/
│   ├── navbar.html          # Navigation
│   ├── footer.html          # Footer
│   └── messages.html        # Django messages
├── blog/
│   ├── base.html            # Blog-specific layout
│   ├── post_list.html       # Post listing
│   ├── post_detail.html     # Individual post
│   └── post_create.html     # Create post
└── auth/
    ├── login.html           # Login page
    └── register.html        # Registration page
```
---
### 📝 Step 1: Create Base Template
**base.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <!-- Navigation -->
    {% include "includes/navbar.html" %}
    <!-- Main Content -->
    {% block content %}
    {% endblock %}
    <!-- Footer -->
    {% include "includes/footer.html" %}
</body>
</html>
```
---
### 📝 Step 2: Extend Base Template
**blog/post_list.html**
```html
{% extends "base.html" %}
{% block title %}Blog Posts | {{ block.super }}{% endblock %}
{% block content %}
    <h1>All Posts</h1>
    {% for post in posts %}
        <article>
            <h2>{{ post.title }}</h2>
            <p>{{ post.excerpt }}</p>
        </article>
    {% empty %}
        <p>No posts available.</p>
    {% endfor %}
{% endblock %}
```
---
### 🔄 Advanced: block.super
Use `block.super` to append to parent block content:
```html
{% block title %}
    Blog | {{ block.super }}
{% endblock %}
<!-- Result: "Blog | My Site" if parent is "My Site" -->
```
---
### 📦 Include Tag (Reusable Components)
**Basic Include**
```html
{% include "includes/navbar.html" %}
```
**Include with Context**
```html
{% include "includes/pagination.html" with page_obj=page_obj %}
```
---
### 🏷️ With Tag (Temporary Variables)
```html
{% with total=posts.count %}
    <p>Total posts: {{ total }}</p>
{% endwith %}
```
---
### 📄 Static Files
```html
{% load static %}
<!-- CSS -->
<link rel="stylesheet" href="{% static 'css/style.css' %}">
<!-- JavaScript -->
<script src="{% static 'js/main.js' %}"></script>
<!-- Images -->
<img src="{% static 'images/logo.png' %}" alt="Logo">
```
---
## 🎨 MOST USED DTL FILTERS
Filters transform variable output. **Syntax:** `{{ variable|filter }}`
### Text Filters
| Filter | Example | Output |
|--------|---------|--------|
| `upper` | `{{ "hello"\|upper }}` | `HELLO` |
| `lower` | `{{ "HELLO"\|lower }}` | `hello` |
| `title` | `{{ "hello world"\|title }}` | `Hello World` |
| `truncatewords:N` | `{{ text\|truncatewords:3 }}` | Truncate to N words |
| `truncatechars:N` | `{{ text\|truncatechars:10 }}` | Truncate to N chars |
### Date/Time Filters
| Filter | Example | Output |
|--------|---------|--------|
| `date` | `{{ post.date\|date:"Y-m-d" }}` | `2023-12-25` |
| `timesince` | `{{ post.date\|timesince }}` | `2 days ago` |
| `timeuntil` | `{{ event.date\|timeuntil }}` | `3 days` |
### Utility Filters
| Filter | Example | Purpose |
|--------|---------|---------|
| `default` | `{{ value\|default:"N/A" }}` | Fallback value |
| `length` | `{{ posts\|length }}` | Get object count |
| `slice` | `{{ posts\|slice:":5" }}` | Get first 5 items |
| `safe` | `{{ html\|safe }}` | Mark as safe HTML |
### Combining Filters
```html
{{ post.title|upper|truncatewords:5 }}
```
---
## 🔒 Security & Best Practices
### Auto-Escaping is ON by Default
```html
<!-- SAFE: Auto-escaped (HTML entities shown as text) -->
<p>{{ user_comment }}</p>
<!-- DANGEROUS: Only use |safe when absolutely certain! -->
<p>{{ trusted_html|safe }}</p>
```
### ✅ DO This
- Keep templates simple and readable
- Use inheritance for consistent layouts
- Include reusable components
- Use descriptive block names
- Add comments for complex logic
- Use URL namespacing
- Escape all user input
### ❌ DON'T Do This
- Write business logic in templates
- Complex nested if/for statements
- Hardcoded URLs
- Inline JavaScript/CSS
- Database queries in templates
- Large template files (>500 lines)
---
## 🏛️ Professional Template Architecture
### Complete Example Structure
```
templates/
├── base.html
├── includes/
│   ├── navbar.html
│   ├── footer.html
│   ├── messages.html
│   └── pagination.html
├── blog/
│   ├── base.html
│   ├── post_list.html
│   ├── post_detail.html
│   └── post_create.html
└── auth/
    ├── login.html
    └── register.html
```
---
## 📚 Practice Tasks
### Beginner Level
1. Create a `base.html` with title block
2. Create a child template that extends base
3. Use `{% for %}` loop to display items
4. Use `{% if %}` to show conditional content
### Intermediate Level
1. Implement multiple inheritance levels
2. Create reusable include components
3. Use filters to format dates and text
4. Add CSRF protection to forms
### Advanced Level
1. Build a complete blog layout system
2. Create custom model managers for templates
3. Implement dynamic navigation based on user role
4. Add search functionality with highlighting
---
## 📋 DTL Master Summary
### Core Concepts
✨ `{{ }}` - Display variables  
✨ `{% %}` - Template logic (for, if, include)  
✨ Filters - Transform output  
✨ Inheritance - Extend base templates  
✨ Includes - Reusable components  
### Security
🔒 Auto-escaping by default  
🔒 CSRF tokens in forms  
🔒 Never trust user input  
🔒 Use external CSS/JS files  
### Best Practices
💡 Keep logic minimal  
💡 Use professional structure  
💡 Follow DRY principle  
💡 Maintain consistency  
💡 Write clean code  
---
## 🚀 What Comes Next?
Ready to continue your Django mastery?
### Phase 3: Forms & Validation
- Django ModelForms
- Form validation
- Custom widgets
- File uploads
### Phase 4: Authentication & Security
- User registration
- Login/logout
- Permissions system
- Password management
### Phase 5: Advanced Features
- Custom template tags
- Context processors
- Template caching
- Performance optimization
---
## 🎉 Congratulations!
If you fully understand this guide, you can now:
- ✅ Build professional Django templates
- ✅ Use template inheritance properly
- ✅ Implement security best practices
- ✅ Create maintainable template code
- ✅ Work with forms and dynamic content
**You are now at intermediate level in Django templates!**
Ready for Phase 3? Let's master Django Forms next! 🚀




# 📘 PHASE 2 – DYNAMIC URLS (MASTER LEVEL)
## Django Dynamic URLs, URL Converters & Professional Routing

---

## 🎯 Lesson Objectives

By the end of this comprehensive guide, you will:

- ✅ Master dynamic URL routing patterns
- ✅ Use all URL converters effectively
- ✅ Implement slug-based SEO-friendly URLs
- ✅ Handle multiple parameters professionally
- ✅ Use URL namespaces for scalable apps
- ✅ Apply security best practices
- ✅ Design RESTful URL structures
- ✅ Avoid common routing pitfalls
- ✅ Build maintainable URL configurations

---

## 🧠 What Are Dynamic URLs?

Dynamic URLs are **data-driven routes** that change based on database content, user input, or application state.

### Static vs Dynamic URLs

| Type | Example | Use Case |
|------|---------|----------|
| **Static** | `/about/` | Fixed pages (contact, privacy) |
| **Dynamic** | `/post/5/` | Database-driven content |
| **Dynamic** | `/user/john/` | User-generated content |
| **Dynamic** | `/blog/2024/01/` | Date-based archives |

> **💡 Key Concept:** Dynamic URLs make your app **scalable** and **SEO-friendly**

---

## 🏗️ How Dynamic URLs Work in Django

```
User Request → URLconf → View Function → Database/Model → Template → Response
                    ↓
            URL Parameters Extracted
```

### The Complete Flow:

1. **Browser** sends request to `/blog/post/5/`
2. **URLconf** matches pattern `post/<int:id>/`
3. **View** receives `id=5` as parameter
4. **Model** queries `Post.objects.get(id=5)`
5. **Template** renders post content
6. **Response** sent back to browser

---

## 🔹 1. Basic Dynamic URL Example

### Step 1: URL Configuration

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'  # Always use app_name for namespaces

urlpatterns = [
    path('post/<int:id>/', views.post_detail, name='post_detail'),
]
```

### Step 2: View Function

```python
# blog/views.py
from django.shortcuts import render, get_object_or_404
from .models import Post

def post_detail(request, id):
    """
    Display individual post by ID
    """
    post = get_object_or_404(Post, id=id)
    return render(request, 'blog/post_detail.html', {
        'post': post,
    })
```

### Step 3: Template Usage

```html
<!-- blog/templates/blog/post_detail.html -->
{% extends "base.html" %}

{% block title %}{{ post.title }} | {{ block.super }}{% endblock %}

{% block content %}
<article class="post-detail">
    <h1>{{ post.title }}</h1>
    <div class="post-meta">
        <span>By {{ post.author.username }}</span>
        <span>{{ post.created_at|date:"F j, Y" }}</span>
    </div>
    <div class="post-content">
        {{ post.content|safe }}
    </div>
</article>
{% endblock %}
```

### Step 4: Creating Links (CRITICAL)

```html
<!-- NEVER hardcode URLs -->
<!-- ❌ WRONG -->
<a href="/blog/post/{{ post.id }}/">Read More</a>

<!-- ✅ CORRECT -->
<a href="{% url 'blog:post_detail' post.id %}">Read More</a>
```

---

## 🔢 2. URL Converters (Path Converters)

Django provides **5 built-in converters** for different data types:

| Converter | Description | Example URL | Python Type |
|-----------|-------------|-------------|-------------|
| `int` | Positive integers | `/post/5/` | `int` |
| `str` | Any string except `/` | `/user/john/` | `str` |
| `slug` | Letters, numbers, hyphens | `/post/hello-world/` | `str` |
| `uuid` | UUID format | `/order/123e4567-e89b/` | `UUID` |
| `path` | Full path with slashes | `/files/docs/readme.txt` | `str` |

### Integer Converter Examples

```python
# urls.py
urlpatterns = [
    path('post/<int:id>/', views.post_detail, name='post_detail'),
    path('category/<int:cat_id>/posts/', views.category_posts, name='category_posts'),
    path('page/<int:page_num>/', views.paginated_list, name='paginated_list'),
]
```

### String Converter Examples

```python
# urls.py
urlpatterns = [
    path('user/<str:username>/', views.user_profile, name='user_profile'),
    path('tag/<str:tag_name>/', views.tag_posts, name='tag_posts'),
]
```

---

## 🧱 3. Slug-Based URLs (SEO Best Practice)

Slugs create **human-readable, SEO-friendly URLs** instead of numeric IDs.

### Model with Slug Field

```python
# blog/models.py
from django.db import models
from django.utils.text import slugify

class Post(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True, blank=True)
    content = models.TextField()
    author = models.ForeignKey('auth.User', on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)

    def __str__(self):
        return self.title
```

### Slug URL Pattern

```python
# blog/urls.py
urlpatterns = [
    path('post/<slug:slug>/', views.post_detail, name='post_detail'),
]
```

### Slug View Function

```python
# blog/views.py
def post_detail(request, slug):
    """
    Display post by slug (SEO-friendly)
    """
    post = get_object_or_404(Post, slug=slug)
    return render(request, 'blog/post_detail.html', {
        'post': post,
    })
```

### Handling Slug Changes

```python
# Advanced: Handle old slugs with redirects
def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug)
    
    # Check if slug matches current post slug
    if post.slug != slug:
        # Redirect to correct URL
        return redirect('blog:post_detail', slug=post.slug, permanent=True)
    
    return render(request, 'blog/post_detail.html', {'post': post})
```

---

## 📅 4. Multiple Dynamic Parameters

Handle complex routing with multiple parameters.

### Date-Based Archives

```python
# urls.py
urlpatterns = [
    path('archive/<int:year>/', views.year_archive, name='year_archive'),
    path('archive/<int:year>/<int:month>/', views.month_archive, name='month_archive'),
    path('archive/<int:year>/<int:month>/<int:day>/', views.day_archive, name='day_archive'),
]
```

```python
# views.py
from django.shortcuts import render
from .models import Post

def year_archive(request, year):
    posts = Post.objects.filter(created_at__year=year)
    return render(request, 'blog/archive.html', {
        'posts': posts,
        'year': year,
        'archive_type': 'year'
    })

def month_archive(request, year, month):
    posts = Post.objects.filter(
        created_at__year=year,
        created_at__month=month
    )
    return render(request, 'blog/archive.html', {
        'posts': posts,
        'year': year,
        'month': month,
        'archive_type': 'month'
    })
```

### Complex Parameter Example

```python
# urls.py - Product catalog
urlpatterns = [
    path('products/<slug:category>/<slug:product>/', views.product_detail, name='product_detail'),
]
```

```python
# views.py
def product_detail(request, category, product):
    product_obj = get_object_or_404(
        Product.objects.select_related('category'),
        slug=product,
        category__slug=category
    )
    return render(request, 'shop/product_detail.html', {
        'product': product_obj,
    })
```

---

## 🏢 5. URL Namespaces (Multiple Apps)

Namespaces prevent URL name conflicts in large projects.

### Project Structure

```
myproject/
├── blog/
│   ├── urls.py
│   └── views.py
├── shop/
│   ├── urls.py
│   └── views.py
└── myproject/
    └── urls.py
```

### App-Level URLs with Namespaces

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'  # Namespace declaration

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<slug:slug>/', views.post_detail, name='post_detail'),
    path('category/<slug:slug>/', views.category_detail, name='category_detail'),
]
```

```python
# shop/urls.py
from django.urls import path
from . import views

app_name = 'shop'

urlpatterns = [
    path('', views.product_list, name='product_list'),
    path('product/<slug:slug>/', views.product_detail, name='product_detail'),
    path('category/<slug:slug>/', views.category_detail, name='category_detail'),
]
```

### Project-Level URL Configuration

```python
# myproject/urls.py
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),      # blog:post_list
    path('shop/', include('shop.urls')),      # shop:product_list
    path('accounts/', include('django.contrib.auth.urls')),
]
```

### Template Usage with Namespaces

```html
<!-- Correct usage -->
<a href="{% url 'blog:post_detail' post.slug %}">Read Post</a>
<a href="{% url 'shop:product_detail' product.slug %}">View Product</a>

<!-- Both apps have 'category_detail' but no conflict due to namespaces -->
<a href="{% url 'blog:category_detail' category.slug %}">Blog Category</a>
<a href="{% url 'shop:category_detail' category.slug %}">Shop Category</a>
```

---

## 🔐 6. Security Best Practices

### Always Use get_object_or_404

```python
# ❌ DANGEROUS - Can cause 500 errors
def post_detail(request, id):
    try:
        post = Post.objects.get(id=id)
    except Post.DoesNotExist:
        raise Http404("Post not found")
    return render(request, 'blog/post_detail.html', {'post': post})

# ✅ SAFE - Returns proper 404
def post_detail(request, id):
    post = get_object_or_404(Post, id=id)
    return render(request, 'blog/post_detail.html', {'post': post})
```

### Validate Parameters

```python
# views.py
from django.core.exceptions import ValidationError

def post_detail(request, slug):
    # Additional validation
    if not slug or len(slug) > 100:
        raise Http404()
    
    post = get_object_or_404(Post, slug=slug, status='published')
    return render(request, 'blog/post_detail.html', {'post': post})
```

### Prevent Information Disclosure

```python
# Only show published posts
def post_detail(request, slug):
    post = get_object_or_404(
        Post,
        slug=slug,
        status='published',  # Security filter
        published_at__lte=timezone.now()  # Future posts hidden
    )
    return render(request, 'blog/post_detail.html', {'post': post})
```

---

## 🏆 7. Professional URL Design Patterns

### RESTful URL Structure

```
# Blog App
GET    /posts/              # List all posts
GET    /posts/<id>/         # Get specific post
POST   /posts/create/       # Create new post
GET    /posts/<id>/edit/    # Edit form
PUT    /posts/<id>/         # Update post
DELETE /posts/<id>/         # Delete post

# Shop App
GET    /products/           # List products
GET    /products/<id>/      # Product details
POST   /cart/add/<id>/      # Add to cart
GET    /orders/<id>/        # Order details
```

### URL Design Best Practices

```python
# ✅ GOOD URL patterns
urlpatterns = [
    # Use slugs for public content
    path('blog/<slug:slug>/', views.post_detail, name='post_detail'),
    
    # Use IDs for admin actions
    path('admin/posts/<int:id>/edit/', views.post_edit, name='post_edit'),
    
    # Consistent naming
    path('users/<str:username>/', views.user_profile, name='user_profile'),
    path('tags/<slug:tag>/', views.tag_detail, name='tag_detail'),
    
    # Date archives
    path('archive/<int:year>/<int:month>/', views.month_archive, name='month_archive'),
]
```

---

## 🚨 8. Common Mistakes & How to Fix Them

### ❌ Mistake 1: Hardcoded URLs

```html
<!-- WRONG -->
<a href="/blog/post/1/">Post 1</a>

<!-- CORRECT -->
<a href="{% url 'blog:post_detail' post.id %}">Post 1</a>
```

### ❌ Mistake 2: No Namespaces

```python
# WRONG - Conflicts possible
urlpatterns = [
    path('category/<slug:slug>/', blog_views.category_detail, name='category_detail'),
    path('category/<slug:slug>/', shop_views.category_detail, name='category_detail'),
]

# CORRECT - Namespaced
# blog/urls.py
app_name = 'blog'
urlpatterns = [path('category/<slug:slug>/', views.category_detail, name='category_detail')]

# shop/urls.py  
app_name = 'shop'
urlpatterns = [path('category/<slug:slug>/', views.category_detail, name='category_detail')]
```

### ❌ Mistake 3: Using .get() Without Try/Except

```python
# WRONG - Server error on invalid ID
def post_detail(request, id):
    post = Post.objects.get(id=id)
    return render(request, 'post_detail.html', {'post': post})

# CORRECT - Proper 404
def post_detail(request, id):
    post = get_object_or_404(Post, id=id)
    return render(request, 'post_detail.html', {'post': post})
```

### ❌ Mistake 4: Inconsistent URL Patterns

```python
# WRONG - Inconsistent
path('post-detail/<int:id>/', views.post_detail)
path('blog/post/<slug:slug>/', views.post_detail)

# CORRECT - Consistent slugs
path('blog/<slug:slug>/', views.post_detail, name='post_detail')
```

---

## 🔧 9. Advanced URL Techniques

### Custom Path Converters

```python
# converters.py
from django.urls import converters

class FourDigitYearConverter(converters.IntConverter):
    regex = r'[0-9]{4}'
    
    def to_python(self, value):
        return int(value)
    
    def to_url(self, value):
        return f'{value:04d}'

# urls.py
from .converters import FourDigitYearConverter

register_converter(FourDigitYearConverter, 'yyyy')

urlpatterns = [
    path('archive/<yyyy:year>/', views.year_archive, name='year_archive'),
]
```

### URL Redirects for SEO

```python
# views.py
from django.shortcuts import redirect

def old_post_detail(request, id):
    post = get_object_or_404(Post, id=id)
    return redirect('blog:post_detail', slug=post.slug, permanent=True)
```

### Query Parameters with Dynamic URLs

```python
# urls.py
path('search/', views.search_results, name='search_results'),

# views.py
def search_results(request):
    query = request.GET.get('q', '')
    category = request.GET.get('category', '')
    
    posts = Post.objects.filter(title__icontains=query)
    if category:
        posts = posts.filter(category__slug=category)
    
    return render(request, 'blog/search.html', {
        'posts': posts,
        'query': query,
        'category': category,
    })
```

---

## 🧪 10. Practice Tasks

### Beginner Level 🟢

1. **Basic Dynamic URL**
   - Create a post detail view with `<int:id>`
   - Add template with post display
   - Create links using `{% url %}` tag

2. **URL Converters**
   - Add user profile page with `<str:username>`
   - Add category page with `<slug:category>`
   - Test different converter types

### Intermediate Level 🟡

3. **Slug Implementation**
   - Add slug field to Post model
   - Implement auto-slug generation
   - Update URLs to use slugs instead of IDs
   - Handle slug redirects

4. **Multiple Parameters**
   - Create date archive: `/blog/2024/01/`
   - Add month/year filtering in views
   - Create archive template

5. **Namespaces**
   - Split project into blog and shop apps
   - Add namespaces to both apps
   - Update all templates to use namespaced URLs

### Advanced Level 🔴

6. **RESTful URLs**
   - Implement full CRUD URLs for posts
   - Add proper HTTP methods handling
   - Create admin URLs for management

7. **Security Implementation**
   - Add permission checks to views
   - Implement soft deletes with URL handling
   - Add rate limiting for dynamic URLs

8. **Custom Converters**
   - Create custom converter for phone numbers
   - Add validation in converter
   - Use in user profile URLs

---

## 🏗️ 11. Professional URL Architecture

### Complete Blog URL Structure

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # Public pages
    path('', views.post_list, name='post_list'),
    path('post/<slug:slug>/', views.post_detail, name='post_detail'),
    path('category/<slug:slug>/', views.category_detail, name='category_detail'),
    path('tag/<slug:slug>/', views.tag_detail, name='tag_detail'),
    path('author/<str:username>/', views.author_posts, name='author_posts'),
    
    # Archives
    path('archive/<int:year>/', views.year_archive, name='year_archive'),
    path('archive/<int:year>/<int:month>/', views.month_archive, name='month_archive'),
    
    # Search
    path('search/', views.search, name='search'),
    
    # Admin (protected)
    path('admin/create/', views.post_create, name='post_create'),
    path('admin/<int:id>/edit/', views.post_edit, name='post_edit'),
    path('admin/<int:id>/delete/', views.post_delete, name='post_delete'),
]
```

### URL Testing Strategy

```python
# tests.py
from django.test import TestCase
from django.urls import reverse

class URLTests(TestCase):
    def test_post_detail_url(self):
        post = Post.objects.create(title="Test Post", slug="test-post")
        url = reverse('blog:post_detail', kwargs={'slug': post.slug})
        self.assertEqual(url, '/blog/post/test-post/')
        
    def test_invalid_slug_404(self):
        response = self.client.get('/blog/post/nonexistent/')
        self.assertEqual(response.status_code, 404)
```

---

## 📊 12. Performance Considerations

### Database Optimization

```python
# views.py - Optimized queries
def post_detail(request, slug):
    post = get_object_or_404(
        Post.objects.select_related('author', 'category'),  # Join related tables
        slug=slug,
        status='published'
    )
    return render(request, 'blog/post_detail.html', {'post': post})
```

### URL Caching

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/',
    }
}

# views.py
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # Cache for 15 minutes
def post_detail(request, slug):
    # View logic here
    pass
```

---

## 🎯 Dynamic URL Mastery Summary

### ✅ What You Now Master:

- **URL Converters**: `int`, `str`, `slug`, `uuid`, `path`
- **Slug Implementation**: SEO-friendly URLs with auto-generation
- **Multiple Parameters**: Complex routing patterns
- **Namespaces**: Conflict-free URL organization
- **Security**: `get_object_or_404`, validation, permissions
- **RESTful Design**: Professional URL structures
- **Best Practices**: Performance, testing, maintainability

### 🚀 Next Level Topics:

- **Class-Based Views** with dynamic URLs
- **Django Forms** integration
- **Authentication** & user management
- **API Design** with Django REST Framework
- **Advanced Routing** patterns

---

## 🏆 Congratulations!

If you understand all concepts in this guide, you have achieved **intermediate Django routing mastery**! 

You can now build:
- ✅ Scalable multi-app Django projects
- ✅ SEO-friendly URL structures  
- ✅ Secure dynamic routing
- ✅ Professional URL architectures
- ✅ RESTful API designs

**Ready for Phase 3: Django Forms & Validation?** 🔥

Just say the word and we'll dive into creating interactive web forms! 🚀



# Lesson 8: Static Files (Comprehensive)

## Lesson Objective
Learn how Django handles CSS, JavaScript, and images.

---

## 1. What Are Static Files?
Assets that don't change dynamically.

Examples:
- CSS
- JS
- Images

---

## 2. Static Directory Structure

```
static/
 └── app_name/
     └── style.css
```

---

## 3. Loading Static Files

```html
{% load static %}
<link rel="stylesheet" href="{% static 'app_name/style.css' %}">
```

---

## 4. Settings Configuration

```python
STATIC_URL = '/static/'
```

---

## 5. Development vs Production
- Dev: Django serves static files
- Prod: Web server serves static files

---

## Common Mistakes
- Forgetting `{% load static %}`
- Wrong path

---

## Practice Tasks
1. Add CSS
2. Add image
3. Verify static loading

---
End of Lesson 8



# Lesson 9: Template Inheritance (Comprehensive)

## Lesson Objective
Understand how to reuse layouts using Django template inheritance.

---

## 1. Why Template Inheritance?
Avoid duplication and keep layouts consistent.

---

## 2. Base Template

```html
<html>
<body>
{% block content %}{% endblock %}
</body>
</html>
```

---

## 3. Extending Base Template

```html
{% extends 'base.html' %}

{% block content %}
<h1>Home</h1>
{% endblock %}
```

---

## 4. Block Rules
- Blocks can be overridden
- Names must match

---

## 5. Best Practices
- One main base template
- Minimal logic

---

## Common Mistakes
- Forgetting `{% extends %}`
- Duplicate blocks

---

## Practice Tasks
1. Create base layout
2. Extend it
3. Add navigation

---
End of Lesson 9

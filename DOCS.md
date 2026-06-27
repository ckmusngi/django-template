# Documentation

## Table of Contents

1. [How Authentication Works](#1-how-authentication-works)
2. [File Structure](#2-file-structure)
3. [Where to Edit Things](#3-where-to-edit-things)
4. [Adding New Pages](#4-adding-new-pages)
5. [Adding a New App](#5-adding-a-new-app)
6. [Template System](#6-template-system)
7. [Middleware](#7-middleware)
8. [Environment Variables](#8-environment-variables)

---

## 1. How Authentication Works

This project uses **Django's built-in session-based authentication**. No JWT, no third-party library.

### Login Flow

```
User submits login form (POST /users/login/)
        │
        ▼
UserLoginView (extends Django's LoginView)
        │
        ▼
AuthenticationForm validates username + password
against the auth_user table in PostgreSQL
        │
   ┌────┴────┐
 Fail      Pass
   │          │
   ▼          ▼
Re-render   django.contrib.auth.login()
form with   creates a session in the DB
errors      sets sessionid cookie in browser
                │
                ▼
            Redirect to /users/dashboard/
```

### Logout Flow

```
User clicks Logout (POST /users/logout/)
        │
        ▼
Django's LogoutView
        │
        ▼
Session deleted from DB
sessionid cookie cleared
        │
        ▼
Redirect to /users/login/
```

### Protecting a View

Use `@login_required` on any view that requires authentication:

```python
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    return render(request, 'my_template.html')
```

If an unauthenticated user visits that URL, Django redirects them to `LOGIN_URL` (`/users/login/`) with a `?next=` parameter so they land back on the right page after logging in.

For class-based views, use `LoginRequiredMixin`:

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView

class MyView(LoginRequiredMixin, TemplateView):
    template_name = 'my_template.html'
```

### Session Storage

Sessions are stored in the PostgreSQL database (`django_session` table). The browser holds only a `sessionid` cookie — no sensitive data is stored client-side.

### Back-Button Prevention After Logout

Two layers handle this:

| Layer | File | What it does |
|-------|------|--------------|
| HTTP headers | `config/middleware.py` | Sets `Cache-Control: no-store` on all authenticated responses so the browser never caches them |
| JavaScript | `templates/base.html` | Listens for `pageshow` with `e.persisted` (bfcache restore) and forces a reload, which triggers `@login_required` |

---

## 2. File Structure

```
django-template/
│
├── config/                   # Project-level configuration
│   ├── settings.py           # All Django settings
│   ├── urls.py               # Root URL dispatcher
│   ├── middleware.py         # Custom middleware (NoCacheAuthMiddleware)
│   ├── wsgi.py               # WSGI entry point
│   └── asgi.py               # ASGI entry point
│
├── users/                    # Authentication app
│   ├── views.py              # Login view, dashboard view
│   ├── urls.py               # URL patterns for /users/*
│   ├── models.py             # Custom models (empty — uses Django's User)
│   ├── admin.py              # Admin registrations
│   ├── apps.py               # App config
│   └── templates/
│       └── users/
│           ├── login.html    # Login page
│           └── dashboard.html
│
├── templates/                # Project-wide base templates
│   ├── base.html             # Layout for authenticated pages
│   ├── base_auth.html        # Layout for login/register pages
│   └── components/
│       ├── navbar.html       # Top navigation bar
│       ├── sidebar.html      # Left sidebar
│       └── footer.html       # Page footer
│
├── static/
│   └── css/
│       └── style.css         # All custom styles
│
├── .env                      # Environment variables (never commit this)
├── Pipfile                   # Dependencies
├── manage.py
├── README.md
└── DOCS.md
```

---

## 3. Where to Edit Things

### Change the app name / branding

Search for `MyApp` across these files and replace:

| File | What to change |
|------|---------------|
| `templates/base.html` | Default `<title>` tag |
| `templates/base_auth.html` | Default `<title>` tag |
| `templates/components/sidebar.html` | Brand name and icon |
| `templates/components/footer.html` | Footer copyright text |
| `users/templates/users/login.html` | Page `<title>` block |

### Change login / logout redirect destinations

`config/settings.py`:

```python
LOGIN_URL = 'login'              # Where @login_required redirects unauthenticated users
LOGIN_REDIRECT_URL = 'dashboard' # Where to go after successful login
LOGOUT_REDIRECT_URL = 'login'    # Where to go after logout
```

Replace the value with any URL name defined in `urls.py`.

### Change the login URL prefix

`config/urls.py`:

```python
path('users/', include('users.urls')),  # Change 'users/' to anything
```

All routes under the `users` app will move with it.

### Add or change sidebar links

`templates/components/sidebar.html` — add a new `<a>` with the `sidebar-link` class:

```html
<a href="{% url 'your-url-name' %}" class="sidebar-link">
    <i class="bi bi-your-icon"></i>
    Label
</a>
```

Mark the active link by adding the `active` class. Browse icons at
[icons.getbootstrap.com](https://icons.getbootstrap.com).

### Change styles

`static/css/style.css` — CSS variables at the top control the main theme:

```css
:root {
    --sidebar-width: 240px;
    --sidebar-bg: #0f172a;      /* Sidebar background */
    --sidebar-hover: #1e293b;   /* Sidebar hover state */
    --sidebar-active: #3b82f6;  /* Active link + brand icon color */
    --body-bg: #f8fafc;         /* Main content background */
}
```

### Change password validation rules

`config/settings.py` → `AUTH_PASSWORD_VALIDATORS`. Remove or add validators:

```python
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
     'OPTIONS': {'min_length': 8}},
]
```

---

## 4. Adding New Pages

### Step 1 — Create the view

`users/views.py`:

```python
@login_required
def my_page(request):
    return render(request, 'users/my_page.html')
```

### Step 2 — Register the URL

`users/urls.py`:

```python
from .views import UserLoginView, dashboard, my_page

urlpatterns = [
    ...
    path('my-page/', my_page, name='my-page'),
]
```

### Step 3 — Create the template

`users/templates/users/my_page.html`:

```html
{% extends 'base.html' %}

{% block title %}My Page — MyApp{% endblock %}

{% block content %}
<div class="mb-4">
    <h4 class="fw-bold">My Page</h4>
</div>

<!-- your content here -->
{% endblock %}
```

### Step 4 — Link it in the sidebar

`templates/components/sidebar.html`:

```html
<a href="{% url 'my-page' %}" class="sidebar-link">
    <i class="bi bi-file-text"></i>
    My Page
</a>
```

---

## 5. Adding a New App

```bash
python manage.py startapp myapp
```

Register it in `config/settings.py`:

```python
INSTALLED_APPS = [
    ...
    'users',
    'myapp',   # add here
]
```

Wire up its URLs in `config/urls.py`:

```python
path('myapp/', include('myapp.urls')),
```

Create `myapp/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='myapp-index'),
]
```

---

## 6. Template System

Two base templates exist so auth pages don't show the sidebar:

| Template | Has sidebar? | Use for |
|----------|-------------|---------|
| `base.html` | Yes | Dashboard and all authenticated pages |
| `base_auth.html` | No | Login, register, password reset |

### Blocks available in `base.html`

| Block | Purpose |
|-------|---------|
| `{% block title %}` | Browser tab title |
| `{% block content %}` | Main page content |
| `{% block extra_css %}` | Page-specific CSS |
| `{% block extra_js %}` | Page-specific JavaScript |

### Accessing the logged-in user in templates

Django makes `{{ user }}` available in every template automatically:

```html
{{ user.username }}
{{ user.email }}
{{ user.is_staff }}
{% if user.is_authenticated %} ... {% endif %}
```

---

## 7. Middleware

`config/middleware.py` — `NoCacheAuthMiddleware`

Runs on every request after `AuthenticationMiddleware` (so `request.user` is available).
If the user is authenticated, it sets:

```
Cache-Control: no-store, no-cache, must-revalidate, max-age=0
Pragma: no-cache
Expires: 0
```

This prevents the browser from storing authenticated pages in its HTTP cache.
Combined with the `pageshow` JavaScript in `base.html`, it also prevents bfcache restores after logout.

To add more middleware, append to `MIDDLEWARE` in `config/settings.py`. Order matters — place it after `AuthenticationMiddleware` if it needs `request.user`.

---

## 8. Environment Variables

All sensitive config lives in `.env` at the project root. Never commit this file.

| Variable | Description |
|----------|-------------|
| `DEBUG` | `true` for development, `false` for production |
| `SECRET_KEY` | Django secret key — keep this private |
| `DB_HOST` | Database host (`localhost` or remote IP/URL) |
| `DB_NAME` | PostgreSQL database name |
| `DB_USER` | PostgreSQL username |
| `DB_PASSWORD` | PostgreSQL password |
| `DB_PORT` | PostgreSQL port (default `5432`) |

Loaded in `config/settings.py` via:

```python
from dotenv import load_dotenv
load_dotenv(os.path.join(BASE_DIR, '.env'))
```

Values are read with `os.getenv('KEY')`.

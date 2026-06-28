# Django Boilerplate

A modern Django starter template with session-based authentication, a dark sidebar layout, and PostgreSQL support.

## Stack

- **Django 6** — web framework
- **PostgreSQL** — database (`psycopg2`)
- **Bootstrap 5.3** + **Bootstrap Icons** — UI
- **python-dotenv** — environment variable loading
- **Pipenv** — dependency management

## Project Structure

```
django-template/
├── config/               # Project settings, URLs, WSGI/ASGI
│   ├── settings.py
│   ├── urls.py
│   └── middleware.py     # NoCacheAuthMiddleware
├── users/                # Auth app (login, signup, logout, dashboard)
│   ├── views.py
│   ├── urls.py
│   └── templates/users/
│       ├── login.html
│       ├── signup.html
│       └── dashboard.html
├── templates/            # Base templates and components
│   ├── base.html         # Authenticated layout (sidebar + topbar)
│   ├── base_auth.html    # Minimal layout for auth pages
│   └── components/
│       ├── navbar.html
│       ├── sidebar.html
│       └── footer.html
├── static/
│   └── css/style.css
├── .env
├── Pipfile
└── manage.py
```

## Getting Started

### 1. Clone and install dependencies

```bash
git clone <repo-url>
cd django-template
pipenv install
pipenv shell
pipenv sync
```

### 2. Configure environment variables

Copy `.env` and fill in your values:

```env
DEBUG=true
SECRET_KEY=your-secret-key

DB_HOST=localhost
DB_NAME=your_db_name
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_PORT=5432
```

### 3. Set up the database

```bash
python manage.py migrate
python manage.py createsuperuser
```

### 4. Run the development server

```bash
python manage.py runserver
```

Visit `http://127.0.0.1:8000/users/login/`.

## URL Routes

| URL | Name | Description |
|-----|------|-------------|
| `/users/login/` | `login` | Login page |
| `/users/signup/` | `signup` | Registration page |
| `/users/logout/` | `logout` | Logout (POST) |
| `/users/dashboard/` | `dashboard` | Main dashboard (requires login) |
| `/admin/` | — | Django admin panel |

## Templates

| Template | Extends | Used for |
|----------|---------|----------|
| `base.html` | — | All authenticated pages |
| `base_auth.html` | — | Login, signup, and other auth pages |
| `users/login.html` | `base_auth.html` | Login form |
| `users/signup.html` | `base_auth.html` | Registration form |
| `users/dashboard.html` | `base.html` | Dashboard |

To add a new authenticated page, extend `base.html`:

```html
{% extends 'base.html' %}

{% block title %}My Page — MyApp{% endblock %}

{% block content %}
  <!-- your content -->
{% endblock %}
```

## Production Checklist

- Set `DEBUG=false` in `.env`
- Set a strong `SECRET_KEY`
- Add the following to `settings.py`:
  ```python
  SESSION_COOKIE_SECURE = True
  CSRF_COOKIE_SECURE = True
  ALLOWED_HOSTS = ['yourdomain.com']
  ```
- Run `python manage.py collectstatic`

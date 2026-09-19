---
tags: [coding, python, django, web]
type: cheatsheet
source: compiled reference (Django)
last-verified: 2026-09-03
---

# Django

## Up
- [[Python]]

Django is a batteries-included Python web framework — ORM, migrations, admin, auth, forms, templating — following the **MVT** (Model-View-Template) pattern. Great for full apps fast. Install: `pip install django`. Compare with [[Flask]]/[[FastAPI]] (lighter, API-first).

---

## Project Layout

```bash
django-admin startproject mysite         # project (settings, urls, wsgi/asgi)
cd mysite && python manage.py startapp blog   # an "app" (models, views)
```

```text
mysite/
├── manage.py               # CLI entry point
├── mysite/settings.py      # config, INSTALLED_APPS, DB, middleware
├── mysite/urls.py          # root URL routing
└── blog/                   # an app
    ├── models.py  views.py  urls.py  admin.py
    └── migrations/  templates/
```

Register the app in `settings.py` → `INSTALLED_APPS = [..., "blog"]`.

---

## Models & Migrations

```python
# blog/models.py
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()
    author = models.ForeignKey("auth.User", on_delete=models.CASCADE, related_name="posts")
    published = models.BooleanField(default=False)
    created = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["-created"]

    def __str__(self): return self.title
```

```bash
python manage.py makemigrations       # generate migration from model changes
python manage.py migrate              # apply to DB
python manage.py sqlmigrate blog 0001 # preview SQL
```

---

## ORM Queries

```python
Post.objects.all()
Post.objects.filter(published=True).exclude(title="")
Post.objects.get(pk=1)                         # raises if 0 or >1
Post.objects.filter(title__icontains="django").order_by("-created")[:10]
Post.objects.filter(author__username="ada")    # traverse FK
Post.objects.create(title="x", body="y", author=user)
p.save(); p.delete()

# aggregation / annotation
from django.db.models import Count, Q
User.objects.annotate(n=Count("posts")).filter(n__gt=2)
Post.objects.filter(Q(published=True) | Q(author=user))

# avoid N+1
Post.objects.select_related("author")          # FK join
Post.objects.prefetch_related("tags")          # M2M / reverse FK
```

Field lookups: `__gt __gte __lt __in __icontains __startswith __isnull __range __date`.

---

## Views & URLs

```python
# blog/views.py — function-based
from django.shortcuts import render, get_object_or_404, redirect

def post_list(request):
    posts = Post.objects.filter(published=True)
    return render(request, "blog/list.html", {"posts": posts})

def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, "blog/detail.html", {"post": post})

# class-based views (less boilerplate)
from django.views.generic import ListView, DetailView, CreateView
class PostList(ListView): model = Post; template_name = "blog/list.html"
```

```python
# blog/urls.py
from django.urls import path
from . import views
urlpatterns = [
    path("", views.post_list, name="post_list"),
    path("<int:pk>/", views.post_detail, name="post_detail"),
]
# mysite/urls.py: path("blog/", include("blog.urls"))
```

---

## Templates (DTL)

```django
{% extends "base.html" %}
{% block content %}
  {% for post in posts %}
    <a href="{% url 'post_detail' post.pk %}">{{ post.title }}</a>
    <p>{{ post.body|truncatewords:30 }}</p>
  {% empty %}
    <p>No posts.</p>
  {% endfor %}
{% endblock %}
```

Filters: `|date`, `|length`, `|default`, `|safe`, `|truncatewords`. Tags: `{% if %}`, `{% for %}`, `{% url %}`, `{% csrf_token %}` (required in POST forms).

---

## Admin, Auth, Forms

```python
# blog/admin.py
from django.contrib import admin
from .models import Post
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ("title", "author", "published", "created")
    list_filter = ("published",); search_fields = ("title",)
```
```bash
python manage.py createsuperuser        # then /admin/
```
- **Auth** built in: `request.user`, `@login_required`, `User`/permissions/groups.
- **Forms/ModelForms** validate + render HTML; **messages**, **sessions**, **middleware** included.

---

## manage.py essentials

```bash
python manage.py runserver              # dev server
python manage.py shell                  # ORM REPL (shell_plus with django-extensions)
python manage.py createsuperuser
python manage.py collectstatic          # gather static files for prod
python manage.py test                   # run tests (see [[pytest]] + pytest-django)
```

---

## Tips
- Use **class-based generic views** for CRUD; function views for custom logic.
- Fix **N+1** with `select_related` (FK) / `prefetch_related` (M2M).
- Keep secrets/DB config in env (via [[Pydantic]]-settings or `django-environ`); never commit `SECRET_KEY`, `DEBUG=True` in prod.
- For **REST APIs**, add **Django REST Framework** (serializers, viewsets, auth); async views supported for I/O ([[asyncio]]).
- Test with **pytest-django** (`@pytest.mark.django_db`) — see [[pytest]].
- Production: `gunicorn`/`uvicorn` (ASGI) + WhiteNoise/nginx for static.

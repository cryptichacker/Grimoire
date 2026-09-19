---
tags: [coding, python, web, flask]
type: cheatsheet
source: compiled reference (Flask)
last-verified: 2026-08-27
---

# Flask

## Up
- [[Python]]

Flask is a lightweight WSGI web micro-framework. Minimal core + extensions. Install: `pip install flask`. Related: [[FastAPI]] (async, typed) for APIs.

---

## Minimal App

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, World"

if __name__ == "__main__":
    app.run(debug=True, port=5000)     # dev server only
```

```bash
flask --app app run --debug            # preferred CLI (auto-reload, debugger)
flask --app app run --host 0.0.0.0 --port 8000
```

---

## Routing

```python
@app.route("/about")
def about(): ...

@app.route("/users/<int:user_id>")        # typed URL param
def user(user_id): return f"User {user_id}"
# converters: <string:x> (default), <int:x>, <float:x>, <path:x>, <uuid:x>

@app.route("/items", methods=["GET", "POST"])
def items(): ...

@app.get("/ping")                          # shortcuts (Flask 2+)
@app.post("/submit")
def submit(): ...

url_for("user", user_id=5)                 # → "/users/5"  (reverse routing)
```

---

## Request Data

```python
from flask import request

request.args.get("q")                 # query string ?q=...
request.args.get("page", 1, type=int)
request.form.get("username")          # form-encoded POST body
request.json                          # parsed JSON body (or get_json())
request.get_json(silent=True)
request.files["upload"]               # uploaded file → .save(path)
request.headers.get("Authorization")
request.cookies.get("session")
request.method                        # "GET" / "POST"
request.path; request.url
```

---

## Responses

```python
from flask import jsonify, make_response, redirect, abort, render_template

return jsonify({"status": "ok"})                 # JSON + correct headers
return jsonify(data), 201                         # with status code
return "text", 200, {"X-Custom": "1"}             # (body, status, headers)
return redirect(url_for("home"))
return abort(404)                                 # raise an HTTP error
return render_template("page.html", name="Ada")   # Jinja2 template

resp = make_response("body")
resp.set_cookie("k", "v", httponly=True, max_age=3600)
resp.headers["Cache-Control"] = "no-store"
return resp
```

---

## JSON API Example

```python
from flask import Flask, request, jsonify

app = Flask(__name__)
tasks = []

@app.get("/tasks")
def list_tasks():
    return jsonify(tasks)

@app.post("/tasks")
def create_task():
    data = request.get_json()
    if not data or "title" not in data:
        return jsonify({"error": "title required"}), 400
    task = {"id": len(tasks) + 1, "title": data["title"], "done": False}
    tasks.append(task)
    return jsonify(task), 201

@app.errorhandler(404)
def not_found(e):
    return jsonify({"error": "not found"}), 404
```

---

## Templates (Jinja2)

```python
render_template("index.html", user=user, items=items)
```

```html
<!-- templates/index.html -->
<h1>Hello {{ user.name }}</h1>
{% for item in items %}
  <li>{{ item.title | upper }}</li>
{% else %}
  <li>No items</li>
{% endfor %}
{% if user.is_admin %}<a href="{{ url_for('admin') }}">Admin</a>{% endif %}
{% extends "base.html" %}{% block content %}...{% endblock %}
```

Static files live in `static/` → referenced as `url_for('static', filename='style.css')`.

---

## Blueprints (modular apps)

```python
# users.py
from flask import Blueprint
bp = Blueprint("users", __name__, url_prefix="/users")

@bp.get("/")
def index(): ...

# app.py
from users import bp
app.register_blueprint(bp)
```

---

## App Lifecycle Hooks & Config

```python
@app.before_request
def before(): ...           # runs before every request

@app.after_request
def after(resp): return resp

@app.teardown_request
def teardown(exc): ...

app.config["SECRET_KEY"] = "..."      # needed for sessions/flash
app.config.from_prefixed_env()        # load FLASK_* env vars
app.config.from_object("config.Prod")
```

### Sessions & context globals

```python
from flask import session, g, flash

session["user_id"] = 5      # signed cookie (needs SECRET_KEY)
session.get("user_id")
g.db = get_connection()     # per-request scratch storage
flash("Saved!")             # one-time message for templates
```

---

## Extensions & Production

Common extensions: **Flask-SQLAlchemy** (ORM), **Flask-Migrate** (DB migrations), **Flask-Login** (auth sessions), **Flask-WTF** (forms/CSRF), **Flask-CORS**, **Flask-RESTful/Flask-Smorest** (APIs).

```bash
# never use app.run() in production — use a WSGI server:
gunicorn -w 4 -b 0.0.0.0:8000 app:app
waitress-serve --port=8000 app:app        # Windows-friendly
```

- `debug=True` exposes an interactive debugger — **never** enable it in production.
- Set `SECRET_KEY` from the environment, not hard-coded.
- Put blocking I/O behind a proper server + workers; Flask is sync by default.

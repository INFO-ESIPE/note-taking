[[Python]]
***
**Table of Contents**
```table-of-contents
```

****
## Flask?

Lightweight WSGI web framework for Python. It’s designed for simplicity and extensibility, ideal for small to medium projects or microservices. 
Unlike Django, Flask doesn’t include built-in tools for databases or authentication—you add only what you need via extensions.

> [!info]- WSGI?
> **Web Server Gateway Interface (WSGI)** is a Python standard (PEP 3333) defining how web servers (e.g., Nginx, Apache) communicate with Python web apps (like Flask). In other words, it lets Flask talk to web servers.
> 
> 1: **Server Setup**: A WSGI server (Gunicorn, uWSGI) runs the Flask app
> 2: **Request Flow** (something like this):
> ```bash
> graph LR
> WebServer -->|HTTP Request| WSGIServer
> WSGIServer -->|WSGI Callable| FlaskApp
> FlaskApp -->|Response| WSGIServer
> WSGIServer -->|HTTP Response| WebServer
> ```
>3: **WSGI Callable**: Flask app instance (`app`) is the WSGI application
> 

Flask uses a python **templating engine** called Jinja2 to dynamically generate HTML files. It allows embedding Python-like logic directly in templates while maintaining separation of concerns.
	*It's very similar to Laravel blade files, from what I remember*
```python
# Variables
<h1>{{ title }}</h1>

# Conditionals
{% if user %}
  <p>Welcome {{ user }}!</p>
{% else %}
  <p>Login required.</p>
{% endif %}

# Loops
<ul>
{% for item in items %}
  <li>{{ loop.index }}. {{ item.name }}</li>
{% endfor %}
</ul>

# Template Inheritance
{% extends "base.html" %}
{% block content %}...{% endblock %}

# Filters
<p>{{ text|truncate(50) }}</p>  # Trim to 50 characters
```
> [!info]
> This keeps the python code apart from pure HTML stuff, and also helps to prevent basic XSS attacks thanks to auto-escaping.

Flask is, in fact, kind of like a high-level wrapper around a tool called **Werkzeug** (German for "tool"), a WSGI utility library that provides:
- Request/response object handling (`flask.Request` and `flask.Response`)
- Routing (`flask.Flask.route` decorator)
- Debugging (auto-reloader and interactive debugger)
- Cookie/session management
```python
from werkzeug.wrappers import Request, Response

def application(environ, start_response):
    request = Request(environ)
    response = Response(f"Hello {request.args.get('name', 'World')}!")
    return response(environ, start_response)

# Run with: werkzeug.serving.run_simple('localhost', 5000, application)
```
> [!info]
> Overall, Werkzeug handles low-level HTTP/WSGI details, and Flask wraps it. 
> For example:
> - `flask.Flask` inherits from `werkzeug.Application`
>- `flask.request` is a subclass of `werkzeug.Request`

Here are the main WSGI servers:

| **Server** | **Use Case**     | **Command**                           |
| ---------- | ---------------- | ------------------------------------- |
| Gunicorn   | Production       | `gunicorn app:app`                    |
| uWSGI      | High-performance | `uwsgi --http :5000 --module app:app` |
| Werkzeug   | Development only | `flask run`                           |

We can install it—preferably in our venv—with a simple pip command:
```bash
pip install flask
```

And here's what a very basic app looks like:
```python
from flask import Flask    # Imports Flask class from the module, it handles most basic features
app = Flask(__name__)      # Initialise the webapp by instanciating the `Flask` class
                           # Here, __name__ tells Flask how to name the app, and where to find resources relative to
                           # the module where `__name__` is defined

@app.route("/")
def home():
    return "Hello, World!"

if __name__ == "__main__": # Run if executed directly
    app.run(debug=True)    # Auto-reloads on code changes (debug only in dev, of course)
```
> [!tip]
>Run with `FLASK_APP=app.py flask run` for production-like settings

Flask uses thread-local objects (via Werkzeug's `Local` and `LocalProxy`) to create **request contexts**, allowing global-like access to `request`, `session`, and `g` objects without thread collisions.
Because web servers handle requests concurrently using threads. Without thread-local storage:
- `request` data from different threads could overwrite each other
- Code like `from flask import request` would be unsafe
```python
from werkzeug.local import Local

_request_ctx_stack = Local() # Each thread gets its own 'request' stack

def handle_request():        # When a request starts:
    _request_ctx_stack.push(current_request_data)
    
current_request = _request_ctx_stack.top # Access in view:
```

You mainly have:
- `app.test_request_context()`: Manually create contexts
- `flask.g`: Request-scoped global storage (e.g., database connections)

By default, a Flask app is organised like so:
```bash
project/
├── app/  
│   ├── __init__.py          # App factory (creates Flask instance)
│   ├── routes/              # Blueprints (auth, blog, etc.)
│   ├── models.py            # Database models (SQLAlchemy) 
│   ├── templates/           # Jinja2 HTML templates
│   └── static/              # CSS, JS, images
├── config.py                # Configuration (dev/prod keys, DB URLs)  
├── requirements.txt         # Dependencies
├── migrations/              # Database migration scripts (Flask-Migrate)  
├── tests/                   # Unit/integration tests
└── run.py                   # Entry point (starts the app)
```


***
## Routing

We use the `route` decorator to bind a function to an URL:
```python
@app.route('/')
def index():
    return 'Index Page'

@app.route("/user/<username>")     # Dynamic URL segments
def show_user(username):
    return f"User: {username}"

@app.route("/post/<int:post_id>")  # Type converters (int, float, path)
def show_post(post_id):
    return f"Post ID: {post_id}"
```

By default, each route above is for the `GET` method. We can specify which method(s) it accepts by filling the `methods` argument:
```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```
> [!info]
> So, the default value of `methods` is `[GET]`, we change this here to also accept POST.

There is also an **URL building** feature, which dynamically generating URLs for your routes instead of hardcoding them.
We do it via the `url_for` function, which creates URLs based on the **endpoint name** (usually the name of your view function) and any dynamic parameters in the route.
```python
from flask import url_for

# Inside a request context (e.g., in a route or template):
url = url_for("profile", username="Alice")                # Returns "/user/Alice"
url2 = url_for("profile", username="Bob", page=2)         # Returns "/user/Bob?page=2"
url_staticfile = url_for("static", filename="style.css")  # Returns "/static/style.css"
```
> [!info]
> This is useful for several reasons:
> - **Avoid hardcoding**: No need to manually fix every link in your code or templates if the route's URL changes
> - **Escape special characters**: It automatically URL-encodes dynamic values. `Éléonore` will return `%C3%89l%C3%A9onore`
> - **Convenient for complex/nested routes**
> - **Work across Blueprints** (details [[02 - Flask#Blueprints|later]])


***
## Templates
*Using Jinja2, as we mentioned in the first chapter*




***
## Blueprints
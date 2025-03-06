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

We can use the `redirect()` function to reorient the user to another route. Furthermore, the `abort()` method can be used to prematurely interrupt a request with an HTTP error code:
```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    # Any code placed below this abort() statement is never reached (like a return)
```

> [!tip]- Customise error page
> We can easily do so with the `errorhandler` decorator:
> ```python
> from flask import render_template
> 
> @app.errorhandler(404)
> def page_not_found(error):
>     return render_template('page_not_found.html'), 404 # Indicate that page should be an HTTP 404. 
> 													   # Thus far, flask was (correctly) assuming
> 													   # an HTTP 200, because its the default behaviour.
> ```


***
## Templates
*Using Jinja2, as we mentioned in the first chapter*

We use the `render_template()` method, which will automatically check in the `template` directory for the HTML file we gave:
```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')  # We can stack routes!
def hello(name=None):
    return render_template('hello.html', name=name)
```
```jinja2
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello, World!</h1>
{% endif %}
```

Similar to blade files, we can extend a **base template** with a **child template**.
Base:
```jinja2

<!doctype html>
<html>
  <head>
    {% block head %}
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    <title>{% block title %}{% endblock %} - My Webpage</title>
    {% endblock %}
  </head>
  <body>
    <div id="content">{% block content %}{% endblock %}</div>
    <div id="footer">
      {% block footer %}
      &copy; Copyright 2010 by <a href="http://domain.invalid/">you</a>.
      {% endblock %}
    </div>
  </body>
</html>
```

Child:
```jinja2
{% extends "layout.html" %}
{% block title %}Index{% endblock %}
{% block head %}
  {{ super() }}
  <style type="text/css">
    .important { color: #336699; }
  </style>
{% endblock %}
{% block content %}
  <h1>Index</h1>
  <p class="important">
    Welcome on my awesome homepage.
{% endblock %}
```
> [!info]
> The extends tag must be the first tag in the template, as the system has to locate the parent when evaluating the child template.
> `super()` is called if we want to render the contents of a block defined in the parent template.

> [!tip]
> We have access to `config`, `request`, `session` and `g` from inside templates, along with `url_for()` and `get_flashed_messages()`.

Parameters are escaped by default, which means that if we provide HTML code, it will be rendered.
If the HTML is trustworthy (comes from a reliable module, not user input for instance), you can highlight it by using the `Markup` class, or by using the `|safe` filter.
```python
from markupsafe import Markup

>>> Markup('<strong>Hello %s!</strong>') % '<blink>hacker</blink>'
# Markup('<strong>Hello &lt;blink&gt;hacker&lt;/blink&gt;!</strong>')

>>> Markup.escape('<blink>hacker</blink>')
# Markup('&lt;blink&gt;hacker&lt;/blink&gt;')

>>> Markup('<em>Marked up</em> &raquo; HTML').striptags()
# 'Marked up \xbb HTML'
```


***
## Request data

As you might expect, we will leverage the `request` global object for this task.
```python
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    return render_template('login.html', error=error)
```
> [!caution]
> Problem: what if the keys `username` and `password` aren't defined? It throws a `KeyError`.

It is important to use the `args` attribute to collect the URL parameters (`?key=value`):
```python
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        username = request.form.get('username') # Here and below
        password = request.form.get('password')

        if valid_login(username, password):
            return log_the_user_in(username)
        else:
            error = 'Invalid username or password'
    return render_template('login.html', error=error)
```

Flask also allows to easily retrieve files passed by the user:
```python
from werkzeug.utils import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        file = request.files['the_file']
        file.save(f"/var/www/uploads/{secure_filename(f.filename)}") # f.filename retrieves the original file name
															         # (on the user's machine)
															         #
															         # secure_filename converts the name to a safe
															         # ASCII format, so it can be correctly named
															         # on any File system
```

The `cookies` attribute is used to manipulate... cookies:
```python
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')  # Get cookies from request
    resp = make_response(render_template())
    resp.set_cookie('username', 'the username') # Store cookies on responses
    return resp
```
> [!info]
> Similarly to how we used `request.form.get('username')` instead of `request.form['username']` above, we use `request.cookies.get()` to correctly retrieve (or not) the cookie without raising a `KeyError` if the cookie isn't there.


***
## Response

The return value of a view function is systematically converted to a response object.
For instance, if we simply return a "Hello world" string, it will be considered as the body of a `200 OK` response of `text/html` MIME type.
If we return a dictionary, it will be converted to JSON by implicitly calling the `jsonify()` function on it.
	*Note the later function is very useful to turn anything to JSON, very convenient for APIs*

We can have a preview of the response our code will produce by using the `make_response()` function, which allows us to directly manipulate and return it (if needed):
```python
from flask import make_response

@app.errorhandler(404)
def not_found(error):
    resp = make_response(render_template('error.html'), 404)
    resp.headers['X-Something'] = 'A value'
    return resp
```


***
## Session

In addition to the `request` object, Flask provides a `session` object to store user-specific data across requests. This object is built on top of cookies and cryptographically signs them, ensuring that while users can view the cookie content, they cannot modify it without the secret key.

To use sessions, you must set a secret key. Here’s how it works (basic approach):
```python
from flask import session

# Set the secret key to some random bytes. Keep this really secret!
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    if 'username' in session:
        return f'Logged in as {session["username"]}'
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
```

As always, we are consumers of cryptography. So, we will generate our key with reliable tools, not by ourselves:
```bash
python -c 'import os; print(os.urandom(128))'
b"\r)\x93\xf7\xf8\xa6\xae\xca\x8e+\xe1Tx\x96F\xa6\x0b7\xf9\xfc\xf6\xb0\x84\xfdN\xb3\x88\xc8\x7f5\xe4\xb8\xdc\x92,\xff\xcd\\_P\xb5\x1c\x0cSn\xdb\xea\x1b\xc3\x8f\xe3\x0b\x1f\xfb;I\xad\x0c\x84%L+\x0f\xe2\x07\xd0\x11\xb3JV\xa5\x7f\xa7'c\xfeZ\xb9\x1fO\x03\xaa@}\x19O|\xda\x12j\xcf}\x88\xa8\xcc\xd8}%\xc9\t\x85\xffNZY\x8c\x81\xa5#):LBz\x880\x11I\x11$\xdf\x86}\x87\x10\x11\xd38"
```


***
## (Flask-)SQLAlchemy

Widely-used **Object-Relational Mapping (ORM)** tool for **interacting with databases using Python objects**. 
	*Flask-SQLAlchemy is an extension that simplifies using SQLAlchemy with Flask.*
```bash
pip install flask-sqlalchemy
```

We usually configure it in a simple way like that:
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Configure database URI
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Initialize SQLAlchemy instance
db = SQLAlchemy(app)
```

We can now create models through classes inheriting from `db.Model`:
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

    def __repr__(self):
        return f'<User {self.username}>'
```
> [!info]
> Very similar to Laravel migrations

Basic queries:
```python
# Add new user
new_user = User(username='john_doe', email='john@example.com')
db.session.add(new_user)
db.session.commit()

# Query users
users = User.query.all()
```


***
## Blueprints

Blueprints allow to organise the application (views and related code) into smaller, reusable modules. This is particularly useful for larger applications where you want to separate functionality into distinct components.
- **Modularity**: Organise related views, templates, and static files into separate modules
- **Reusability**: Blueprints can be registered on multiple applications
- **URL Prefixes and namespaces**: Easily add a URL prefix to all routes in a Blueprint, allowing to separate parts of the application in dedicated namespaces and avoid conflicts

```python
from flask import (
    Blueprint, flash, g, redirect, render_template, request, session, url_for
)
from werkzeug.security import check_password_hash, generate_password_hash

from flaskr.db import get_db

bp = Blueprint('auth', __name__, template_folder='templates') # Creates an "auth" blueprint. 
															  # Object must know where it was defined, so 
															  # we pass __name__.
```

If we want to use this blueprint, we have to **register it with the app instance**:
```python
from flask import Flask

app = Flask(__name__)

# Register the Blueprint with a URL prefix (which will be added to all URLs associated to the bp)
app.register_blueprint(bp, url_prefix='/pages')
```

> [!example]-
> Imagine building a simple webapp with multiple features: user authentication, blog posts, and an admin dashboard. 
> Using Blueprints, you can organise these features into separate modules:
> - **Auth Blueprint**: Handles user registration, login, and logout
> - **Blog Blueprint**: Manages blog posts, including creating, reading, updating, and deleting posts
> - **Admin Blueprint**: Provides administrative functionality, such as user management and site configuration
> 
> **Without blueprints**, all your routes and view functions would be defined in a single file or a few large files. This can quickly become unwieldy as your application grows:
> ```python
> @app.route('/register', methods=('GET', 'POST'))
> def register():
>     # Registration logic
>     pass
> 
> @app.route('/login', methods=('GET', 'POST'))
> def login():
>     # Login logic
>     pass
> 
> @app.route('/posts', methods=('GET', 'POST'))
> def posts():
>     # Blog post logic
>     pass
> 
> @app.route('/admin/users', methods=('GET', 'POST'))
> def admin_users():
>     # Admin user management logic
>     pass
> ```
> 
> **With blueprints** on the other hand, you can separate these concerns into different modules, each with its own Blueprint:
> ```python
> # auth.py
> from flask import Blueprint
> 
> bp = Blueprint('auth', __name__, url_prefix='/auth')
> 
> @bp.route('/register', methods=('GET', 'POST'))
> def register():
>     # Registration logic
>     pass
> 
> @bp.route('/login', methods=('GET', 'POST'))
> def login():
>     # Login logic
>     pass
> 
> # blog.py
> from flask import Blueprint
> 
> bp = Blueprint('blog', __name__, url_prefix='/blog')
> 
> @bp.route('/posts', methods=('GET', 'POST'))
> def posts():
>     # Blog post logic
>     pass
> 
> # admin.py
> from flask import Blueprint
> 
> bp = Blueprint('admin', __name__, url_prefix='/admin')
> 
> @bp.route('/users', methods=('GET', 'POST'))
> def admin_users():
>     # Admin user management logic
>     pass
> ```
> Let's take the first `@bp.route`: it associates the `/register` URL to the `register` view function. 


***
## Signals

Relies on the **blinker** library, provides a way to decouple components in the application by allowing them to communicate with each other through event-like mechanisms.
	*Senders notify subscribers about an event which happened, similarly to an Observer design pattern*
```bash
pip install blinker
```

We begin by **defining the signal** (that can be emitted when a specific event occurs):
```python
from blinker import Namespace

my_signals = Namespace()           # Create a Namespace for your signals
user_registered = my_signals.signal('user-registered') # Define a signal
```
> [!info]
> A namespace is a container for related signals. It helps organise signals logically, especially when you have multiple signals that belong to the same module or feature (avoids naming conflicts). 

We can now **register a listener function** (with the `connect()` function) that will be **called when the signal is emitted**:
```python
def notify_admin(sender, **extra):
    print(f"Admin notified: A new user has registered with email {extra.get('email')}")

user_registered.connect(notify_admin) # Connect the signal to the listener
```
> [!info]
> `**extra` is a way to pass additional keyword arguments to the signal listeners. When you emit a signal, you can include extra data that the listeners might need to perform their actions. This data is passed as keyword arguments to the listener functions, as they can have any name you want.
> `sender` represents the object that sent the signal. It's useful if you have multiple objects that can emit the same signal, and you want to differentiate between them in your listener.

Finally, we **trigger the signal when the event occurs**, passing any relevant data.
```python
@app.route('/register', methods=['POST'])
def register():
    # Registration logic...
    email = request.form['email']

    # Emit the signal
    user_registered.send(current_app._get_current_object(), email=email)

    return "Registered!"
```
> [!info]
> `current_app` is a proxy for the current Flask application instance. `_get_current_object()` is a method that retrieves the actual application instance that is handling the current request.
> You often pass `current_app._get_current_object()` as the `sender` to indicate that the signal is being sent by the current application instance. This is useful for listeners that need to access the application context.

Note that signals are made for observation, not for direct modification of the data...


***
## Security



***
## Logging and error handling



***
## Asynchronous Tasks
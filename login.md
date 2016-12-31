
Simple Example
---------------
source: https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-v-user-logins

#### installation
```
pip install flask flask-sqlalchemy, flask-login, flask-wtf
```

#### app.py
```python
import os
from urllib.parse import urlparse, urljoin

import flask
from flask import request, url_for
from flask import Flask, g, redirect, url_for, request

from flask_sqlalchemy import SQLAlchemy
from flask_login import login_user, logout_user, current_user, login_required, LoginManager, UserMixin
from wtforms import  TextField, PasswordField, validators
from flask_wtf import FlaskForm


basedir = os.path.abspath(os.path.dirname(__file__))
print(basedir)

app = Flask(__name__)

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir, 'test.db')
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True
app.config['SECRET_KEY'] = 'SAOE343425TDISDA434THS422eiiiiOthethastcaoegysjJE3e836h'

db = SQLAlchemy(app)

login_manager = LoginManager()
login_manager.init_app(app)


class User(db.Model, UserMixin):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)
    password = db.Column(db.String(50), unique=True)
    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password


class LoginForm(FlaskForm):
    username = TextField('Username', [validators.Required()])
    password = PasswordField('Password', [validators.Required()])

    def __init__(self, *args, **kwargs):
        FlaskForm.__init__(self, *args, **kwargs)
        self.user = None


def is_safe_url(target):
    ref_url = urlparse(request.host_url)
    test_url = urlparse(urljoin(request.host_url, target))
    return test_url.scheme in ('http', 'https') and \
           ref_url.netloc == test_url.netloc

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))


@app.before_request
def before_request():
    g.user = current_user


@app.route('/login', methods=['GET', 'POST'])
def login():
    if g.user is not None and g.user.is_authenticated:
        return redirect(url_for('index'))
    # Here we use a class of some kind to represent and validate our
    # client-side form data. For example, WTForms is a library that will
    # handle this for us, and we use a custom LoginForm to validate.
    form = LoginForm()
    if request.method == 'POST':
        if form.validate_on_submit():
            # Login and validate the user.
            # user should be an instance of your `User` class
            username = request.form['username']
            user = User.query.filter_by(username=username).first()
            
            login_user(user)

            flask.flash('Logged in successfully.')

            next = flask.request.args.get('next')
            # is_safe_url should check if the url is safe for redirects.
            # See http://flask.pocoo.org/snippets/62/ for an example.
            if not is_safe_url(next):
                return flask.abort(400)

            return flask.redirect(next or flask.url_for('index'))
    return flask.render_template('login.html', form=form)

@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('index'))

@app.route('/')
def index():
    return flask.render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```

#### `/templates/login.html`
```html
<html>
<form method="POST" action="/login">
    {{ form.csrf_token }}
    {{ form.username.label }} {{ form.username(size=20) }}
    {{ form.password.label }} {{ form.password(size=20) }}
    <input type="submit" value="Go">
</form>
</html>
```

#### `/templates/index.html`
```html
<html>
  <head>
  </head>
  <body>
    
    <hr>
    {% with messages = get_flashed_messages() %}
    {% if messages %}
    <ul>
    {% for message in messages %}
        <li>{{ message }} </li>
    {% endfor %}
    </ul>
    {% endif %}
    {% endwith %}
    {% block content %}


    {% endblock %}
  </body>
</html>
```
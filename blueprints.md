Overview
---------
Flask blueprints allow you to modularize & connect your views
so it makes it easier to organize your project
structure.

Basic examples
-------------

```python
# apps/myapp/simple_page.py
from flask import Blueprint, render_template, abort
from jinja2 import TemplateNotFound

simple_page = Blueprint('simple_page', __name__,
                        template_folder='templates')

@simple_page.route('/', defaults={'page': 'index'})
@simple_page.route('/<page>')
def show(page):
    '''
    Based on url_prefix, the route for this view
    is : /pages/mypage
    '''
    return 'simple page: {}'.format(page)
```


```python
# app.py
from flask import Flask
from apps.myapp.simple_page import simple_page

from flask import Flask
app = Flask(__name__)
app.register_blueprint(simple_page, url_prefix='/pages')

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run()
```

More info:
http://flask.pocoo.org/docs/0.12/blueprints/
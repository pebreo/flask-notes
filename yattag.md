
Overview
-------
yattag is a Python library that makes it easier to write html
using Python.

Examples
--------
### A simple example using `for` loop
```python
from yattag import Doc
# get the methods
doc, tag, text = Doc().tagtext()

for i in range(5):
    with tag('div', id='{}'.format(i)):
        with tag('h1'):
            text('hello {}'.format(i))

print(doc.getvalue())

```

### A form example
```python
from yattag import Doc
doc, tag, text, line = Doc(
    defaults = {
        'title': 'Untitled',
        'contact_message': 'you won the lottery!'
    },
    errors = {
        'contact_message': 'your message looks like spam'
    }
).ttl()

line('h1', 'Contact form')
with tag('form', action=""):
    doc.input(name='title', type='text')
    with doc.textarea(name='contact_mesage'):
        pass
    doc.stag('input', type='submit', value='send my message')

print(doc.getvalue())

```
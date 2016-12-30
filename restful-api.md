
Simple API
----------
```python
'''
Troubleshooting:
Make sure to refresh your webpage which refreshes the code too

curl -i -H "Content-Type: application/json" \
-X POST \
-d '{"title":"Read a book"}' \
http://127.0.0.1:5000/todo/tasks


or using httpie: 
pip install httpie

http POST localhost:5000/todo/tasks title='mytitle'

with password:
http POST -a username:pw localhost:5000/todo/tasks title='mytitle'

deleting:
http DELETE  localhost:5000/todo/tasks/4  

updating:
http PUT localhost:5000/todo/tasks/2 title='be our guest'
'''
from flask import Flask, jsonify, abort, make_response, request


from flask import Flask
app = Flask(__name__)


tasks = [
    {
        'id': 1,
        'title': u'Buy groceries',
        'description': u'Milk, Cheese, Pizza, Fruit, Tylenol', 
        'done': False
    },
    {
        'id': 2,
        'title': u'Learn Python',
        'description': u'Need to find a good Python tutorial on the web', 
        'done': False
    }
]

@app.errorhandler(404)
def not_found(error):
    return make_response(jsonify({'error': 'Not found'}), 404)

@app.route('/todo/tasks', methods=['GET'])
def get_tasks():
    return jsonify({'tasks':tasks})

@app.route('/todo/tasks/<int:task_id>')
def get_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]
    if len(task) == 0:
        abort(404) # calls errorhandler-wrapped function
    return jsonify({'tasks': task[0]})

@app.route('/todo/tasks', methods=['POST'])
def create_task():
    if not request.json or not 'title' in request.json:
        abort(400)
    task = {
        'id': tasks[-1]['id'] + 1,
        'title': request.json['title'],
        'description': request.json.get('description', ""),
        'done': False
    }
    tasks.append(task)
    return jsonify({'task': task}), 201

@app.route('/todo/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    task = [task for task in tasks if task['id'] == task_id]
    if len(task) == 0:
        abort(404)
    tasks.remove(task[0])
    return jsonify({'result': True})

@app.route('/todo/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    '''
    for demonstration purposes only,
    this function doesn't actually update the global tasks
    '''
    task = [task for task in tasks if task['id'] == task_id]
    if len(task) == 0:
        abort(404)
    if not request.json:
        abort(400)
    if 'title' in request.json and type(request.json['title']) != str:
        abort(400)
    if 'description' in request.json and type(request.json['description']) is not str:
        abort(400)
    if 'done' in request.json and type(request.json['done']) is not bool:
        abort(400)
    task[0]['title'] = request.json.get('title', task[0]['title'])
    task[0]['description'] = request.json.get('description', task[0]['description'])
    task[0]['done'] = request.json.get('done', task[0]['done'])
    return jsonify({'task': task[0]})

if __name__ == '__main__':
    app.run()
```
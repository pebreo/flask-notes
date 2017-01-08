
Overview
--------
JWT authenticiation is a type of authentication used for frontends 
that make REST calls to a backend.

The main use case for using JWT is when you have a single page javascript
application that will be controlling your whole front end and 
you want to do CRUD operations with the backend with authentication.

We first send the JSON to `/auth` 
```
POST /auth HTTP/1.1
Host: localhost:5000
Content-Type: application/json

{
    "username": "joe",
    "password": "mypassword"
}
```
and then it will respond with
```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "access_token": "thetokenstring"
}
```
You should save the access token above and use it for your subsequent requests to the backend.
So when we send a request to the backend (GET, POST, PATCH, DELETE, etc),
we have to send a request with a header `Auth: JWT thetokenstring`
```
GET /myprotectedurl HTTP/1.1
Authorization: JWT thetokenstring
```

source: https://pythonhosted.org/Flask-JWT/

EXAMPLE USING JQUERY
--------------------
In this example we use jQuery to get a token from the server
and it shows you how to get data from the backend using
jQuery ajax.

Keep in mind that JWT is only good for REST-type communication.
If you need to do logging in then you should use flask-login (see the login notes in this repo).

### Installation
```
pip install flask-jwt
```

### Flask code
```python
from flask import Flask
from flask_jwt import JWT, jwt_required, current_identity
from werkzeug.security import safe_str_cmp

class User(object):
    def __init__(self, id, username, password):
        self.id = id
        self.username = username
        self.password = password

    def __str__(self):
        return "User(id='%s')" % self.id

users = [
    User(1, 'user1', 'abcxyz'),
    User(2, 'user2', 'abcxyz'),
]

username_table = {u.username: u for u in users}
userid_table = {u.id: u for u in users}

def authenticate(username, password):
    user = username_table.get(username, None)
    if user and safe_str_cmp(user.password.encode('utf-8'), password.encode('utf-8')):
        return user

def identity(payload):
    user_id = payload['identity']
    return userid_table.get(user_id, None)

app = Flask(__name__)
app.debug = True
app.config['SECRET_KEY'] = 'super-secret'

jwt = JWT(app, authenticate, identity)

@app.route('/protected')
@jwt_required()
def protected():
    return jsonify(dict(data=[1,2,3]))

if __name__ == '__main__':
    app.run()

```

### login.html
```javascript

<form id="form" action="" method="post">
  

  <div class="container">
    <label><b>Username</b></label>
    <input type="text" placeholder="Enter Username" name="username" required>

    <label><b>Password</b></label>
    <input type="password" placeholder="Enter Password" name="password" required>

    <input id="submit" type="button" name="submit" value="submit">
    
  </div>

</form>

<div id="message"></div>

<div><input id="get_protected_data" type="button" name="Get protected data"></input></div>


<script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js" type="text/javascript"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/js-cookie/2.1.3/js.cookie.min.js"></script>
<script>

    function serialToJson(serializer){
        var _string = '{';
        for(var ix in serializer)
        {
            var row = serializer[ix];
            _string += '"' + row.name + '":"' + row.value + '",';
        }
        var end =_string.length - 1;
        _string = _string.substr(0, end);
        _string += '}';
        //console.log('_string: ', _string);
        //return JSON.parse(_string);
        return _string;
    };
 
    $(document).ready(function () {
        $("#message").html('');
        localStorage.setItem('token', '');
        $("#submit").on('click',function(){

          var vals = $('#form').serializeArray();
          json_data = serialToJson(vals);

          $.ajax({
            url: "/auth", // the default route
            type: 'POST',
            data: json_data,
            contentType: "application/json; charset=utf-8",
            error : function(err) {
              console.log('Error!', err);
              $("#message").html('Invalid login!');
            },
            success: function(data) {
              $("#message").html('success!');
              console.log('Success!');
              console.log('the token');
              console.log(data.access_token);
              localStorage.setItem('token', data.access_token);
              
            }
          });

        });
        $("#get_protected_data").on('click', function(){
            $.ajax({
                url: "/protected",
                type: 'GET',
                // Fetch the stored token from localStorage and set in the header
                headers: {"Authorization": "JWT " + localStorage.getItem('token')},
                error : function(err) {
                    console.log('Error!', err);
                    $("#message").html('Getting proctected data failed!');
                },
                success: function(data) {
                    $("#message").html('successfully get protected data');
                    console.table(data);
                }
            });
        });
        
    });
 
</script>

```
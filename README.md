# Setting Up a Flask App

When setting up a Flask app, it's important to ensure that things are properly named or else you'll create problems for yourself in the future, so begin with this file structure:

- __flask_app__
    - __config__
        - mysqlconnection.py
    - __controllers__
        - route_controller.py
        - x_controller.py
    - __models__
        - x_model.py
    - __static__
        - css
            - style.css
        - js
            - script.js
    - __templates__
        - index.html
    - \_\_init__.py
- Pipfile
- Pipfile.lock
- server.py

## Config

Now, if your app requires a database to talk to, it won't be able to do so without the proper config files, so here is basic template to use:

#### mysqlconnection.py

```py
import pymysql.cursors

class MySQLConnection:
    def __init__(self, db):

        connection = pymysql.connect(host = 'localhost',
                                    user = 'root', 
                                    password = 'root', 
                                    db = db,
                                    charset = 'utf8mb4',
                                    cursorclass = pymysql.cursors.DictCursor,
                                    autocommit = True)

        self.connection = connection

    def query_db(self, query, data=None):
        with self.connection.cursor() as cursor:
            try:
                query = cursor.mogrify(query, data)
                print("Running Query:", query)
    
                cursor.execute(query)
                if query.lower().find("insert") >= 0:
                    self.connection.commit()
                    return cursor.lastrowid
                elif query.lower().find("select") >= 0:
                    result = cursor.fetchall()
                    return result
                else:
                    self.connection.commit()
            except Exception as e:
                print("Something went wrong", e)
                return False
            finally:
                self.connection.close() 

def connectToMySQL(db):
    return MySQLConnection(db)
```

## MVC

We want to ensure that we are following MVC (Model, View, Controller) principles when creating our app, so we must ensure that the files found within the file structure are doing their jobs and nothing else.

### Models

Within flask_app, you'll find the models folder. Here, you'll find the files (think of these as blueprints) for those models.

#### user_model.py

Here is a basic model template for pulling data from a database:

```py
from flask_app.config.mysqlconnection import connectToMySQL
# model the class after the name table from our database
class User:
    def __init__( self , data ):
        self.id = data['id']
        self.first_name = data['first_name']
        self.last_name = data['last_name']
        self.email = data['email']
        self.created_at = data['created_at']
        self.updated_at = data['updated_at']
    # Now we use class methods to query our database
    @classmethod
    def get_all(cls):
        query = "SELECT * FROM users;"
        results = connectToMySQL('users_schema').query_db(query)
        all_users = []

        for dict in results:
            all_users.append( cls(dict) )
        return all_users

    def create(cls, data):
        query = "INSERT INTO users (first_name, last_name, email) VALUE (%(first_name)s, %(last_name)s, %(email)s)"
        user_id = connectToMySQL('users_schema').query_db(query, data)
        return user_id
    
    def get_one(cls):
        return

    def update_one(cls):
        return

    def delete_one(cls):
        return
```

### Views

Our views will come from our templates found in the templates folder. Here is a basic template to get you set up using the above db connection and model:

#### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Practice</title>
</head>
<body>
    <h1>YO YOUR APP RUNS!</h1>
    <h2>Hello all users!</h2>
    <ul>
        {% for user in all_users %}
        <li>
            {{ user.first_name }}
        </li>
        {% endfor %}
    </ul>
    
</body>
</html>
```

### Controllers

Now we'll need a way to get our models to interact with our views.

#### \_\_init__.py

First, you'll need to instantiate your Flask app:

```py
from flask import Flask
app = Flask (__name__)
app.secret_key = 'keep it secret, keep it safe'
```

#### server.py

Then you'll need to create the server:

```py
from flask_app import app
# Import ALL controller files
from flask_app.controllers import user_controller, route_controller

if __name__ == "__main__":
    app.run(debug=True)
```

#### route_controller.py

Now, you'll need some routes:

```py
from flask_app import app
from flask import render_template, session, request, redirect
from flask_app.models.user_model import User


@app.route("/")
def index():
    all_users = User.get_all()
    print(all_users)
    return render_template("index.html", all_users=all_users)

@app.route("/dashboard")
def dashboard():
    return render_template("dashboard.html")
```

#### user_controller.py

Your route controller shouldn't do all the routing, so here's a template for some routing based on the user_model.py file:

```py
from flask_app import app
from flask import render_template, session, request, redirect
from flask_app.models.user_model import User


@app.route("/user/new")
def user_new():
    return render_template("user_new.html")



@app.route("/user/create", methods=['POST'])
def user_create():
    return redirect("/")



@app.route("/user/<int:id>")
def user_show():
    return render_template("user_show.html")



@app.route("/user/<int:id>/edit")
def user_edit():
    return render_template("user_edit.html")



@app.route("/user/<int:id>/update")
def user_update():
    return redirect("/")



@app.route("/user/<int:id>/delete")
def user_delete():
    return redirect("/")
```

## Further Notes

This is just a template that uses a MySql database to display some made up usernames. If you want it to work, you'll need to create a database schema called __users_schema__  with a table called __users__. That table will require the following rows: 

- id 
- first_name
- last_name
- email
- created_at
- updated_at

You'll also want to make sure that your MySql settings match those on the config file or else this thing just won't work. Now you've got a basic skeleton for getting a Flask app going using a database. Set it up however you want, and you'll be good to go!

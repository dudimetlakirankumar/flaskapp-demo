Clone this project

pip3 install -r requirements.txt

Change the password in db.yaml to that of your MySQL's password

Run the application by executing the command python3 app.py

The application runs on localhost:5000

======================

sudo apt update
sudo apt install libmysqlclient-dev

export MYSQLCLIENT_CFLAGS=`mysql_config --cflags`
export MYSQLCLIENT_LDFLAGS=`mysql_config --libs`
--------------------

from flask import Flask, render_template, request, redirect
from flask_mysqldb import MySQL
from yaml import safe_load

app = Flask(__name__)

# Configure db
db = safe_load(open('db.yaml'))
app.config['MYSQL_HOST'] = db['mysql_host']
app.config['MYSQL_USER'] = db['mysql_user']
app.config['MYSQL_PASSWORD'] = db['mysql_password']
app.config['MYSQL_DB'] = db['mysql_db']

mysql = MySQL(app)

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        # Fetch form data
        userDetails = request.form
        name = userDetails['name']
        email = userDetails['email']
        cur = mysql.connection.cursor()
        cur.execute("INSERT INTO users(name, email) VALUES(%s, %s)",(name, email))
        mysql.connection.commit()
        cur.close()
        return redirect('/users')
    return render_template('index.html')

@app.route('/users')
def users():
    cur = mysql.connection.cursor()
    resultValue = cur.execute("SELECT * FROM users")
    if resultValue > 0:
        userDetails = cur.fetchall()
        return render_template('users.html',userDetails=userDetails)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80, debug=True)


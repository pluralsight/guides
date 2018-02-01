[Flask](http://flask.pocoo.org/) is an amazingly lightweight framework, and in my opinion it's a great option for writing simple applications in Python.

## Background

An application in my company performs some tasks and then updates a MySql database accordingly. So to know how the application is progressing with the day's tasks, one has to use SSH (Secure Shell protocol) to get into the application server and run the appropriate queries using MySQL client. This is because the application only starts creating reports once all tasks are finished. 

My objective was to figure out a way for someone without system administration knowledge to access this data securely without additional assistance. 

## Here we go...

My plan was to create a flask application that would be accessible from any web browser. In this application, when the correct path is accessed, the software will run some predefined shell command and return the results to the browser. Now this output might be a sensitive information, so we just can't allow anyone to be able to access this. To implement a basic layer of security 
I have used **IP whitelisting**, or only allowing certain IPs to acccess the application.

> *I ran the below commands on Ubuntu 14.04 Trusty, 
if you are using a different OS then your commands may vary accordingly.*

### Prerequisites

_To go along with this tutorial you must have the following installed in your system._ 

* Python3 **(usually installed by default on Ubuntu 14.04)**
* virtualenv **(sudo apt-get install python-virtualenv)**
* MySql Server 5.5+ & Client **(sudo apt-get install mysql-server-5.5)**
* Supervisor **(sudo apt-get install supervisor)**

### Prepare your virtual environment

Use the following commands to prepare your virtual environment (virtualenv) - 

~~~bash
# go to your workspace directory
cd ~/workspace/
# create a virtualenv using python3
virtualenv -p /usr/bin/python3 flaskshell
# enter the virtualenv directory and perform the basic package installations and tasks
cd flaskshell
# activate virtualenv
source bin/activate
# install flask
pip install flask
# create src and logs directory
mkdir src logs

~~~

### Prepare a sample database

Since we will soon run a MySQL query, we need to prepare a sample database for our task.

~~~bash
# create the database 
mysql -u root -p -e "CREATE DATABASE flasktest; GRANT ALL ON flasktest.* TO flaskuser@localhost IDENTIFIED BY 'flask123'; FLUSH PRIVILEGES"
# create a sample table
mysql -uflaskuser -pflask123 -e "CREATE TABLE flasktest.tasks (task_id INT NOT NULL AUTO_INCREMENT, task_title VARCHAR(50), task_status VARCHAR(50), PRIMARY KEY (task_id));"
# insert some sample data
mysql -uflaskuser -pflask123 -e "INSERT INTO flasktest.tasks (task_title, task_status) VALUES ('Task 1', 'Success');"
mysql -uflaskuser -pflask123 -e "INSERT INTO flasktest.tasks (task_title, task_status) VALUES ('Task 2', 'Pending');"
mysql -uflaskuser -pflask123 -e "INSERT INTO flasktest.tasks (task_title, task_status) VALUES ('Task 3', 'Failed');"

~~~

Let's check whether the database and table creations went according to plan.

~~~bash
mysql -uflaskuser -pflask123 -e "USE flasktest; SELECT COUNT(*) FROM tasks WHERE task_status='Success';"

# the above command should print this,

+----------+
| COUNT(*) |
+----------+
| 1 |
+----------+

~~~

You may run the commands with statements for 'Pending' and 'Failed' as well just to double-check. All the queries should return the same result.

### The code

Create a file called **app.py** inside the src directory,

~~~bash
touch ~/workspace/flaskshell/src/app.py
~~~

Inside the file put in the code listed below.

~~~python
from flask import Flask
from flask import request
import subprocess


app = Flask('flaskshell')
ip_whitelist = ['192.168.1.2', '192.168.1.3']
query_success = "SELECT COUNT(*) FROM flasktest.tasks WHERE task_status='Success'"
query_pending = "SELECT COUNT(*) FROM flasktest.tasks WHERE task_status='Pending'"
query_failed = "SELECT COUNT(*) FROM flasktest.tasks WHERE task_status='Failed'"


def valid_ip():
    client = request.remote_addr
    if client in ip_whitelist:
        return True
    else:
        return False


@app.route('/status/')
def get_status():
    if valid_ip():
        command_success = "mysql -uflaskuser -pflask123 -e '{0}'".format(
            query_success)
        command_pending = "mysql -uflaskuser -pflask123 -e '{0}'".format(
            query_pending)
        command_failed = "mysql -uflaskuser -pflask123 -e '{0}'".format(
            query_failed)

        try:
            result_success = subprocess.check_output(
                [command_success], shell=True)
            result_pending = subprocess.check_output(
                [command_pending], shell=True)
            result_failed = subprocess.check_output(
                [command_failed], shell=True)
        except subprocess.CalledProcessError as e:
            return "An error occurred while trying to fetch task status updates."

        return 'Success %s, Pending %s, Failed %s' % (result_success, result_pending, result_failed)
    else:
        return """<title>404 Not Found</title>
               <h1>Not Found</h1>
               <p>The requested URL was not found on the server.
               If you entered the URL manually please check your
               spelling and try again.</p>""", 404


if __name__ == '__main__':
    app.run()
~~~

[Code Gist](https://gist.github.com/redmoses/347a2ad006a518f09fbc)

> **Line 7** >> This is the array for the whitelisted IPs. 
> You should replace the IPs as needed. You may put in virtually as many ips as you want in this array

> **Lines 8-10** >> I'm defining the queries here. You may change the query to suit your needs. 

> **Lines 13-18** >> The valid_ip() method returns true if the client's IP belongs in the white list, otherwise it returns false. It gets the client's IP using the request package from Flask. This request package is defined on **line 2** 

> **Line 21** >> Defines the route for accessing the application

> **Line 23** >> Before processing the request check if the client's IP belongs to the white list. If it does not, show Flask's default 404 page **(lines 43-47)**

> **Lines 24-29** >> Compose the shell commands using the queries defined earlier. 

> **Lines 32-37** >> Try running the shell commands. The application will either throw an error or, if execution is successful, it will return the results **(line 41)**


### Run the application as a service

To run the application as a service I used [Supervisor](http://supervisord.org/). This is a matter of personal preference; feel free to use any other process control system.

Define a program on Supervisor.

Create a new Supervisor config file.

~~~bash
sudo vim /etc/supervisor/conf.d/flaskshell.conf
~~~

Copy and paste the following code into the file. At this point, you must put the app in - **/home/user/workspace/flaskshell**.

~~~bash
[program:stats]
directory = /home/user/workspace/flaskshell/src
command = /home/user/workspace/flaskshell/bin/python app.py
redirect_stderr = true
stdout_logfile = /home/user/workspace/flaskshell/logs/out.log
stderr_logfile = /home/user/workspace/flaskshell/logs/error.log
~~~

Now you should update the Supervisor config and start the application.

~~~bash
sudo supervisorctl update stats
sudo supervisorctl start stats
~~~

## Accessing the application

Since we have not defined any port for the application, it will default to port 5000. To change this, follow the instructions I found on [this Stack Overflow page](http://stackoverflow.com/a/29079598/2894655).

You can find the application at [http://SERVER_IP:5000](#). And the status updates should be available at [http://SERVER_IP:5000/status/](#).


I hope my post was enjoyable and of help to you. Please feel free to leave any comments below with thoughts and feedback.

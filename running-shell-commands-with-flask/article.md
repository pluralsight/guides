[Flask](http://flask.pocoo.org/) is an amazingly lightweight framework, and to my opinion its a great option 
for writing small simple applications in Python.

## Background

An application in my company performs some tasks and then updates a MySql database 
accordingly. So to know how the application is progressing with the day's tasks one 
has to ssh into the application server and then run the appropriate queries using 
mysql client. This is because the application only starts creating the reports once 
all the tasks are finished. 

My task was to figure out a way so that someone without any system administration 
knowledge might also access this data securely without anyone's help. 

## So here it goes...

My plan was to create a flask application that will be accessible from any web browser. When 
the correct path will be accessed the application will run some predefined shell command and 
return the results to the browser. Now this output might be a sensitive information, so we 
just can't allow anyone to be able to access this. To implement a basic layer of security 
I have used IP white listing i.e, only allowing certain IPs to acccess the application.

> *I ran the below commands on Ubuntu 14.04 Trusty, 
if you are using a different OS then your commands may vary accordingly*

### Prerequisites

To go along with this tutorial you must have the following installed in your system - 

* Python3 **(usually installed by default on Ubuntu 14.04)**
* virtualenv **(sudo apt-get install python-virtualenv)**
* MySql Server 5.5+ & Client **(sudo apt-get install mysql-server-5.5)**
* Supervisor **(sudo apt-get install supervisor)**

### Prepare your virtual environment

To prepare your virtual environment use the following commands - 

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

As we would be running a mysql query we first need to prepare a sample database for our task. To create a database do 
the following,

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

Now to check whether the database and table creations all went okay do the following,

~~~bash
mysql -uflaskuser -pflask123 -e "USE flasktest; SELECT COUNT(*) FROM tasks WHERE task_status='Success';"

# the above command should print this,

+----------+
| COUNT(*) |
+----------+
| 1 |
+----------+

~~~

You may run the commands with where statements for 'Pending' and 'Failed' as well just to make sure. All the 
queries should give the same result.

### The code

Now create a file called **app.py** inside the src directory,

~~~bash
touch ~/workspace/flaskshell/src/app.py
~~~

Inside the file put in the below code -

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

> **Line 7** >> This is the array for the white listed ips. 
> You should replace the ips according to your needs. 
> You may put in virtually as many ips as you want in this array

> **Lines 8-10** >> I'm defining the queries here. You may change 
> the query according to your needs here. 

> **Lines 13-18** >> The valid_ip() method. It returns true if the 
> client's ip belongs in the white list, otherwise it returns false.
> It gets the client ip using the request package from flask that is 
> defined on **line 2** 

> **Line 21** >> Define the route for accessing the application

> **Line 23** >> Before processing the request check if the client's
> ip belong to the white list. If not then show Flask's default 404 
> page **(lines 43-47)**

> **Lines 24-29** >> Compose the shell commands using the queries 
> defined earlier

> **Lines 32-37** >> Try running the shell commands if it faces any error 
> trying then the method will return the error message **(line 39)**. If the 
> execution is successful then it will return the results **(line 41)**


### Run the application as a service

To run the application as a service I used supervisor. I'm not using it because 
its the best process manager, rather I'm just used to it and it seems to do the 
job.

Define a program on supervisor.

Create a new supervisor config file,

~~~bash
sudo vim /etc/supervisor/conf.d/flaskshell.conf
~~~

Copy and paste the following code in the file. I'm assuming you have put the 
app in the location - **/home/user/workspace/flaskshell** . You must change 
this value accordingly.

~~~bash
[program:stats]
directory = /home/user/workspace/flaskshell/src
command = /home/user/workspace/flaskshell/bin/python app.py
redirect_stderr = true
stdout_logfile = /home/user/workspace/flaskshell/logs/out.log
stderr_logfile = /home/user/workspace/flaskshell/logs/error.log
~~~

Now you should update the supervisor config and start the application

~~~bash
sudo supervisorctl update stats
sudo supervisorctl start stats
~~~

## Accessing the application

As we have not defined any port for the application it will run on the default 
port 5000. To change this you may have a look at [here](http://stackoverflow.com/a/29079598/2894655).

So you can find the application at [http://SERVER_IP:5000](#). And the 
the status updates should be available at [http://SERVER_IP:5000/status/](#).


I hope you've enjoyed the post and it was of help to you. Please feel free 
to leave any comments below.
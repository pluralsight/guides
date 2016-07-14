# Introduction

Hello everyone!, this tutorial aims to show you how in simple steps to create your first application in real time using Node.JS and RethinkDB.

RethinkDB, is an open source database-oriented documents (using JSON format) which can be used in building applications as it integrates easily from drivers languages like Javascript, Java, Ruby and Python. This database reverses the architecture of traditional database to support the developing non-invasive, simple and efficient of real-time applications. Thus instead of performing constant polling for data changes, you can delegate this task to RethinkDB and get updates whenever a change occurs. As a result, we can develop applications less complex and easier to maintain and scale.


For the development of the application we will use the following technologies:
- Node.JS [4.2.4]
- Express [4.13.1]
- Jade [1.11.0]
- Socket.IO [1.4.6]
- RethinkDB [2.3.2]

## Requirements

### Install Node.js
To learn how to install Node.JS, follow the link below: https://nodejs.org/en/download/

### Install express-generator

> While not mandatory, with express-generator we will very easily be able to build the structure of our application.

- Install **express-generator** with the following command:
```
$ npm install express-generator -g
```
- Create the application. For this tutorial, we'll call **real-time-league**:
```
$ express real-time-league
```
- Then, install dependencies:
```
$ cd real-time-league
$ npm install
```

## Let's do it!
The application that we will build, will allow us to know in real time the results of football matches. And believe it or not, we will in a few steps ;)

First, we will install [RethinkDB](https://www.rethinkdb.com/docs/install/). Once installed, we access the [administration console](http://localhost:8080) and click on the menu [Data Explorer](http://localhost:8080/#dataexplorer).

### Creating our data model

Suppose, we have to create a database for the local football league where teams and different tournament games are stored. The structure of the tables would be the following:

```
team: {
	data_base: "league",
	table_name: "teams",
	description: "A team",
	data: {
		id: {
			description: "First three letters of the team's name"
		},
		name: {
			description: "Full team name"
		}
	},
	example: {
		id: "RIV",
		name: "River Plate"
	}
}
```
```
match: {
	data_base: "league",
	table_name: "matches",
	description: "A match",
	data: {
		id: {
			description: "Auto-generated ID"
		},
		home: {
			description: "Home team ID"
		},
		away: {
			description: "Away team ID"
		},
		goal_home: {
			description: "Number of home team's goals"
		},
		gol_away: {
			description: "Number of away team's goals"
		}	
	},
	example: {
		id: '76ab9148-a527-41a4-aeec-a6ea0994a167',
		local: "RIV",
		visitante: "BOC",
		goal_home: 1,
		goal_away: 0
	}
}
```
**Create the database**
```
r.dbCreate('league');
```
**Create tables**
```
r.db('league').tableCreate('teams');
r.db('league').tableCreate('matches');
```
**Insert documents**
```
r.db('league').table('teams').insert([
  { name: 'River', id: 'RIV' },
  { name: 'Boca', id: 'BOC' },
  { name: 'Racing', id: 'RAC' },
  { name: 'Independiente', id: 'IND' },
  { name: 'Huracan', id: 'HUR' },
  { name: 'San Lorenzo', id: 'SLA' },
  { name: 'Ferro', id: 'FER' },
  { name: 'Velez', id: 'VEL' }
]);

r.db('league').table('matches').insert([
 { home:'RIV', goal_home:0, away:'BOC', goal_away:0 }, 
 { home:'RAC', goal_home:0, away:'IND', goal_away:0 }, 
 { home:'HUR', goal_home:0, away:'SLA', goal_away:0 }, 
 { home:'VEL', goal_home:0, away:'FER', goal_away:0 }
]);
```


**You can query the tables if you wish:**

```
r.db('league').table('teams');
```
![](https://i.imgur.com/iqxeowD.png)

```
r.db('league').table('matches');
```
![](https://i.imgur.com/FzNt1OK.png)

### Install Socket.io
[Socket.IO](http://socket.io/) is a JavaScript library for realtime web applications. Socket.IO enables real-time bidirectional event-based communication. We will use Socket.IO to inform the customer of any changes in our database. To do so, within the project root execute the following command: `npm install socket.io --save`

### Installing RethinkDB client drivers
RethinkDB has several client drivers which can be found [here](https://rethinkdb.com/docs/install-drivers/). For our application, we use the official driver for JavaScript.
We installed with the following command in the root of the project:`npm install rethinkdb --save`

If you want, take a moment to review the previous steps and return here and continue with the example. In short, up here we did the following:
- [x] Install Node.JS and RethinkDB.
- [x] Create the skeleton of our application with express-generator.
- [x] Create the database and tables.
- [x] Install Socket.IO and RethinkDB client driver for JavaScript.

What are we missing?
- [ ] Connect to RethinkDB.
- [ ] Create the page for today's game.
- [ ] Updating a football match and receive real-time notification.

### Connecting the application to RethinkDB


We will work with the `www` file which is located inside the `real-time-league\bin\` folder.

```javascript
/**
 * Module dependencies.
 */
...
var rethinkdb = require('rethinkdb');
...
/**
 * Listen on provided port, on all network interfaces.
 */
...
var io = require('socket.io').listen(server);

/**
 * Connection Rethinkdb
 */

var configdb = {
    host: 'localhost'
    , port: 28015
};

var name_database = 'league';

//function to connect Rethinkdb
function connect() {
    var promise = new Promise(
        function (resolve, reject) {
            rethinkdb.connect(configdb, function (err, conn) {
                if (err) reject(err);
                resolve({
                    r: rethinkdb
                    , db: rethinkdb.db(name_database)
                    , conn: conn
                });
            });

        });
    return promise;
}

//execute function connect
connect().then(function (result) {
        console.log("app >> connection success to RethinkDB.");

        //store connection data as global variables
		app.set('r', result.r);
        app.set('db', result.db);
        app.set('conn', result.conn);

        //Start monitoring the table matches
        var db = result.db;
        var conn = result.conn;

        //Once an event occurs then...
        db.table("matches")
            .changes()
            .run(conn).then(function (cursor) {
                //for every modified document...
                cursor.each(function (err, item) {  
                    if (item && item.new_val) {
                        //We save the previous and new document
                        var match_old = item.old_val;
                        var match_new = item.new_val;
                        
                        //Find out who scored a goal
                        var team_goal = null;
                        if (match_old.goal_home + 1 === match_new.goal_home) {
                            team_goal = match_new.home;
                        } else {
                            team_goal = match_new.away;
                        }
                        //Notifies to the client the event...
                        io.sockets.emit("goal", {
                            'match': match_new
                            , 'team': team_goal
                        });
                    }
                });
            }) //An error occured while trying to access the table 'matches'
            .error(function (err) {
                console.log("An error occured while trying to access the table matches: " + err);
            });
    }) //An error occured while trying to connect to the db league
    .catch(function (err) {
        console.log("An error occured while trying to connect to the db: " + err);
    });

/**
 * Rules socket.io
 */

io.sockets.on('connection', function (socket) {
    console.log('user connected: ' + socket.id);

    //When a user connects to the application socket.io notifies to the user that the server is listening
    socket.emit('connected');
 
});

...
```

































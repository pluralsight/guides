# Introduction
Hello everyone, this tutorial aims to show you how in simple steps to create your first application in real time using Node.JS and RethinkDB.

RethinkDB, is an open source database-oriented documents (using JSON format) which can be used in building applications as it integrates easily from drivers languages like Javascript, Java, Ruby and Python.

RethinkDB, is known for investing the architecture of traditional database to support the development of real-time applications of non-invasive, simple and efficient. Thus instead of performing constant polling for data changes, you can delegate this task to RethinkDB get updates whenever a change occurs. As a result, we can develop applications in less complex and easier to maintain and scale real time.


For the development of the application we will use the following technologies:
- Node.JS [4.2.4]
- Express [4.13.1]
- Jade [1.11.0]
- Socket.IO [1.4.6]
- RethinkDB [2.3.2]

## Requirements

### Install Node.js
To learn how to install Node.JS, follow the link below: https://nodejs.org/en/download/

### Install express-generator to create an application skeleton
While not mandatory, with Express-Generator we will very easily be able to build the structure of our application.

`npm install express-generator -g`


`express real-time-league`


```
create : real-time-league
   create : real-time-league/package.json
   create : real-time-league/app.js
   create : real-time-league/public/stylesheets
   create : real-time-league/public/stylesheets/style.css
   create : real-time-league/public/javascripts
   create : real-time-league/public/images
   create : real-time-league/public
   create : real-time-league/routes
   create : real-time-league/routes/index.js
   create : real-time-league/routes/users.js
   create : real-time-league/views
   create : real-time-league/views/index.jade
   create : real-time-league/views/layout.jade
   create : real-time-league/views/error.jade
   create : real-time-league/bin
   create : real-time-league/bin/www

   install dependencies:
     $ cd real-time-league && npm install

   run the app:
     $ DEBUG=real-time-league:* npm start

```

Execute the above commands to install the dependencies and run the application.

## Install Socket.io

`
npm install socket.io --save
`
Installs Socket.io and save it to the file  `package.json`

## Install client driver RethinkDB
`npm install rethinkdb --save`


## Install Rethinkdb

On windows 
https://www.rethinkdb.com/docs/install/windows/

## Create Database Model

Supongamos, que tenemos que crear una base de datos para la liga de futbol local donde se almacenen los equipos y los diferentes partidos del torneo. La estructura de las tablas seria la siguiente:

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
## Create database

1. Go to dataexplorer >> http://localhost:8080/#dataexplorer

2. Create the database

        `r.dbCreate('league');`


3. Create tables

        ```
        r.db('league').tableCreate('teams');
        r.db('league').tableCreate('matches');
        ```

4. Insert documents

**TEAMS**
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
```

**MATCHES**
```
r.db('league').table('matches').insert([
 { home:'RIV', goal_home:0, away:'BOC', goal_away:0 }, 
 { home:'RAC', goal_home:0, away:'IND', goal_away:0 }, 
 { home:'HUR', goal_home:0, away:'SLA', goal_away:0 }, 
 { home:'VEL', goal_home:0, away:'FER', goal_away:0 }
]);
```

You can query the tables if you wish

```
r.db('league').table('teams');
```
![](https://i.imgur.com/iqxeowD.png)

```
r.db('league').table('matches');
```
![](https://i.imgur.com/FzNt1OK.png)

Next, we will create th page that will display the matches and update the information for view results in real time.

Then, you need to edit the file `www` in: 
`<application-home>/bin/`

```
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
        console.log("app >> connection success to Rethinkdb.");

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

## Link application to RethinkDB

1. Create a folder 'services' 

Crear dentro del proyecto, una carpeta llamada services y adentro crear el archivo 

rethinkdb.service.js el cual nos permitira realizar consultas hacia la base de datos.






































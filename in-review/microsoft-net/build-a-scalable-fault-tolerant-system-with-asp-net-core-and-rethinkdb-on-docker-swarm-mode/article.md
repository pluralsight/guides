This guide shows you how to use RethinkDB with ASP.NET Core and how to deploy your app along with a RethinkDB cluster using Docker for Windows and Docker Swarm mode. 
We will be using the TokenGen app from the [Scale ASP.NET Core apps with Docker Swarm Mode](http://tutorials.pluralsight.com/microsoft-net/scale-asp-net-core-apps-with-docker-swarm-mode) guide and store the generated tokens along with the issuers in a RethinkDB database.

RethinkDB is an open source distributed document-oriented database. As most NoSQL databases, it stores JSON documents with no required schema or table structure. 

What makes RethinkDB great:
- horizontal scaling 
- table replication and sharing
- automatic fail-over
- query language with join support, map-reduce and geospatial queries
- pub/sub for data changes

If you want a scalable, fault tolerant database system for your ASP.NET Core applications then you should consider using RethinkDB. 

## Setting up a RethinkDB Cluster with Docker Swarm Mode

First we need to enable Swarm Mode and create a dedicated network for our RethinkDB cluster:

```powershell
# initialize swarm
docker swarm init

# create RethinkDB overlay network
docker network create --driver overlay rdb-net
```

We start building our RethinkDB cluster by running a single RethinkDB server, we will remove this instance later on:

```powershell
# create and start rethinkdb primary 
docker service create --name rdb-primary --network rdb-net --replicas 1 rethinkdb:latest rethinkdb --bind all --no-http-admin
```

Now we can create a secondary RethinkDB node that will join the `rdb-primary` node and form a cluster:

```powershell
# create and start rethinkdb secondary
docker service create --name rdb-secondary --network rdb-net --replicas 1 rethinkdb:latest rethinkdb --bind all --no-http-admin --join rdb-primary
```

Scale the secondary node so we can have a minimum of 3 nodes needed for RethinkDB automatic fail-over mechanism:

```powershell
# up 3 nodes (primary + two secondary) to enable automatic failover
docker service scale rdb-secondary=2
```

We now have a functional RethinkDB cluster, but we are not done yet. 
Because we started the primary node without a join command, our cluster has a single point of failure.
If for some reason `rdb-primary` container crashes, the Docker Swarm engine will recreate and start this container, but he can't join the existing cluster. If we start new `rdb-secondary` instances, they will join the new `rdb-primary` container and form another cluster.

To resolve this issue we have to remove the `rdb-primary` service and recreate it with the `join` command like so:

```powershell
# remove primary
docker service rm rdb-primary

# recreate primary with --join flag
docker service create --name rdb-primary --network rdb-net --replicas 1 rethinkdb:latest rethinkdb --bind all --no-http-admin --join rdb-secondary
```

Now we can also scale the primary node:

```powershell
# start two rdb-primary instances
docker service scale rdb-primary=2
```

At this point we have 4 nodes in our cluster, two `rdb-primary` and two `rdb-secondary`. We can further scale any of these two services and they will all join our cluster. If a `rdb-primary` or `rdb-secondary` instance crashes, the Docker Swarm will automatically start another container that will join our current cluster.

Last step is to create a RethinkDB proxy node, we expose port 8080 for the web admin and port 28015 so we can connect to the cluster from our app. Note that you don't have to expose port 28015 if your application runs inside the swarm. I'm exposing this port so I can connect to the cluster while debugging in VS.NET.

```powershell
# create and start rethinkdb proxy 
docker service create --name rdb-proxy --network rdb-net --publish 8080:8080 --publish 28015:28015 rethinkdb:latest rethinkdb proxy --bind all --join rdb-primary
```

Open a browser and navigate to `http://localhost:8080` to check the cluster state. In the servers page you should see 4 servers connected to the cluster.

if we run `docker service ls` we should get:

```
ID            NAME           REPLICAS  IMAGE             COMMAND
157bd7yg7d60  rdb-secondary  2/2       rethinkdb:latest  rethinkdb --bind all --no-http-admin --join rdb-primary
41eloiad4jgp  rdb-primary    2/2       rethinkdb:latest  rethinkdb --bind all --no-http-admin --join rdb-secondary
67oci5m1wksi  rdb-proxy      1/1       rethinkdb:latest  rethinkdb proxy --bind all --join rdb-primary
```

## Connecting to RethinkDB from ASP.NET Core

Open the TokenGen project from the previous [guide](http://tutorials.pluralsight.com/microsoft-net/scale-asp-net-core-apps-with-docker-swarm-mode) and install the RethinkDB driver NuGet package:

```
Install-Package RethinkDb.Driver
```

We are going to create an object that holds the RethinkDB cluster address. Add a class named `RethinkDbOptions` inside TokenGen project:

```cs
    public class RethinkDbOptions
    {
        public string Host { get; set; }
        public int Port { get; set; }
        public string Database { get; set; }
        public int Timeout { get; set; }
    }
```

We will be storing the connection data in `appconfig.json` and use a `RethinkDbOptions` object to inject the data into `RethinkDbConnectionFactory`. 
The `RethinkDbConnectionFactory` will provide a persistent connection to the RethinkDB cluster for our app:

```cs
    public class RethinkDbConnectionFactory : IRethinkDbConnectionFactory
    {
        private static RethinkDB R = RethinkDB.R;
        private Connection conn;
        private RethinkDbOptions _options;

        public RethinkDbConnectionFactory(IOptions<RethinkDbOptions> options)
        {
            _options = options.Value;
        }

        public Connection CreateConnection()
        {
            if (conn == null)
            {
                conn = R.Connection()
                    .Hostname(_options.Host)
                    .Port(_options.Port)
                    .Timeout(_options.Timeout)
                    .Connect();
            }

            if(!conn.Open)
            {
                conn.Reconnect();
            }

            return conn;
        }

        public void CloseConnection()
        {
            if (conn != null && conn.Open)
            {
                conn.Close(false);
            }
        }

        public RethinkDbOptions GetOptions()
        {
            return _options;
        }
    }
```

Open `appsettings.json` and add the RethinkDB cluster connection data:

```json
  "RethinkDbDev": {
    "Host": "localhost",
    "Port": 28015,
    "Timeout": 10,
    "Database": "TokenStore"
  }
```

Now we can create a singleton instance of `RethinkDbConnectionFactory` in `Startup.cs`. You should reuse the same connection across whole application since the `RethinkDb` connection is thread safe. 

```cs
public void ConfigureServices(IServiceCollection services)
{
    //....
	
    services.Configure<RethinkDbOptions>(Configuration.GetSection("RethinkDbDev"));
    services.AddSingleton<IRethinkDbConnectionFactory, RethinkDbConnectionFactory>();
}
```

Test the connection inside `Startup.Configure` method:

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory,
    IRethinkDbConnectionFactory connectionFactory)
{
    //....

    var con = connectionFactory.CreateConnection();
    con.CheckOpen();
}
```

##  Database initialization

The TokenGen app logic involves two entities: token and issuer. Tokens are issued by an app instance (issuer) to a client by calling the `/api/token` endpoint. We will create a database to store these two entries.

First we create a class for each entity:

```cs
public class Issuer
{
    [JsonProperty("id", NullValueHandling = NullValueHandling.Ignore)]
    public string Id { get; set; }
    public string Name { get; set; }
    public string Version { get; set; }
    public DateTime Timestamp { get; set; }
}

public class Token
{
    [JsonProperty("id", NullValueHandling = NullValueHandling.Ignore)]
    public string Id { get; set; }
    public DateTime Expires { get; set; }
    public string Issuer { get; set; }
}
```

Now we will create a service that uses the RethinkDb connection factory and implements the database initialization logic:

```cs
public class RethinkDbStore : IRethinkDbStore
{
    private static IRethinkDbConnectionFactory _connectionFactory;
    private static RethinkDB R = RethinkDB.R;
    private string _dbName;

    public RethinkDbStore(IRethinkDbConnectionFactory connectionFactory)
    {
        _connectionFactory = connectionFactory;
        _dbName = connectionFactory.GetOptions().Database;
    }

    public void InitializeDatabase()
    {
        // database
        CreateDb(_dbName);

        // tables
        CreateTable(_dbName, nameof(Token));
        CreateTable(_dbName, nameof(Issuer));

        // indexes
        CreateIndex(_dbName, nameof(Token), nameof(Token.Issuer));
        CreateIndex(_dbName, nameof(Issuer), nameof(Issuer.Name));

    }

    protected void CreateDb(string dbName)
    {
        var conn = _connectionFactory.CreateConnection();
        var exists = R.DbList().Contains(db => db == dbName).Run(conn);

        if (!exists)
        {
            R.DbCreate(dbName).Run(conn);
            R.Db(dbName).Wait_().Run(conn);
        }
    }

    protected void CreateTable(string dbName, string tableName)
    {
        var conn = _connectionFactory.CreateConnection();
        var exists = R.Db(dbName).TableList().Contains(t => t == tableName).Run(conn);
        if (!exists)
        {
            R.Db(dbName).TableCreate(tableName).Run(conn);
            R.Db(dbName).Table(tableName).Wait_().Run(conn);
        }
    }

    protected void CreateIndex(string dbName, string tableName, string indexName)
    {
        var conn = _connectionFactory.CreateConnection();
        var exists =  R.Db(dbName).Table(tableName).IndexList().Contains(t => t == indexName).Run(conn);
        if (!exists)
        {
            R.Db(dbName).Table(tableName).IndexCreate(indexName).Run(conn);
            R.Db(dbName).Table(tableName).IndexWait(indexName).Run(conn);
        }
    }

    public void Reconfigure(int shards, int replicas)
    {
        var conn = _connectionFactory.CreateConnection();
        var tables = R.Db(_dbName).TableList().Run(conn);
        foreach (string table in tables)
        {
            R.Db(_dbName).Table(table).Reconfigure().OptArg("shards", shards).OptArg("replicas", replicas).Run(conn);
            R.Db(_dbName).Table(table).Wait_().Run(conn);
        }
    }
}
```

The `InitializeDatabase` method checks if the TokenStore database, tables and indexes are present on the cluster, if they are missing then it will create them. Because these operations are expensive you should call this method only once at application start.

On application startup, create a singleton instance of `IRethinkDbStore` and invoke `InitializeDatabase`:

```cs
public void ConfigureServices(IServiceCollection services)
{
    //...

    services.Configure<RethinkDbOptions>(Configuration.GetSection("RethinkDbDev"));
    services.AddSingleton<IRethinkDbConnectionFactory, RethinkDbConnectionFactory>();
 
    services.AddSingleton<IRethinkDbStore, RethinkDbStore>();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory, 
    IRethinkDbStore store)
{
    //...

    store.InitializeDatabase();
}
```

When the `InitializeDatabase` runs, the tables are created by default with one shard and one replica. We've setup earlier a RethinkDB cluster with 4 nodes, to make our tables fault tolerant we need to replicate our data on all 4 nodes. In order to do this, after the database initialization we can call `store.Reconfigure(shards = 1, replicas = 4)`.

## CRUD Operations

When a TokenGen app instance starts it has to register in the database as an issuer, we will use the `MachineName` to identity the issuer. When docker starts a new container, the `MachineName` gets populated with the container ID, so we will use this ID to track the load of each app instance.

First we create a method to persist issuers:

```cs
public class RethinkDbStore : IRethinkDbStore
{
    //..

    public string InsertOrUpdateIssuer(Issuer issuer)
    {
        var conn = _connectionFactory.CreateConnection();
        Cursor<Issuer> all = R.Db(_dbName).Table(nameof(Issuer))
            .GetAll(issuer.Name)[new { index = nameof(Issuer.Name) }]
            .Run<Issuer>(conn);

        var issuers = all.ToList();

        if (issuers.Count > 0)
        {
            // update
            R.Db(_dbName).Table(nameof(Issuer)).Get(issuers.First().Id).Update(issuer).RunResult(conn);

            return issuers.First().Id;
        }
        else
        {
            // insert
            var result = R.Db(_dbName).Table(nameof(Issuer))
                .Insert(issuer)
                .RunResult(conn);

            return result.GeneratedKeys.First().ToString();
        }
    }
}
```

This method uses the `Issuer.Name` secondary index we created earlier to search for an issuer by name. If the issuer exists, it will update the version and timestamp, else it will insert a new one. 
Note that the update will only happen when you run the app from Visual Studio. On docker swarm, when you scale up a service, each instance has a unique ID.

We can now call this method on application start:

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory, 
    IRethinkDbStore store)
{
    //...

    // create TokenStore database, tables and indexes if not exists
    store.InitializeDatabase();

    // register issuer
    store.InsertOrUpdateIssuer(new Issuer
    {
        Name = Environment.MachineName,
        Version = PlatformServices.Default.Application.ApplicationVersion,
        Timestamp = DateTime.UtcNow
    });
}
```

Now that we've implemented the issuers persistence we can do the same for the tokens by adding the following method to `RethinkDbStore`:

```cs
public void InserToken(Token token)
{
    var conn = _connectionFactory.CreateConnection();
    var result = R.Db(_dbName).Table(nameof(Token))
        .Insert(token)
        .RunResult(conn);
}
```

Edit the `TokenController` and call `InsertToken` before sending the token to the client:

```cs
[Route("api/[controller]")]
public class TokenController : Controller
{
    private IRethinkDbStore _store;

    public TokenController(IRethinkDbStore store)
    {
        _store = store;
    }

    [HttpGet]
    public Token Get()
    {
        var token = new Token
        {
            Id = Guid.NewGuid().ToString(),
            Expires = DateTime.UtcNow.AddHours(1),
            Issuer = Environment.MachineName
        };

        _store.InserToken(token);

        return token;
    }
}
```

What we've done so far is registering an app instance as issuer at startup and persist generated tokens. Let's proceed further by creating a report with each issuer and the numbers of tokens generated.

First we create a `IssuerStatus` model to hold our report data:

```cs
    public class IssuerStatus
    {
        public string Name { get; set; }
        public string Version { get; set; }
        public DateTime RegisterDate { get; set; }
        public long TotalTokensIssued { get; set; }
    }
``` 

Then we we create a method in `RethinkDbStore` to build our report:

```cs
public List<IssuerStatus> GetIssuerStatus()
{
    var conn = _connectionFactory.CreateConnection();
    Cursor<Issuer> all = R.Db(_dbName).Table(nameof(Issuer)).RunCursor<Issuer>(conn);
    var list = all.OrderByDescending(f => f.Timestamp)
        .Select(f => new IssuerStatus
        {
            Name = f.Name,
            RegisterDate = f.Timestamp,
            Version = f.Version,
            TotalTokensIssued = R.Db(_dbName).Table(nameof(Token))
                    .GetAll(f.Name)[new { index = nameof(Token.Issuer) }]
                    .Count()
                    .Run<long>(conn)
        }).ToList();

    return list;
}
```

Note are using the `Token.Issuer` secondary index to count all tokens belonging to the same issuer.

We expose this report over the TokenGen API by creating `IssuerController`:

```cs
[Route("api/[controller]")]
public class IssuerController : Controller
{
    private IRethinkDbStore _store;

    public IssuerController(IRethinkDbStore store)
    {
        _store = store;
    }

    [HttpGet]
    public dynamic Get()
    {
        return _store.GetIssuerStatus();
    }
}
```

## Running the app service on Docker Swarm 

Let's deploy the TokenGen app to docker swarm like we did in the previous guide. Create a Powershell script with the following content: 

```powershell
# build tokengen image
if(docker images -q tokengen-img){
    "using existing tokengen image" 
}else{
    docker build -t tokengen-img -f TokenGen.dockerfile .
}

# create and start tokengen service
docker service create --publish 5000:5000 --name tokengen --network rdb-net tokengen-img

# wait for the database initialization to finish
Start-Sleep -s 10

# scale x3
docker service scale tokengen=3
```

Note that we are using the `--network rdb-net` param, so our app service will share the same network with the RethinkDB cluster. Before running the script we have to add a new RethinkDB connection data to the `appsetting.json` file.
When our app runs inside docker swarm, the address of the RethinkDB proxy should be the name of the proxy service `rdb-proxy`.
Best is to create a dedicated entry for the staging environment like this:

```json
  "RethinkDbStaging": {
    "Host": "rdb-proxy",
    "Port": 28015,
    "Timeout": 10,
    "Database": "TokenStore"
  }
```

Then modify the `Startup.cs` to load these options if environment is not development:

```cs
public void ConfigureServices(IServiceCollection services)
{
    //...

    if (_env.IsDevelopment())
    {
        services.Configure<RethinkDbOptions>(Configuration.GetSection("RethinkDbDev"));
    }
    else
    {
        services.Configure<RethinkDbOptions>(Configuration.GetSection("RethinkDbStaging"));
    }
    services.AddSingleton<IRethinkDbConnectionFactory, RethinkDbConnectionFactory>(); 
    services.AddSingleton<IRethinkDbStore, RethinkDbStore>();
}
```

Note that we've set `ENV ASPNETCORE_ENVIRONMENT="Staging"` in `TokenGen.dockerfile`.

After running the deployment script use `docker service ls` command, you should get the following result:

```
ID            NAME           REPLICAS  IMAGE             COMMAND
0184a1ou12ue  tokengen       3/3       tokengen-img
157bd7yg7d60  rdb-secondary  2/2       rethinkdb:latest  rethinkdb --bind all --no-http-admin --join rdb-primary
41eloiad4jgp  rdb-primary    2/2       rethinkdb:latest  rethinkdb --bind all --no-http-admin --join rdb-secondary
67oci5m1wksi  rdb-proxy      1/1       rethinkdb:latest  rethinkdb proxy --bind all --join rdb-primary
```

If you access the RethinkDB web UI at `http://localhost:8080` you can notice that the `TokenStore` database has been created and the `Token` and `Issuer` tables have 4 replicas each.

## Testing the Docker Swarm load balancer 

Create a load test for our cluster to see how Docker Swarm distributes the load between our app instances. For this purpose we will use a Powershell workflow that will call `api/token` endpoint in parallel, from 10 threads. After the load tests finishes, we'll call `api/issuer` endpoint to see how many tokens each app instance has generated.

```powershell
workflow loadtest{
    Param($Iterations)

    $array = 1..$Iterations

    foreach -Parallel -ThrottleLimit 10 ($i in $array){
        Invoke-RestMethod http://localhost:5000/api/token
    }

    Invoke-RestMethod http://localhost:5000/api/issuer
}

loadtest 500
```

This is the load test result on my PC:

```
name              : d04aad77448a
version           : 1.0.1.0
registerDate      : 2016-08-17T16:53:40.321Z
totalTokensIssued : 160

name              : f6adf92273de
version           : 1.0.1.0
registerDate      : 2016-08-17T16:53:40.22Z
totalTokensIssued : 161

name              : cf3d85de52b4
version           : 1.0.1.0
registerDate      : 2016-08-17T16:53:31.62Z
totalTokensIssued : 179
```

As you can see, the load balancer did a great job. We requested 500 tokens and each app container serviced a fair share of tokens.

## Testing fault tolerance

We can simulate failures in our system by using the `docker kill <container-id>` command. First let's run `docker ps` to find out our containers ids:

```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS
d04aad77448a        tokengen-img:latest   "dotnet run"             2 hours ago         Up 2 hours
f6adf92273de        tokengen-img:latest   "dotnet run"             2 hours ago         Up 2 hours
cf3d85de52b4        tokengen-img:latest   "dotnet run"             2 hours ago         Up 2 hours
18d0548c1dae        rethinkdb:latest      "rethinkdb proxy --bi"   3 hours ago         Up 3 hours
7f14aa948092        rethinkdb:latest      "rethinkdb --bind all"   3 hours ago         Up 3 hours
c38e7337d9f6        rethinkdb:latest      "rethinkdb --bind all"   3 hours ago         Up 3 hours
9fbbc3b72c18        rethinkdb:latest      "rethinkdb --bind all"   3 hours ago         Up 3 hours
bb0c58d3c26d        rethinkdb:latest      "rethinkdb --bind all"   3 hours ago         Up 3 hours
```

While the load test is running let's kill an app instance using `docker kill d04aad77448a`. 
If we run `docker ps` once more, we'll notice that the Docker Swarm engine has detected that one of the tokengen instance is down so he promptly started another one.

```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS
ec585e44960e        tokengen-img:latest   "dotnet run"             4 seconds ago       Up 1 seconds
d04aad77448a        tokengen-img:latest   "dotnet run"             2 hours ago         Exited (137) 7 seconds ago
f6adf92273de        tokengen-img:latest   "dotnet run"             2 hours ago         Up 2 hours
cf3d85de52b4        tokengen-img:latest   "dotnet run"             2 hours ago         Up 2 hours
``` 

As soon as the new instance is up, the load balancer will start distribute requests to it. You'll notice that id `ec585e44960e` is showing up in the issuers load test result.

Next let's see how RethinkDB cluster can cope with server failures. Open the RethinkDB web UI and go to the `Token` table details page. Locate the `Servers used by this table` section and look for the server name that holds the `Primary replica`. The server name should be in this format `<container id>_<random letters>`. 

![rdb-1](https://raw.githubusercontent.com/pluralsight/guides/master/images/4b112828-8d96-419e-a196-cab39a6c0fca.png)

Now let's bring down the server that holds the primary replica with `docker kill 7f14aa948092`. As soon as the server becomes unavailable, the RethinkDB automatic fail-over kicks in and elects one of the remaining  servers to act as primary replica.

![rdb-2](https://raw.githubusercontent.com/pluralsight/guides/master/images/78bbe5ee-9c22-4bba-8795-24511a4dd669.png)

In the same time, Docker Swarm has lunch a new RethinkDB server to replace the one that went down. If you look in the RethinkDB web UI server list, you'll notice that we still have 4 servers, but one of them doesn't hold any data. 

![rdb-3](https://raw.githubusercontent.com/pluralsight/guides/master/images/9bf3080d-ebeb-4dff-8af8-38ffbc181a7c.png)

In order to fully restore our cluster state, we have to reconfigure the replicas and expand to the new server. 
For this purpose I've created a `RdbController` in TokenGen app that calls `RethinkDbStore.Reconfigure` method:

```cs
[Route("api/[controller]")]
public class RdbController : Controller
{
    private IRethinkDbStore _store;

    public RdbController(IRethinkDbStore store)
    {
        _store = store;
    }

    [Route("[action]/{shards:int}/{replicas:int}")]
    public void Reconfigure(int shards, int replicas)
    {
        _store.Reconfigure(shards, replicas);
    }
}
``` 

We can now call the `api/rdb/reconfigure` from Powershell like this:

```powershell
Invoke-RestMethod http://localhost:5000/api/rdb/reconfigure/1/4
```

![rdb-4](https://raw.githubusercontent.com/pluralsight/guides/master/images/a9bfe222-919c-426c-90b9-c11dbe8d9447.png)

Reconfiguration is done and our cluster is healthy again. 

## Conclusion

Scalability and fault tolerance can be easily achieved nowadays if you use the right tools. For a production environment, you'll have to setup a DNS server, a reverse proxy and at least 3 Docker Swarm nodes to host the app and RethinkDB services. This will be the subject for another article.

The TokenGen project and all PowerShell scripts can be found on GitHub in this public [repository](https://github.com/stefanprodan/aspnetcore-dockerswarm).

Thanks for reading this guide. Please leave all comments and feedback in the discussion section below.

You can also follow me on twitter [@stefanprodan](https://twitter.com/stefanprodan)

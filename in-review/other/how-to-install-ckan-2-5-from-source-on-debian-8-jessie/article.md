This manual is based on the [CKAN official installation manual](http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html): 

We will be using a fresh Debian 8 system with minimum installation options. Without user interface and no web server. This installation is only for testing or development purposes.

## 1. Install the required packages

Login as root and install the sudo package to run the command line instructions with super user permissions safely:

```bash
apt-get install sudo
```

Logout as root and login with the user that will install and run CKAN. Then install the required packages for CKAN:

```bash
sudo apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core solr-jetty
```

It is not necessary to install the openjdk-6-jdk package because Debian 8 comes with open-jdk-7 pre-installed, and has no issue running the solr server

## 2. Install CKAN into a Python virtual environment

Create a Python virtual environment (virtualenv) to install CKAN into, and activate it. In the following command line text you have to substitute the 'username' with the name of the user that will run the CKAN application:

```bash
sudo mkdir -p /usr/lib/ckan/default

sudo chown <username> /usr/lib/ckan/default

virtualenv --no-site-packages /usr/lib/ckan/default

. /usr/lib/ckan/default/bin/activate
```

The final command above activates your virtualenv. The virtualenv has to remain active for the rest of the installation and deployment process, or commands will fail. You can tell when a virtualenv is active because its name appears in front of your shell prompt, and will look something like this:

```sh
(default) $ _
```

For example, if you close your current terminal session, your virtualenv will no longer be activated. You can always reactivate the virtualenv with this command:

```sh
. /usr/lib/ckan/default/bin/activate

```
Install the CKAN source code into your virtualenv. To install the latest stable release of CKAN (CKAN 2.5.2), run:

```bash
pip install -e 'git+https://github.com/ckan/ckan.git@ckan-2.5.2#egg=ckan'
```

We will test the installation with the 2.5.2 version. If you want to install another version just change the CKAN version in the previous command line text.

If you’re installing CKAN for development, you may want to install the latest development version (the most recent commit on the master branch of the CKAN git repository). In that case, run this command instead:

```bash
pip install -e 'git+https://github.com/ckan/ckan.git#egg=ckan'
```

Just remember that this version may contain bugs.

Install the Python modules that CKAN requires into your virtualenv:

```bash
pip install -r /usr/lib/ckan/default/src/ckan/requirements.txt
```

Deactivate and reactivate your virtualenv, to make sure you’re using the virtualenv’s copies of commands like paster rather than any system-wide installed copies:

```bash
deactivate

. /usr/lib/ckan/default/bin/activate
```

## 3. Setup a PostgreSQL database

List existing databases:

```bash
sudo -u postgres psql -l
```

Check that the encoding of any existing database is UTF8, else you may run into problems with non-ascii characters (e.g. 蠨,ဂ). Since changing the encoding of PostgreSQL may mean deleting existing databases, it is suggested that you remedy this before continuing with the CKAN install.

Next you’ll need to create a database user if one doesn’t already exist. Create a new PostgreSQL database user called ckan_default, and enter a password for the user when prompted. You’ll need this password later:

```bash
sudo -u postgres createuser -S -D -R -P ckan_default
```

Create a new PostgreSQL database, called ckan_default, owned by the database user you just created:

```bash
sudo -u postgres createdb -O ckan_default ckan_default -E utf-8
```

>Note:
>
>If PostgreSQL is run on a separate server, you will need to edit `postgresql.conf` and `pg_hba.conf`. For PostgreSQL 9.1 on Ubuntu, these files are located in `/etc/postgresql/9.1/main`.

Uncomment the listen_addresses parameter and specify a comma-separated list of IP addresses of the network interfaces PostgreSQL should listen on or ‘*’ to listen on all interfaces. For example,

```bash
listen_addresses = 'localhost,192.168.1.21'
```

Add a line similar to the line below to the bottom of `pg_hba.conf` to allow the machine running Apache to connect to PostgreSQL. Please change the IP address as desired according to your network settings.

```
host all all 192.168.1.22/32 md5
```

## 4. Create a CKAN config file

Create a directory to contain the site’s config files:

```bash
sudo mkdir -p /etc/ckan/default

sudo chown -R username /etc/ckan/
```

Change the username with the name of your user.

#### Create the CKAN config file:

```bash
paster make-config ckan /etc/ckan/default/development.ini
```

Edit the `development.ini` file in a text editor, changing the following options:

```ini
sqlalchemy.url
```

This should refer to the database we created in step 3 Setup a PostgreSQL database above:

```
sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default
```

Replace pass with the password that you created in 3. Setup a PostgreSQL database above. If you’re using a remote host with password authentication rather than SSL authentication, use:

```python
sqlalchemy.url = postgresql://ckan_default:pass@<remotehost>/ckan_default?sslmode=disable
```

#### site_id:

Each CKAN site should have a unique `site_id`, for example:

```bash
ckan.site_id = default

site_url
```

Provide the site’s URL (used when putting links to the site into the FileStore, notification emails etc). For example:

```typescript
ckan.site_url = http://demo.ckan.org
```

## 5. Setup Solr

CKAN uses Solr as its search platform, and uses a customized Solr schema file that takes into account CKAN’s specific search needs. Now that we have CKAN installed, we need to install and configure Solr.

These instructions explain how to setup Solr with a single core. If you want multiple applications, or multiple instances of CKAN, to share the same Solr server then you probably want a multi-core Solr setup instead. See [Multicore Solr Setup](http://docs.ckan.org/en/latest/maintaining/solr-multicore.html).

These instructions explain how to deploy Solr using the Jetty web server, but CKAN doesn’t require Jetty - you can deploy Solr to another web server, such as Tomcat, if that’s convenient on your operating system.

Edit the Jetty configuration file (`/etc/default/jetty`) and change the following variables:

```roboconf
NO_START=0 # (line 4)

JETTY_HOST=127.0.0.1 # (line 16)

JETTY_PORT=8983 # (line 19)
```

This `JETTY_HOST` setting will only allow connections from the same machine. If CKAN is not installed on the same machine as Jetty/Solr you will need to change it to the relevant host or to `0.0.0.0` (and probably set up your firewall accordingly).

Start the Jetty server:

```bash
sudo service jetty8 start
```

You should now see a welcome page from Solr if you open `http://localhost:8983/solr/` in your web browser (replace localhost with your server address if needed).

Replace the default `schema.xml` file with a symlink to the CKAN schema file included in the sources.

```bash
sudo mv /etc/solr/conf/schema.xml /etc/solr/conf/schema.xml.bak

sudo ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml /etc/solr/conf/schema.xml
```

Now restart Solr:

```bash
sudo service jetty8 restart
```

and check that Solr is running by opening `http://localhost:8983/solr/`.

Finally, change the `solr_url` setting in your CKAN config file to point to your Solr server, for example:

 ```bash
 solr_url=http://127.0.0.1:8983/solr
 ```

## 6. Create database tables

Now that you have a configuration file that has the correct settings for your database, you can create the database tables:

```bash
cd /usr/lib/ckan/default/src/ckan

paster db init -c /etc/ckan/default/development.ini
```

You should see:

`> Initialising DB: SUCCESS.`

## 7. Set up the DataStore

Setting up the DataStore is optional. However, if you do skip this step, the DataStore features will not be available and the DataStore tests will fail.

Follow the instructions in [DataStore extension](http://docs.ckan.org/en/latest/maintaining/datastore.html) to create the required databases and users, set the right permissions and set the appropriate values in your CKAN config file.

## 8. Link to who.ini

`who.ini` (the Repoze.who configuration file) needs to be accessible in the same directory as your CKAN config file, so create a symlink to it:

```bash
ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini
```

## 9. Running and testing ckan

You can now use the Paste development server to serve CKAN from the command-line. This is a simple and lightweight way to serve CKAN that is useful for development and testing:

```bash
cd /usr/lib/ckan/default/src/ckan

paster serve --daemon /etc/ckan/default/development.ini
```

Open `http://127.0.0.1:5000/` in a web browser, and you should see the CKAN front page.

#### To stop the server:

```bash
paster serve /etc/ckan/default/development.ini stop
```

Now that you’ve installed CKAN, you should:

Run CKAN’s tests to make sure that everything’s working, see [Testing CKAN](http://docs.ckan.org/en/latest/contributing/test.html).

If you want to use your CKAN site as a production site, not just for testing or development purposes, then deploy CKAN using a production web server such as Apache or Nginx. See [Deploying a source install](http://docs.ckan.org/en/latest/maintaining/installing/deployment.html).

Begin using and customizing your site, see [Getting started](http://docs.ckan.org/en/latest/maintaining/getting-started.html).

## 10. Start and stop ckan scripts

In order to facilitate the start and stop of the ckan application in your debian machine, you can create the following bash scripts:

#### start_ckan.sh:

```bash
 #!/bin/bash

sudo service jetty8 start

. /usr/lib/ckan/default/bin/activate

paster serve /etc/ckan/default/development.ini
```

#### stop_ckan.sh:

```bash
#!/bin/bash

paster serve /etc/ckan/default/development.ini stop

deactivate

sudo service jetty8 stop
```    

After the creation of the files you have set permissions for execution:

```bash
chmod 755 [path_to_the_script]/start_ckan.sh

chmod 755 [path_to_the_script]/stop_ckan.sh
```    
To start and stop all the ckan services just run this scripts:

#### To start: 

```bash
. [path_to_the_script]/start_ckan.sh
```

#### To stop 

```bash
. [path_to_the_script]/stop_ckan.sh
```

The start ckan script can be defined to run when the system start. For that just copy the start script to the `/etc/init.d/` directory, set permission for execution and execute the following command:

```bash
update-rc.d start_ckan.sh defaults
```
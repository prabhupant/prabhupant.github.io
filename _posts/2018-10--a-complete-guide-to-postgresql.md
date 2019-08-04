---
layout: post
---
PostgreSQL is nowadays all over the place — major open-source projects use PostgreSQL. So what is PostgreSQL?

PostgreSQL is a powerful, open-source object-relational database system that uses and extend the SQL language combined with many features that safely store and scale the most complicated data workloads. An object-relational database system means is a database management system similar to a relational database, but with an object-oriented database model: objects, classes, and inheritance are directly supported in database schemas and in the query language.

This feature of PostgreSQL, coupled with its stable and highly scalable architecture, makes it the go-to database for various projects.

# Installing PostgreSQL

According to the [official website](https://www.postgresql.org/download/linux/ubuntu/), there are two ways to install PostgreSQL — using PostgreSQL apt repository, or using the package already included in apt. Either way works fine.

## PostgreSQL apt repository

First, note your Ubuntu version. You can find it by

{% highlight bash %}
lsb_release -a
{% endhighlight %}

Then, create the file `/etc/apt/sources.list.d/pgdg.list` and add a line for the repository.

{% highlight bash %}
deb http://apt.postgresql.org/pub/repos/apt/ YOUR_UBUNTU_VERSION_HERE-pgdg main
{% endhighlight %}

Import the repository signing key, and update the package lists

{% highlight bash %}
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
{% endhighlight %}

## Included in the distribution

Ubuntu includes PostgreSQL by default. To install PostgreSQL on Ubuntu, use the apt-get (or other apt-driving) command:

{% highlight bash %}
sudo apt-get install postgresql-10
{% endhighlight %}

# Using PostgreSQL

PostgreSQL uses the concept of ‘roles’. Roles are like the user accounts in your OS, with each having a different type of access and rights, like superuser and such. By default, the user account postgres is associated with the default role.

Now, switch over to the postgres account on your server/system by typing

{% highlight bash %}
sudo -i -u postgres
{% endhighlight %}

Now, you will be able to see that you have switched to the postgres account. Run the Postgres prompt by typing

{% highlight bash %}
psql
{% endhighlight %}

You will now be in the postgres shell which will look something like

{% highlight bash %}
postgres=# \q
{% endhighlight %}

You can exit the shell by typing ‘\q’, as shown above

Congratulations! You now have PostgreSQL database successfully installed in your system.

Some of the basic postgress shell commands that will come in handy are:

{% highlight bash %}
\q - to exit the postgres shell
\l - to list all the created datbases
\du - to list all the roles
{% endhighlight %}

To create a database and delete a database, you can do

{% highlight bash %}
postgres=# CREATE DATABASE db_name WITH OWNER postgres;
postgres=# DROP DATABASE db_name
{% endhighlight %}

Don’t forget to put the semicolon after every postgres shell statement.

# Installing pgAdmin

pgAdmin is a popular and feature rich Open Source administration and development platform for PostgreSQL. It provides you with a GUI to interact with your data tables in PostgreSQL and a bunch of other stuff that would normally require a CLI.

In this post, I’m going to use Python3.6 to install pgAdmin, but the same commands will work fine with Python2.x, just change ‘pip3' to ‘pip’ and ‘python3’ to ‘python’.

Open a terminal and type

{% highlight bash %}
sudo apt-get install virtualenv python3-pip libpq-dev python3-dev
cd
virtualenv -p python3 pgadmin4
cd pgadmin4
source bin/activate

pip3 install https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v3.3/pip/pgadmin4-3.3-py2.py3-none-any.whl
{% endhighlight %}

Now you need to configure default paths and set it to a single-user mode in the local configuration file

{% highlight bash %}
vim lib/python3.6/site-packages/pgadmin4/config_local.py
{% endhighlight %}

Here I’m using vim but you can use any text editor you like, like nano or atom for instance.

Now in config_local.py, write and save and close

{% highlight bash %}
import os
DATA_DIR = os.path.realpath(os.path.expanduser(u'~/.pgadmin/'))
LOG_FILE = os.path.join(DATA_DIR, 'pgadmin4.log')
SQLITE_PATH = os.path.join(DATA_DIR, 'pgadmin4.db')
SESSION_DB_PATH = os.path.join(DATA_DIR, 'sessions')
STORAGE_DIR = os.path.join(DATA_DIR, 'storage')
SERVER_MODE = False
{% endhighlight %}

Now, run this command while in the pgadmin4 directory

{% highlight bash %}
python3 lib/python3.6/site-packages/pgadmin4/pgAdmin4.py
{% endhighlight %}

You can now access pgAdmin4 at http://localhost/5050. You can exit the server by Ctril+C.

Now to run pgAdmin again, you will have to type

{% highlight bash %}
cd ~/pgadmin4
source bin/activate
python3 lib/python3.6/site-packages/pgadmin4/pgAdmin4.py
{% endhighlight %}

To reduce this tedious typing work, we will make an executable bash file for running pgAdmin.

{% highlight bash %}
touch ~/pgadmin4/pgadmin4
chmod +x ~/pgadmin4/pgadmin4
vim ~/pgadmin4/pgadmin4
{% endhighlight %}

Now write in the file

{% highlight bash %}
#!/bin/bash
cd ~/pgadmin4
source bin/activate
python lib/python3.6/site-packages/pgadmin4/pgAdmin4.py
{% endhighlight %}

Now you simply run pgAdmin by

{% highlight bash %}
~/pgadmin4/pgadmin4
{% endhighlight %}

# Some general tweaks

Sometimes, you might get an error that postgres isn’t able to connect to the port 5432 (its default port). There are two workarounds that I found useful.

## First method

Navigate to `/etc/postgresql/10/main` where 10 is the version name of postgres. It can be different depending on the version you have installed. To find the version, do

{% highlight bash %}
psql --version
{% endhighlight %}

Now open the `postgresql.conf` file with root access

{% highlight bash %}
sudo vim postgresql.conf
{% endhighlight %}

Here, find the port, and make sure its binded to 5432, and localhost is ‘*’.

## Second method

Start pgAdmin

{% highlight bash %}
~/pgadmin4/pgadmin4
{% endhighlight %}

Here, create a new server according to the requirements of your project and enter the port accordingly. By default, postgres run on port 5432. So if nothing is mentioned, then make sure you enter 5432 as the port.

# Viewing data in PostgreSQL

Again, pgAdmin to our help.

Run pgAdmin, start your database server in the background, run the application, and do database queries to alter the tables and view changes.

Now in pgAdmin, expand your database tab and look under ‘Schemas’ for your tables. Select the table you want to view, and then right click, then view data. That’s it!

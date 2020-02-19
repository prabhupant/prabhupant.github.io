---
layout: post
title: "How to deploy a Django Application with uWSGI and Nginx on Linux"
---

For me, deploying my django project with uWSGI and Nginx was a hassle. I struggled a lot and there were some terms I did not understand. In this blog post, I will try to explain and simplify the deployment process.

Let's say that we have the following directory structure

```
django_projectss/
|
|--> project1/
|    |
|    --> project1_app/
|    |
|    --> project1/
|        |
|        --> settings.py
|
|--> project2/
|    |
|    --> project2_app/
|    |
|    --> project2/
|        |
|        --> settings.py

```
Here `django_projects` is the directory that contains two Django projects - `project1` and `project2`.

I prefer this directory structure because it is clean and I like to have all of my deployment files in the main project folder, like `project1` and `project2`, and then create a symlink to the `uwsgi` or `nginx` directories. This way the deployment files remain manageable under the same project directory and you don't have to especially go to the `nginx` or `uwsgi` directories in case you need to change something. 


1. First of all, we need to configure uWSGI. uWSGI configuration will require three files -

* `project1_uwsgi.ini` - This file lists the uWSGI configurations.
* `uwsgi.service` - This file describes how to manage the uWSGI service on our server.
* `uwsgi_params` - The official documentation describes this as `It’s convenience, nothing more!`. This file contains various special variables passed by the web server, in our case Nginx.

First, let's create a `project1_uwsgi.ini` file in the path `~/djang_projects/project1/`. In this file, write

```ini
# mysite_uwsgi.ini file
[uwsgi]
project = project1
# Django-related settings
# the base directory (full path)
chdir           = /home/prabhu/django_projects/project1
# Django's wsgi file
module          = project1.wsgi
# the virtualenv (full path)
home            = /home/prabhu/django_projects/project1/venv

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe)
socket          = /tmp/project1.sock
# ... with appropriate permissions - may be needed
chmod-socket    = 666
# clear environment on exit
vacuum          = true

```

Now create `uwsgi.service` file in the same directory. Copy the content below and paste it in that file

```systemd
[Unit]
Description=uWSGI Emperor service

[Service]
ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown ubuntu:www-data /run/uwsgi'
ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

In order to enable Nginx to communicate with your uWSGI server, you need a `uwsgi_params` file in the `/etc/nginx/` directory. Make sure this file is there (it will be there in general), but if isn't, you can create it. Copy the contents below in the file and save it.

```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

2. We are going to serve the django application in Emperor mode of uWSGI. Emperor mode lets you deploy multiple applications as instances called vassals and they are managed by an emperor process. Create a `vassals` directory exists in `/etc/uwsgi/`. In case there isn't one, create it using `mkdir vassals`. Now we need to create a symlink of our `project1_uwsgi.ini` file. Use absolute paths while creating symlinks because relative paths can result in broken links ([refer this](https://stackoverflow.com/a/17738756/12360506)). The syntax for creating symlink goes like `ln -s SOURCE TARGET`. So write 

```bash
ln -s ~/django_projects/project1/project1_uwsgi.ini /etc/uwsgi/vassals/project1_uwsgi.ini
```


3. Now we need to start with Nginx. Go to `/etc/nginx/sites-available` and create a file called `project1_nginx.conf` and inside it write

```conf
# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server unix:///tmp/project1.sock;
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8080;
    # the domain name it will serve for
    server_name project1.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    #location /media  {
    #    alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    #}

    #location /static {
    #    alias /home/prabhu/django_projects/project1/static; # your Django project's static files - amend as required
    #}
    
    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /home/prabhu/django_projects/project1/uwsgi_params; # the uwsgi_params file you installed
    }
}
```

Few things to note from this file -

* Make sure that `upstream` and `uwsgi_pass` names are the same. In case you want to deploy two projects, for project1 you can name `django1` and for project2 you can name `django2`, or whatever suits you well.
* We are serving our site on port 8080. Feel free to use any port you want.
* I'm planning to use this project as an API server, so I have commented the location of `static` files. Uncomment them if you need to use the static files

4. Now create its symlink to the `/etc/nginx/sites-enabled` directory

```bash
ln -s /etc/nginx/sites-available/project1_nginx.conf /etc/nginx/sites-enabled/project1_nginx.conf
```

5. Restart uWSGI and Nginx and you can see your project live on `project1.com`!

```bash
sudo service uwsgi restart
sudo service nginx restart
```

## FAQs

**Q1. What is the difference between WSGI, uWSGI and uwsgi?**

Ans. These three names are confusing, no doubt about that. This is one of the place where Python naming conventions got bad. To differentiate between them, remember -

* WSGI - a specification that defines how web servers communicates with a Python application.
* uWSGI - a Python WSGI server implementation used for running Python web applications. Django, Flask, etc all use uWSGI.
* uwsgi - a low level protocol that is used by uWSGI servers to communicate with other uWSGI servers or upstream servers. Using this protocol, our uWSGI server communicates with Nginx

**Q2. Why are we using Emperor mode?**

Ans. There are different modes for serving our web application on uWSGI. But when we want to deploy a big number of apps on a single server, or a group of servers, Emperor mode makes our life easier.

**Q3. What if I want to deploy multiple projects on the same server?**

Ans. Pretty simple. Like in our case we have `project1` and `project2`, follow the same steps for `project2`, make sure you change the project name in all the files like `project2_uwsgi.ini`. Now, same as you did for `project1`, create a nginx conf file for `project2` making sure that you use a different port number and `upstream` name in this new file is different. Refer to the Nginx section above.


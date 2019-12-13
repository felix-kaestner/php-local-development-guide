## PHP Local Development Guide

A simple setup and config to run a local web server to serve a php web.

The following is a simple guide to quickly bootstrap a php application on your local machine.
If you are not familiar with docker, this will help you to start a web application in development 
mode on your host machine via a simple web server. 
We will use `nginx` to run a web server and `supervisor` to start everything simultaneously.


### Prerequisites
For this purpose both `nginx` and `supervisor` need to be installed on your system.
This guide assumes you are using php 7.4. If not, then adjust the path inside the supervisor.conf with your given php version.

_optional_
Also make sure you have `composer` installed, if you want to quickly bootstrap an application via `composer create-project`.

**ATTENTION**

For this guide, we will call the application we want to develop `contao`. 
You will want to adjust and rename the files `supervisor.conf` and `contao` to the given name of your application.

#### `Ubuntu / Debian`
For any debian-based system (including Ubuntu) just run the following to install both nginx and supervisor

```bash
$ sudo apt install nginx supervisor mysql-server
```

You can substitute `mysql` with `mariadb` if you want.
Simply install `mariadb-server` instead.

If you want, you can install php, with a couple of extensions with:

````bash
sudo apt install -y php7.4 php7.4-{dom,gd,curl,intl,mbstring,mysql,xdebug,fpm,zip}
```

_optional_

Install composer and make it available globally with:
```bash
$ curl -sS https://getcomposer.org/installer | php && sudo mv composer.phar /usr/local/bin/composer
```
To check if the installation was successfull, execute:
````bash
$ composer -v
```

If you prefer a UI to handle the database configuration, you can also install `phpmyadmin`, just do the following:
```bash
$ export VER="4.9.1"
$ cd /tmp
$ curl -sL https://files.phpmyadmin.net/phpMyAdmin/${VER}/phpMyAdmin-${VER}-all-languages.tar.gz  |  tar -xvz
```

Move the resulting folder to `/usr/share/phpmyadmin` folder.

```bash
$ rm phpMyAdmin*.gz
$ sudo mv phpMyAdmin-* /usr/share/phpmyadmin
```

Create the temporary directory and give all the permissions to www-data user and group

```bash
$ sudo mkdir -p /var/lib/phpmyadmin/tmp
$ sudo chown -R www-data:www-data /var/lib/phpmyadmin
```

### Quickstart

Create a new application, e.g. with composer:
````bash
$ composer create-project contao/managed-edition $PWD/contao '4.8'
```

Symlink the application into the `/var/www` directory.
You can do so, by executing the following:

```bash
$ sudo ln -s $PWD/contao        /var/www/contao
```

In either case, make sure the permissions inside the `/var/www` directory are correct by executing:

```bash
$ sudo chmod -R 755  /var/www
```

Afterwards your `/var/www` directory should look similar to the following:

```bash
$ tree /var/www
/var/www
├── html
│   └── index.html
└── contao -> /.../contao

2 directories, 1 file
```

Then go on and configure mysql/mariadb with:

````bash
$ sudo mysql_secure_installation
```

####  Configuration

Both `nginx` and `supervisor` need to be configured. The configuration files are provided in this directory.

Adjust and copy the supervisor configuration `supervisor.conf` into `/etc/supervisor/conf.d`

```bash
$ sudo cp supervisor.conf /etc/supervisor/conf.d/
```
_Alternative_
Symlink it:
```bash
$ sudo ln -s $PWD/supervisor.conf /etc/supervisor/conf.d/supervisor.conf
```

Copy the nginx configuration `contao` into `/etc/nginx/sites-available`

```bash
$ sudo cp contao /etc/nginx/sites-available/
```

The `sites-available` directory of nginx should always contain the configurations of all available web servers. 
In contrast, the `sites-enabled` directory should contain symlinks to the configurations that are currently enabled.
We will create this by executing:

```bash
$ sudo ln -s /etc/nginx/sites-available/contao /etc/nginx/sites-enabled/contao
```

If you would then like to currently disable the ordered.online domain while running nginx for other web server, 
you could simply delete the symlink inside `sites-enabled`, while this would still reserve the configuration inside 
`sites-available` for you the be able to enable it again, by creating the symlink with the command above.

_Alternative_
Symlink it directly into sites-enabled:
```bash
$ sudo ln -s $PWD/contao /etc/nginx/sites-enabled/contao
```

####  Start

If you followed all the steps above, you are now able to run the local web server by simply running:

```bash
$ sudo supervisord -n -c  /etc/supervisor/supervisord.conf 
```

*NOTE:*

The configuration file `/etc/supervisor/supervisord.conf` should contain the lines

```bash
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

...

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

...

[include]
files = /etc/supervisor/conf.d/*.conf
```

You can check that it does so by executing:

```bash
$ cat /etc/supervisor/supervisord.conf
```

If supervisor is alread running and you changed something in the configuration file, simply restart all programms by
```bash
$ sudo supervisorctl reread
$ sudo supervisorctl update
$ sudo supervisorctl restart all
```
*Advanced*

You can also start supervisor with the ordered-online application directly by executing:
```bash
$ sudo /usr/bin/supervisord -n -c /etc/supervisor/conf.d/supervisord.conf  
```

Note that you may want to add the supervisord, supervisorctl and unix_http_server blocks mentioned above.
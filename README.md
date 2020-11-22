# How to Configure Nginx and Apache Together on Debian 10 Buster


## Update
```bash
sudo apt update && sudo apt -y dist-upgrade
```
## Debian package
```bash
sudo apt -y install vim curl gnupg2 ca-certificates lsb-release apt-transport-https wget git expect
```
## File compression
```bash
sudo apt install zstd 
```

## PHP
```bash
sudo apt install php php-cli php-mysql php-fpm php-pear php-json php-json-schema php-dom php-dompdf php-simplexml php-ssh2 php-xml php-xmlreader php-curl php-exif php-ftp php-gd php-iconv php-imagick php-json php-mbstring php-posix php-sockets php-tokenizer composer php-symfony-console php-symfony-filesystem php-symfony-finder php-symfony-process php-mbstring php-zip php-gd php-bz2 php-tcpdf php-dev
```
### Debian only installs version 2.6, and for phpMyAdmin you need a version >=2.9 (https://wiki.debian.org/Backports)
```bash
sudo apt -t buster-backports install php-twig
```
### Debug PHP https://xdebug.org/docs/install
```bash
sudo apt-get install php-xdebug

sudo find / -name xdebug
 # Output
    /var/lib/php/modules/7.3/cli/enabled_by_maint/xdebug
    /var/lib/php/modules/7.3/fpm/enabled_by_maint/xdebug
    /var/lib/php/modules/7.3/apache2/enabled_by_maint/xdebug
    /var/lib/php/modules/7.3/registry/xdebug 

# Configuration php-fpm Nginx
sudo vi /etc/php/7.3/fpm/php.ini

## add Lines https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug
[XDebug]
zend_extension=/var/lib/php/modules/7.3/registry/xdebug 
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
```

## Mysql
```bash
sudo apt install mariadb-common mariadb-server mariadb-client mariadb-backup
sudo service mysql start
sudo mysql_secure_installation  # /!\ WARNING -Do NOT change the root password  --> for the rest (Y)
sudo mysql -u root -p
    # in MySQL
    MariaDB [(none)]> use mysql
    MariaDB [mysql]> select user,host from mysql.user;
    # OUTPOUT
    +------+-----------+
    | user | host      |
    +------+-----------+
    | root | localhost |
    +------+-----------+

    # CREATE USER
    MariaDB [mysql]> CREATE USER 'niko'@'localhost' IDENTIFIED BY 'MyPassword';
    # GRANT ALL PRIVILEGES
    MariaDB [mysql]> GRANT ALL PRIVILEGES on *.* TO 'niko'@'localhost',
    MariaDB [mysql]> select user,host from mysql.user;
    # OUTPOUT
    +------+-----------+
    | user | host      |
    +------+-----------+
    | niko | localhost |
    | root | localhost |
    +------+-----------+
    2 rows in set (0.000 sec)

    # Display users without a password.
    MariaDB [mysql]> SELECT user FROM mysql.user WHERE password='';
    +------+
    | user |
    +------+
    | root |
    +------+
    1 row in set (0.000 sec)
```
## Apache2
```bash
sudo apt install apache2 libapache2-mod-php7.3 libgmp3-dev libpq-dev libapache2-mod-wsgi-py3
sudo service apache2 start
    # Check version PHP
    sudo vi /var/www/html/info.php
    # write the lige
    <?php phpinfo() ?>
    # Go to http://localhost/info.php to see php features
```

## PosgreSQL
```bash
sudo apt install postgresql postgresql-doc postgresql-contrib
sudo service postgresql start
sudo -u postgres -i
psql
    #in Postgres 
    postgres=# CREATE USER niko WITH PASSWORD 'Mypassword';
    # SHOW DATABASEs
    postgres=# \l
                                     List of databases
       Name    |  Owner   | Encoding  | Collate | Ctype |   Access privileges
    -----------+----------+-----------+---------+-------+-----------------------
     postgres  | postgres | SQL_ASCII | C       | C     |
     template0 | postgres | SQL_ASCII | C       | C     | =c/postgres          +
               |          |           |         |       | postgres=CTc/postgres
     template1 | postgres | SQL_ASCII | C       | C     | =c/postgres          +
               |          |           |         |       | postgres=CTc/postgres
    (3 rows)
    # another solution 
    postgres=# SELECT datname FROM pg_database;
      datname
    -----------
     postgres
     template1
     template0
    (3 rows)
    # show users [ROLES]
    postgres=# \du
                                       List of roles
     Role name |                         Attributes                         | Member of
    -----------+------------------------------------------------------------+-----------
     niko      |                                                            | {}
     postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
     # Make user 'niko'  as superUser
     postgres=# ALTER ROLE rolename WITH SUPERUSER;
    postgres=# \du
                                       List of roles
     Role name |                         Attributes                         | Member of
    -----------+------------------------------------------------------------+-----------
     niko      | Superuser                                                  | {}
     postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

## PhpMyAdmin
```bash
sudo apt install phpmyadmin
```

## PGadmin
```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
apt-get install -y pgadmin4  pgadmin4-apache2
```

## APACHE2 With NGINX
```bash
sudo mv /etc/apache2/ports.conf /etc/apache2/ports.conf.default
echo "Listen 8080" | sudo tee /etc/apache2/ports.conf
sudo a2dissite 000-default
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/001-default.conf
sudo vi /etc/apache2/sites-available/001-default.conf
    <VirtualHost *:8080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
sudo a2ensite 001-default
sudo service apache2 restart
```

## Nginx
```bash
sudo apt install nginx
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/001-default

    # Check version PHP
    php -v
    # outpout
    PHP 7.3.19-1~deb10u1 (cli) (built: Jul  5 2020 06:46:45) ( NTS )
    Copyright (c) 1997-2018 The PHP Group
    Zend Engine v3.3.19, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.3.19-1~deb10u1, Copyright (c) 1999-2018, by Zend Technologies
    # Start php7.3-fpm
    sudo service php7.3-fpm start
    # Secure php.ini
    sudo vi /etc/php/7.3/fpm/php.ini
    # Uncomment the line and check if cgi.fix_pathinfo=0
    # RESTART Nginx et php7.3-fpm

sudo vi /etc/nginx/sites-available/default
    server {
        listen 80;
        server_name _;
        index index.html index.htm index.php;
        location / {
            proxy_pass http://localhost:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
}

## nodeJS
https://github.com/nodesource/distributions/blob/master/README.md

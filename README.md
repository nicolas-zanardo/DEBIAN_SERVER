# How to Configure Nginx and Apache Together on Debian 10 Buster WSL2

## Restart WSL2 (powershell)

Get-Service LxssManager | Restart-Service

## Update

```bash
sudo apt update && sudo apt -y dist-upgrade  

or 

sudo apt update && apt full-upgrade && apt autoremove
```
## Debian package

```bash
sudo apt install -y vim curl gnupg2 ca-certificates lsb-release apt-transport-https gnupg-agent software-properties-common wget git expect libnss3-tools
```
## File compression

```bash
sudo apt install zstd zip unzip
```

## PHP

### 7.3 Stable

```bash
sudo apt install -y libapache2-mod-php libphp-embed php php-all-dev php-bcmath php-bz2 php-cgi php-cli php-common php-curl php-dev php-enchant php-fpm php-gd php-gmp php-imap php-interbase php-intl php-json php-ldap php-mbstring php-mysql php-odbc php-pgsql php-phpdbg php-pspell php-readline php-snmp php-soap php-sqlite3 php-sybase php-tidy php-xml php-xmlrpc php-zip php-mysqli php-pear php-phpseclib php-imagick

sudo service php7.3-fpm start
```

### Sury

```bash
#!/bin/bash
# To add this repository please do:

if [ "$(whoami)" != "root" ]; then
    SUDO=sudo
fi

${SUDO} apt-get -y install apt-transport-https lsb-release ca-certificates curl
${SUDO} curl -sSL -o /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
${SUDO} sh -c 'echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
${SUDO} apt-get update
```
```
sudo apt install -y php8.0-{fpm,dev,bcmath,bz2,cgi,cli,common,curl,enchant,gd,gmp,imap,interbase,intl,ldap,mbstring,mysql,odbc,pgsql,phpdbg,pspell,readline,snmp,soap,sqlite3,sybase,tidy,xml,zip,mysqli,gettext,imagick}
sudo apt install -y libphp8.0-embed

sudo service php8.0-fpm start
```

## Composer

```bash
#https://getcomposer.org/download/

php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

## symfony

```bash
# https://symfony.com/download
wget https://get.symfony.com/cli/installer -O - | bash
sudo mv /home/niko/.symfony/bin/symfony /usr/local/bin/symfony
```

## Debian only installs version 2.6, and for phpMyAdmin you need a version >=2.9 (https://wiki.debian.org/Backports)

```bash
sudo apt -t buster-backports install php-twig
```

## Python

```Bash
sudo apt-get install -y python3 python3-pip python-dev libpcre3 libpcre3-dev build-essential libssl-dev libffi-dev python3-venv
```

## .bashr

```bash

# Python 3.7
echo '' >> $HOME/.bashrc
echo '# Set Python default' >> $HOME/.bashrc
echo 'alias python="/usr/bin/python3.7"' >> $HOME/.bashrc
echo '' >> $HOME/.bashrc

# $USER GIT option "print branch" 
echo "# Print Git Branch in terminal
parse_git_branch() {
	     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}" >> $HOME/.bashrc
echo 'export PS1="${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[33m\]\$(parse_git_branch)\[\033[00m\] $ "' >> $HOME/.bashrc
```

## Debug PHP https://xdebug.org/docs/install

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
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
```

## Mysql

```bash
sudo apt install -y mariadb-common mariadb-server mariadb-client mariadb-backup
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
    MariaDB [mysql]> CREATE USER 'admin'@'localhost' IDENTIFIED BY 'admin';
    # GRANT ALL PRIVILEGES
    MariaDB [mysql]> GRANT ALL PRIVILEGES on *.* TO 'admin'@'localhost';
    MariaDB [mysql]> select user,host from mysql.user;
    # OUTPOUT
    +-------+-----------+
    | user  | host      |
    +-------+-----------+
    | admin | localhost |
    | root  | localhost |
    +-------+-----------+
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

## PosgreSQL

```bash
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
sudo apt-get -y install postgresql postgresql-doc postgresql-contrib

sudo service postgresql start
sudo -u postgres -i
psql
    #in Postgres 
    postgres=# CREATE USER admin WITH PASSWORD 'admin';
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
     admin     |                                                            | {}
     postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
     # Make user 'admin'  as superUser
     postgres=# ALTER ROLE admin WITH SUPERUSER;
    postgres=# \du
                                       List of roles
     Role name |                         Attributes                         | Member of
    -----------+------------------------------------------------------------+-----------
     admin     | Superuser                                                  | {}
     postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

## Apache2

```bash
sudo apt install apache2 libapache2-mod-php7.3 libgmp3-dev libpq-dev libapache2-mod-wsgi-py3
sudo a2enmod alias proxy proxy_fcgi
sudo service apache2 start
    # Check version PHP
    sudo vi /var/www/html/info.php
    # write the lige
    <?php phpinfo() ?>
    # Go to http://localhost/info.php to see php features
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
        <FilesMatch \.php$>
        # 2.4.10+ can proxy to unix socket
        SetHandler "proxy:unix:/run/php/php7.3-fpm.sock|fcgi://localhost"
        </FilesMatch>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
sudo a2ensite 001-default
sudo service apache2 restart
```

## Nginx

```bash
sudo apt install nginx-full
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
```

## PhpMyAdmin

### Stable 4.9

```bash
############################
# phpMyAdmin with Nginx
# Config must be empty like this:
# [] apache2
# [] lighttpd
# dbconfig-common <Yes>
# password : Mypassword =>
#  The MySQL application password is only used internally by phpMyAdmin 
#  to communicate with MySQL. You can leave this blank and a password will 
#  be generated automatically. Just press ENTER to continue.
sudo apt update && sudo apt install phpmyadmin
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```
### PhpMyAdmin Download

```bash
# https://www.phpmyadmin.net/downloads/
# https://docs.phpmyadmin.net/en/latest/setup.html#verifying-phpmyadmin-releases

cd ~/Downloads
wget https://files.phpmyadmin.net/phpMyAdmin/5.0.4/phpMyAdmin-5.0.4-english.tar.gz
wget https://files.phpmyadmin.net/phpMyAdmin/5.0.4/phpMyAdmin-5.0.4-english.tar.gz.asc
wget https://files.phpmyadmin.net/phpMyAdmin/5.0.4/phpMyAdmin-5.0.4-english.tar.gz.sha256

# key
wget https://files.phpmyadmin.net/phpmyadmin.keyring
gpg --import phpmyadmin.keyring

# Check Verification
gpg --verify phpMyAdmin-xxxxxx-all-languages.zip.asc
># //OUTPUT
> gpg: Signature made Fri 29 Jan 2016 08:59:37 AM EST using RSA key ID 8259BD92
> gpg: Good signature from "Isaac Bennetch <bennetch@gmail.com>"
> gpg:                 aka "Isaac Bennetch <isaac@bennetch.org>"
> gpg: WARNING: This key is not certified with a trusted signature!
> gpg:          There is no indication that the signature belongs to the owner.
> Primary key fingerprint: 3D06 A59E CE73 0EB7 1B51  1C17 CE75 2F17 8259 BD92

sudo mkdir /usr/share/phpmyadmin
sudo tar xzf phpMyAdmin-xxxxxx-english.tar.gz --strip-components=1 -C /usr/share/phpmyadmin

sudo cp /usr/share/phpmyadmin/config.sample.inc.php /usr/share/phpmyadmin/config.inc.php
$cfg['blowfish_secret'] = 'yourbolfish-whit-32chars'

sudo chown -R www-data:www-data /usr/share/phpmyadmin

sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```

## PGadmin

```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y pgadmin4  pgadmin4-apache2

cd /etc/postgresql/11/main/
# uncomment
listen_addresses = 'localhost'
sudo /etc/init.d/postgresql restart
```


## nodeJS
https://github.com/nodesource/distributions/blob/master/README.md


## ssh-add
```bash
eval `ssh-agent -s`
ssh-add

sudo vi /etc/ssh/ssh_config
AddkeysToAgent yes
```

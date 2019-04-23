# Linux Server Configuration

## Project Overview

Take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

# Linux Server

In this project, I use Amazon Lightsail to host an instance of a Ubuntu Linux server in the cloud:
```
IP: 3.17.184.40 (IP address no longer works)
SSH Port: 2200
```

## How to Access Website
* For server
    ```
    http://3.17.184.40 (IP address no longer works)
    ```
* For server with google auth (facebook won't work on http)
    ```
    http://3.17.184.40.xip.io (IP address no longer works)
    ```

## How to Access Linux Server
* With given key and password (IP address no longer works)
    ```
    ssh grader@3.17.184.40 -p 2200 -i GraderPrivateKey
    ```

## Configuration

### Check update
1. Update package list
    ```
	sudo apt-get update
    ```
2. Perform an upgrade
    ```
    sudo apt-get upgrade
	```
3. Remove old packages
    ```
    sudo apt-get autoremove
    ```

### Create new user and give ```sudo```
* Create the new user
    1. Install finger
        ```
        sudo apt-get install finger
        ```
    2. Add new user "grader"
        ```
        sudo adduser grader
        ```
    3. check user info
        ```
        cat /etc/passwd
        ```

* Give ```sudo``` access to the new user (e.g. grader)
    1. Open sudoers options
        ```
        sudo cat /etc/sudoers
        ```
    2. Get list of sudoers
        ```
        sudo ls /etc/sudoers.d
        ```
    3. Create user profile for the new user (e.g. grader)
        ```
        sudo touch /etc/sudoers.d/grader
        ```
    4. Edit profile
        ```
        sudo nano /etc/sudoers.d/grader
        ```
    5. Add new line
        ```
        grader ALL=(ALL) NOPASSWD:ALL
        ```

### SSH
#### Generate key pairs
* build key pairs in local (in CMD or git bash)
	```
    ssh-keygen
    ```
* name key pairs (e.g. GraderPrivateKey)
* enter passphrase

#### Publish key pairs
1. Login linux us a user (e.g. grader)
2. Create a folder and a file by
    ```
    mkdir .ssh
    touch .ssh/authorized_keys
    ```
4. Go to local key folder
    ```
    cat YourKeyName.pub
    ```
4. copy all
5. back to linux
    ```
    nano .ssh/authorized_keys
    ```
6. paste all
7. save
7. change folder permission
    ```
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
    ```

#### Login with key pairs (IP address no longer works)
1. ssh grader@3.17.184.40 -p 2200 -i GraderPrivateKey
2. enter passphrase

#### Disable password-based login & root login, and change port to 2200
1. Type
    ```
    sudo nano /etc/ssh/sshd_config
    ```
2. scroll down and find
    ```
    PasswordAuthentication Yes
    ```
3. change Yes to No
4. Find
    ```
    PermitRootLogin
    ```
5. Change to
    ```
    PermitRootLogin no
    ```
6. Find
    ```
    #Port 22
    ```
7. Uncomment and change to 2200
8. restart service
    ```
    sudo service ssh restart
    ```

### Change timezone to UTC
```
sudo timedatectl set-timezone UTC
```

### Configuration of Firewall
1. Block all incoming connections on all ports:
    ```
    sudo ufw default deny incoming
    ```
2. Deny incoming connections for SSH on port 22:
    ```
    sudo ufw deny 22
    ```
3. Allow outgoing connection on all ports:
    ```
    sudo ufw default allow outgoing
    ```
4. Allow incoming connection for SSH on port 2200:
    ```
    sudo ufw allow 2200/tcp
    ```
5. Allow incoming connections for HTTP on port 80:
    ```
    sudo ufw allow www
    ```
6. Allow incoming connection for NTP on port 123:
    ```
    sudo ufw allow ntp
    ```
7. To check the rules that have been added before enabling the firewall use:
    ```
    sudo ufw show added
    ```
8. To enable the firewall, use:
    ```
    sudo ufw enable
    ```
9. To check the status of the firewall, use:
    ```
    sudo ufw status
    ```

### Install required packages

1. Install Apache:
	```
    sudo apt-get install apache2
    ```
    1. Install the libapache2-mod-wsgi package and setting:
        ```
        sudo apt-get install libapache2-mod-wsgi
        sudo apt-get install libapache2-mod-wsgi-py3 (if python3, this case used 2)
        ```
    2. Enable the mod_wsgi using the command:
        ```
        sudo a2enmod wsgi
        ```
    3. Install some libraries of python development:
        ```
        sudo apt-get install libpq-dev python-dev	
        ```
2. Install postgreSQL and setting:
	1. Install
        ```
        sudo apt-get install postgresql postgresql-contrib
        ```
    2. Package settings
        1. Only allow localhost
            ```
            sudo nano /etc/postgresql/10/main/postgresql.conf
            ```
        2. Add
            ```
            enable listen_addresses = 'localhost'
            ```
        3. Create a PostgreSQL user called catalog with:
            ```
            sudo -u postgres createuser -P catalog
            ```
        4. Create an empty database called catalog with:
            ```
            sudo -u postgres createdb -O catalog catalog
            ```

3. Install packages: Flask and SQLAlchemy
    ```
    sudo apt-get install python-psycopg2 python-flask
    sudo apt-get install python-sqlalchemy python-pip
    sudo pip install --upgrade pip (not necessary)
    sudo pip install oauth2client
    sudo pip install requests
    sudo pip install httplib2 (not necessary, already installed)
    ```

4. Install git and clone project
    ```
    sudo apt-get install git
    ```
    1. Navigate to
        ```
        /var/www/FlaskApp/
        ```
    2. Clone to FlaskApp folder
        ```
        sudo git clone projectURL FlaskApp
        ```

5. Install Virtual Environment

    * From /var/www/FlaskApp/FlaskApp directory install pip:
        ```
        sudo apt-get install python3-pip
        ```

    * Install the virtual environment:
        ```
        sudo apt-get install python-virtualenv
        ```
    * Create the virtual environment:
        ```
        sudo virtualenv -p python3 venv3
        ```
    * Change the ownership to grader with:
        ```
        sudo chown -R grader:grader venv3/
        ```
        
### Create and update catalog.wsgi file for the installation
* From /var/www/FlaskApp/
    ```
    touch catalog.wsgi
    ```
* Add
    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/FlaskApp/")
    
    from FlaskApp import app as application
    ```
* Save

### Configure and Enabling New Virtual Host
1. Create and edit virtual host
    ```
    sudo nano /etc/apache2/sites-available/catalog.conf
    ```
2. Add (IP address no longer works)
    ```
    <VirtualHost *:80>
            ServerName 3.17.184.40
            ServerAdmin admin@3.17.184.40
            WSGIDaemonProcess catalog user=www-data group=www-data threads=5
            WSGIProcessGroup catalog
            WSGIApplicationGroup %{GLOBAL}
            WSGIScriptAlias / /var/www/FlaskApp/catalog.wsgi
            <Directory /var/www/FlaskApp/FlaskApp/>
                    Require all granted
            </Directory>
            Alias /static /var/www/FlaskApp/FlaskApp/static
            <Directory /var/www/FlaskApp/FlaskApp/static/>
                    Require all granted
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
3. Disable the default host by using the command:
    ```
    sudo a2dissite 000-default.conf
    ```
4. Enable the virtual host by using the command:
    ```
    sudo a2ensite catalog.conf
    ```
5. Type the following command for restarting the apache:
    ```
    service apache2 reload
    service apache2 restart
    ```

## References
1. [sites-enabled and sites-avaiable](https://www.linode.com/community/questions/6592/sites-enabled-and-sites-available-confusion)
2. [Flask ImportError: No Module Named Flask](https://stackoverflow.com/questions/31252791/flask-importerror-no-module-named-flask)
3. [wsgi](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
4. [How To Deploy a Flask Application on an Ubuntu VPS ](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
5. [Python3 + venv + wsgi implementation](https://github.com/jungleBadger/-nanodegree-linux-server-troubleshoot/tree/master/python3%2Bvenv%2Bwsgi)
6. [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

# Linux-Server-Configuration
Linux distribution on a virtual machine, prepared to host Catalog web application

Author: Tyler Villanueva
Rev 1 Date: Jan 20, 2017

* [Project Access](#project-access)
* [Project Overview](#project-overview)
* [Installed Packages](#installed-packages)
* [Configuration Summary](#configuration-summary)
* [Project Step by Step Walkthrough](#project-step-by-step-walkthrough)

## Project Access
#### IP address

**`35.160.177.48`**

#### SSH Port

**`2200`**

#### Web Application URL

http://35.160.177.48/  

## Project Overview 

I configured a remote virtual machine to host a database server with PostgreSQL and web application which are built in Python. This was done using a provided Linux distribution by [Udacity](https://www.udacity.com), and installed Apache2 server to run Python Flask applications that connect to the PostgreSQL database.

## Installed Packages

Package Name | Description
--------------: | :------------
**finger:** | Displays an easy to read information about a user
**apache2** | HTTP Server
**libapache2-mod-wsgi** | hosts Python applications on Apache2 server
**ntp** | Synchronizes time over a network
**postgresql** | Postgresql Database server
**git** | Version control system tools
**python-setuptools** | An easy-install package to facilitate installing Python packages
**sqlalchemy** | ORM and SQL tools for Python
**flask** | Microframework for web applications
**python-psycopg2** | PostgreSQL adapter for Python
**oauth2** | Authorization framework for third-party login (Google and Facebook)
**google-api-python-client** | Google API for OAuth login
**fail2ban** | Protection against suspicious site activity by IP banning

## Configuration Summary

- Setup Virtual Machine and SSH into the server. (Instructions provided by Udacity)
- A new system user `grader` was created with permission to sudo.
- All cuurently installed packages were updated and upgraded.
- Changed SSH Port from `22` to `2200` and configure SSH access.
- Configured `UFW`to only allow incoming connections for `SSH(Port:2200)`, `HTTP(Port:80)` and `NTP(Port:123)`.
- Configured local Time Zone to `UTC`.
- Installed and configure `Apache` to serve a `Python mod_wsgi` application.
- Installed Git and Setup Environment for delopying Flask Application.
- Install and configure `PostgreSQL` with default settings to *not* allow remote connection.
- Created a new user `catalog`, added user to PostgreSQL databse with limited permissions to catalog application database.
- Get OAUTH-LOGINS (Google+) working.
- Installed and Configured `Fail2ban` intrusion protection that bans suspicious IPs.

----------


## Project Step By Step Walkthrough:


----------


### A - Development Environment: Setup Virtual Machine and SSH into the server
Reference: [Udacity](https://www.udacity.com/account#!/development_environment)

1. Hit "Create new development environment".

2. Take note of your `Public IP address`, and download `private key` at the bottom.

3. Move the private key file into `~/.ssh` folder with this command in Terminal:
    
    ```
    $ mv ~/Downloads/udacity_key.rsa ~/.ssh
    ```
    
    where ~ is your environment's home directory). If you downloaded the file to the downloads folder, just execute the above command in your terminal.

4. Allow owner the right to "read" and "write" the file:

    ```
    $ chmod 600 ~/.ssh/udacity_key.rsa
    ```
    
5. SSH into the instance:

    ```        
    $ ssh -i ~/.ssh/udacity_key.rsa  root@YOUR-PUBLIC-IP-ADDRESS 
    ```


----------


### B - User Management: Create a new user with the permission to sudo.
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

*Note:* In some cases, you would have to use the `sudo` commnand to be able to perform some functions depending on the administrative priviledge of your local computer. 

1. Create a new user: `grader`:

    ``` 
    root@ip-10-20-30-101:~# adduser grader
    ```
    
    OPTIONAL: You can confirm that the new user has been created by installing `finger`.
    
    ``` 
    root@ip-10-20-30-10:~# apt-get install finger
    root@ip-10-20-30-10:~# finger grader
   ```
2. Give the new user sudo permission:

    (a) - Open sudoer configuration using visudo command:
    
    ```
        root@ip-10-20-30-10:~# visudo
    ```
    A nano editor will open up. scroll down to the line that reads:

	```
		root ALL=(ALL:ALL) ALL
   ```
    (b) -  Below that line, add the new user: `grader` like so:
    
    ```
        root ALL=(ALL:ALL) ALL
        grader ALL=(ALL:ALL) ALL
    ```
    Save changes by pressing `ctrl+x, y` then Enter key.  
    
    (c.) -  You can list all the users present root.
    
    ```
        root@ip-10-20-30-10:~# cut -d: -f1 /etc/passwd
    ```


----------


### C - Update and upgrade currently installed package

1. Update all available packages: This will provide a list of packages to be upgraded.

    ```
     root@ip-10-20-30-10:~# sudo apt-get update
    ```
    
2. Upgrade packages to newrer versions:
    
    ``` 
    root@ip-10-20-30-10:~# sudo apt-get upgrade
    ```
    
3. Add CRON script to manage `update` and `upgrade` installed packages.

    (a) -  Install unattended-upgrade packages:
    
    ```
        root@ip-10-20-30-10:~# sudo apt-get install unattended-upgrades
    ```
    (b) -  Enable the unattended-upgrade packages:
  
    ```
       root@ip-10-20-30-10:~# sudo dpkg-reconfigure -plow unattended-upgrades
    ```


----------



### D - Change SSH Port from `22` to `2200` and Configure SSH access:
Reference :[Ask Ubuntu](http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server)

1. Change the SSH configuration file:

    (a) -  Access the config file using nano editor:
    
    ```
        root@ip-10-20-30-10:~# nano /etc/ssh/sshd_config
    ```
    while in the nano editor:
  
    (b) - Change `Port 22` to `Port 2200`.

    (c) - Change `PermitRootLogin without-password` to `PermitRootLogin no`

    (d) - Change `PasswordAuthentication no` to `PasswordAuthentication yes`. This is temporal.

    (e) - At the end of the file, add `UseDNS no` and `AllowUsers grader`. This will allow grader SSH login access.

    (f) - Exit nano editor: `ctrl+x, y then enter`, and Restart SSH service for changes.
    
    ``` 
    root@ip-10-20-30-10:~# sudo service ssh restart
    ```
    Now you can connect to `grader` from local Machine using : 

    ```
    YOUR LOCAL MACHINE:~$ ssh grader@35.166.177.48 -p 2200
    ```
2. Create a SSH Key Pair:

    (a) - Switch to your local machine by using the `exit` command: 
    
    ```
    root@ip-10-20-30-10:~# exit
    ```
    
    Now enter this command to generate a SSH key pair.
   
   ```
    YOUR LOCAL MACHINE:~$ ssh-keygen
   ```
   
   enter a `passphrase` which will prevent unauthroized access to the files.
 
   Now you will see two files. Assuming you used the default `id_rsa`, you will see that ssh-keygen has generated `id_rsa` (private_key) and `id_rsa.pub` (public_key) file. The file `id_rsa.pub` will be placed on the server.

    (b) - Switch to the remote server as `grader` and create a directory called `.ssh`
    
    ```
    grader@ip-10-20-30-101:~$ mkdir .ssh
    ```
   
   Create a new file within the `.ssh` directory called `authorized_keys`. A special file that will store the public keys.
   
    ```
    grader@ip-10-20-30-101:~$ touch .ssh/authorized_keys
    ```
    
    (c.) - Switch back to your Local Machine, and copy the contents of `id_rsa.pub`:
    
    ```
    YOUR LOCAL MACHINE:~$ sudo cat ~/.ssh/ida_rsa.pub
    ```
    
    (d) - Switch back to your Remote Server, edit authorized_keys file and paste the content of id_rsa.pub inside. Save file.
    
    ```
    grader@ip-10-20-30-101:~$ sudo nano .ssh/authorized_keys
    ```
    
    (e) - Set specific file permission on `SSH` and `authorized_keys` directories:
    
    ```
    grader@ip-10-20-30-101:~$ chmod 700 .ssh
    grader@ip-10-20-30-101:~$ chmod 644 .ssh/authorized_keys
    ```
    (f) - SSHD Configuration:
    
    ```
    grader@ip-10-20-30-101:~$ sudo nano /etc/ssh/sshd_config
    
    Change the `PasswordAuthentication yes` to `PasswordAuthentication no`
    ```
    
    To remove the `sudo: unable to resolve host...` warning after using sudo, follow these steps:
    
    - Open `sudo nano /etc/hostname`. 
    - You will see something like this `ip-10-20-25-101`. Copy that 
    - open `sudo nano /etc/hosts` and on the first line, append the hostname right before 127.0.0.1 localhost like so:
    
	    ```
	    52.39.26.86 ip-10-20-25-101
		127.0.0.1 localhost
	    ```


----------


### E - Configure UFW to only allow incoming connections for SSH(Port:2200), HTTP(Port:80) and NTP(Port:123).

1. Check the status of UFW. Make sure it is `inactive`:
    
    ```        
    grader@ip-10-20-30-101:~$ sudo ufw status
    ```
    
2. Deny all incoming connections as default so that we can allow the ones we need.

    ```
    grader@ip-10-20-30-101:~$ sudo ufw default deny incoming
    ```
    
3. Allow incoming TCP connection on SSH(Port:2200), HTTP(Port:80), NTP(Port:123)

    ```
    grader@ip-10-20-30-101:~$ sudo ufw allow 2200/tcp
    grader@ip-10-20-30-101:~$ sudo ufw allow 80/tcp
    grader@ip-10-20-30-101:~$ sudo ufw allow 123/udp
    ```    
4. Enable the firewall: 

    ```
    grader@ip-10-20-30-101:~$ sudo ufw enable
    ```


----------


### F - Configure local Time Zone to UTC
Reference: [Ubuntu](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

1. Open Timezone selection dialog:

    ```
    grader@ip-10-20-30-101:~$ sudo dpkg-reconfigure tzdata
    ```

2. Choose and type `None of the above`, then choose `UTC`.

3. Setup `ntp daemon` to improve time sync:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install ntp
    ```


----------


### G - Install and Configure Apache to serve a Python mod_wsgi application.

1. Install Apache web Server:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install apache2
    ```
    
2. In your browser, type in your public ip  address: `http://35.160.177.48`, and it should return - `It works!` Ubuntu page.
        
3. Install `mod_wsgi`, and `python-setuptools` helper package. This will serve Python apps from Apache:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install python-setuptools libapache2-mod-wsgi
    ```
    
4. Configure Apache to handle requests using the `WSGI` module

    ```
    grader@ip-10-20-30-101:~$ sudo nano cat /etc/apache2/sites-enabled/000-default.conf
    ```
    
    Add the following line: `WSGIScriptAlias / /var/www/html/myapp.wsgi` at the end of the `<VirtualHost *:80> block, right before the closing </VirtualHost>`. Now save and quit the nano editor.
        
    Restart Apache: `sudo apache2ctl restart`

5. To test (step can be skipped) create the `myapp.wsgi` file that was added to the deafult-conf file:

    ```
    grader@ip-10-20-30-101:~$ sudo nano /var/www/html/myapp.wsgi
    ```
    Copy and paste the following:
    
    ```python
    def application(environ, start_response):
        status = '200 ok'
        output = 'Hello World - Its Working'
        
        response_headers=[('content-type','text/plain'),('content-length', str(len(output)))]
        start_response(status, response_headers)
        return [output]
    ```
    
    After you save the file, refresh/reload your browser and you should see `Hello World - Its working`. 
    
6. Restart Apache server to load mod_wsgi.

    ```
    grader@ip-10-20-30-101:~$ sudo service apache2 restart
    ```
    
7. To remove the message: `Could not reliably determine the server's fully qualified domain name...`:

    (a) -  Create an Apache config file with the domain name:
        
    ```    
    grader@ip-10-20-30-101:~$ echo "ServerName 35.160.177.48" | sudo tee /etc/apache2/conf-available/fqdn.conf
    ```
    
    (b) -  Enable the file:
    
    ```
    grader@ip-10-20-30-101:~$ sudo a2enconf fqdn
    ```


----------

### H - Install Git and Setup Environment for delopying Flask Application. 
Reference: [Github](https://github.com/elnobun/Item-Catalog-Movie-Collection-App-/tree/master/vagrant)

1. Install Git:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install git
    ```
    
2. Setup process for delopying Flask application:
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

    (a) - Add additional Python package to enable Apache serve Flask applications:
    
    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install libapache2-mod-wsgi python-dev
    ```
    
    (b) - Enable `mod_wsgi` if it is not enabled already:
    
    ```
    grader@ip-10-20-30-101:~$ sudo a2enmod wsgi
    ```
    
    (c.) - Navigate to the `www` directory:
    
     ```
    grader@ip-10-20-30-101:~$ cd /var/www
    ```
    
    - Setup a directory folder. You can call it `Catalog`: This will hold our app,
    
        ```
        grader@ip-10-20-30-101:/var/www$ sudo mkdir Catalog
        ```
    - cd `Catalog` and make another directory called `catalog`.
    
        ```
        grader@ip-10-20-30-101:/var/www$ cd Catalog
        grader@ip-10-20-30-101:/var/www/Catalog$ sudo mkdir catalog
        ```
    
    - cd `catalog` and make a directory called `static templates` 
    
        ```        
        grader@ip-10-20-30-101:/var/www/Catalog$ cd catalog
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo mkdir static templates
        ```
        
    - Inside `catalog` folder, create a flask applicaion logic file called `__init__.py` through the nano editor: *Note* `__init__.py` is written with double underscore like so: `_\_\init\_\_.py`.
    
        ```
        grader@ip-10-20-25-175:/var/www/Catalog/catalog$ sudo nano __init__.py
        ```
        
    - Inside the __init__.py  nano editor, paste this code:
    
        ```python
        from flask import Flask
        app = Flask(__name__)
        @app.route("/")
        def hello():
            return "Hello, Catalog app coming up soon!"
        if __name__ == "__main__":
          app.run()
        ```
        
       You can use `ls` or `ls -al` to view the content of your file path. 
       
       Next we  will test our Python Flask file:
        
3. Flask Installation and Virtual Environment configuration:
    
    (a) -  Install `pip` (good practice)
    
    ```            
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install python-pip
    ```
    
    (b) -  Install virtual environment (virtualenv):
    
    ```
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install virtualenv
    ```
    
    You can set the virtual environment name to a shorter name. e.g `venv`
    
    ```    
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo virtualenv venv
    ```
    
4.  Configure and Enable a New Virtual Host that will house our `.wsgi` file we are to create, just like we did while testing our `myapp.wsgi` file.

    (a) - Create vitual host config file:
    
    ```        
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo nano /etc/apache2/sites-available/catalog.conf
    ```
    - In the newly created `catalog.conf` file, paste in the following lines of code.
        ```
        <VirtualHost *:80>
            ServerName 35.160.177.48
            ServerAdmin admin@35.160.177.48
            WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
            <Directory /var/www/Catalog/catalog/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/Catalog/catalog/static
            <Directory /var/www/Catalog/catalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
        ```
     - Enable the Virtual Host.
        
        ```
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo a2ensite catalog.wsgi         
        ```
    * Note: You may need to disable the default site. Do this using:
    
        ```
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo a2dissite 000-default.conf         
        ```
	
    (b) - Create the `catalog.wsgi` file that was defined in the host.
    
    - Go back to the `Catalog` folder:
    
        ```
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ cd /var/www/Catalog
        ```
        
    - Create the `catalog.wsgi` file, using the nano editor:
    
        ```
        grader@ip-10-20-30-101:~/var/www/Catalog$ sudo nano catalog.wsgi
        ```
        
    - Paste the following code inside the `catalog.wsgi` file
    
        ```python
        #!/user/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/Catalog/catalog/")
        
        from catalog import app as application
        application.secret_key = 'Add your secret key'
        ```
        
    - Restart Apache:
        
        ```
        grader@ip-10-20-30-101:/var/www/Catalog$ sudo service apache2 restart 
        ```
        
5. Clone Your (Project 3 - Item Catalog) respository

    ```
    grader@ip-10-20-30-101:/var/www/Catalog$ git clone https://github.com/trvillan/CatalogApp.git
    ```

6. Move all the contents of your cloned respository directory into `/var/www/Catalog/catalog`, and delete empty directory.

    ```
    grader@ip-10-20-30-101:/var/www/Catalog$ mv CatalogApp/* /var/Catalog/catalog
    ```
    
7. Render your respository inaccessible:

    (a) - Create a `.htaccess file`:
    
    ```
    grader@ip-10-20-30-101:/var/www/Catalog$ sudo nano .htaccess
    ```
    
    (b) - Add this to the opened nano .htaccess file : `RedirectMatch 404 /\.git`
    
8.  Install all the neeeded packages and modules for python.

    (a) - FIrst activate your virtual environment:
    
    ```
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ source venv/bin/activate
    ```
    
    (b) - Install all these packages:
    
    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install python-setuptools
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install Flask
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ pip install httplib2
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ pip install requests
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install flask-seasurf
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install python-psycopg2
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install oauth2client
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install sqlalchemy
    ```
    Restart apache: `sudo apache2ctl restart`.
     *Note*:  If installs fail try without sudo.

----------

### I - Install and configure PostgreSQL with default settings to not allow remote Connection:
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

1. Install the PostgreSQL database:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install postgresql postgresql-contrib
    ```

2.  Ensure that no remote connections are allowed. It should be default.

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf
    ```
    
3.  Open your project 3 `database_setup.py` file:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo nano database_setup.py
    ```
    
4.  Effect these changes:

    (a) - Go to the line that has this syntax:
    
    ```python
    engine = create_engine('sqlite:///YOUR-DATABASE-NAME.db')
    ```
    
    (b) - Change the above syntax to a Postgresql database engine like so.
    
    ```python
    engine = create_engine('postgresql://catalog:DB-PASSWORD@localhost/catalog')
    ```
    you should put down a password where you have DB-PASSWORD. Make sure you rememebr the password because you will need it later.
    
    Also, effect the above changes in your main `catalog.py` file.
    
    (c.)  - Rename your `catalog.py', to `__init__.py`
    
    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ mv YOUR_APP.py __init__.py
    ```
    
    *Note*:  You also need to change your 'catalog.wsgi` file to reflects this change. It should read like so:
    
    ```python
    #!/user/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/Catalog/catalog/")

    from __init__ import app as application
    application.secret_key = 'Add your secret key'
    ```

----------


### J - Create a new user: `catalog`, add user to PostgreSQL databse with limited permissions to catalog application database.

1.  Create a user `catalog` for psql:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo adduser catalog
    
    choose a password for that user.
    ```
    
2.  Change to the default user `Postgres`

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo su - postgres
    postgres@ip-10-20-25-175:~$ 
    ```
    
    Connect to the postgres system:
    
    ```
    postgres@ip-10-20-30-101:~$ psql
    ```
    
    You see this:
    
    ```
    psql (9.3.12)
    Type "help" for help.

    postgres=# 
    ```
    
3.  Add the postgres user: `catalog` and setup users parameters.

    (a) - Create user `catalog` with a `login role` and `password`
    
    ```
    postgres=# CREATE USER catalog WITH PASSWORD 'DB-PASSWORD';
    ```
    *Note* : The *DB-PASSWORD*, should be the same password you used to create the postgresql engine in your database_setup.py file.
    
    (b) - Allow the user `catalog` to be able to create databse tables
    
    ```
    postgres=# ALTER USER catalog CREATEDB;
    ```
    You can list the roles available in postgres, and their attribute:
    
    ```
    postgres=# \du
    ```
    
4.  Create a new database called `catalog` for the user: `catalog`:

    ```
    postgres=# CREATE DATABASE catalog WITH OWNER catalog;
    ```
    
5.  Connect to the database:

    ```
    postgres=# \c catalog
    ```
    
6. Revoke all rights on the database schema, and grant access to catalog only.

    ```
    catalog=# REVOKE ALL ON SCHEMA public FROM public;
    catalog=# GRANT ALL ON SCHEMA public TO catalog;
    ```
    
    Exit Postgresql and postgres user:
    
    ```
    postgres=# \q
    postgres@ip-10-20-30-101~$ exit
    ```
    
7. Create Postgresql database schema:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ python database_setup.py
    ```
    
    We can check that it worked. After you run `database_setup.py`, Go back to your postgres schema, and connect to `catalog` database.
    
    ```
    postgres@ip-10-20-30-101:~$ psql
    psql (9.3.12)
    Type "help" for help.

    postgres=# \c catalog
    ```
    
    When you conect to the `catalog` database, you can view all the relations created by your `python database_setup.py` command.
    
    ```
    catalog=# \dt
    
    ```
    
    Now exit postgres, and return to Virtual environment.
    
8.  Restart Apache:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo service apache2 restart
    ```
    
    In your browser, put in your PUBLIC-IP-ADDRESS : `35.160.177.48`. If you follwed the steps accordingly, Your applciation should come up.
    
    If errors occur you can check the last 30 lines of the error log with: 
    
    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo tail -30 /var/log/apache2/error.log
    ```


----------


### K - Get OAUTH-LOGINS (Google+ and Facebook) working.

1.  To fix the `client_secrets.json` error, go to your '__init__.py' or 'catalog.py' file and change the following (should be near the top and in the login section:

    ```python
    app_token = json.loads(
    open('client_secret.json', 'r').read())['web']['client_id']

	oauth_flow:flow_from_clientsecrets('client_secret.json', scope='')
    ```
    
    Add `/var/www/Catalog/catalog` to your code path.:
    
    ```python
    app_token = json.loads(
    open('/var/www/Catalog/catalog/client_secret.json', 'r').read())['web']['client_id']

	
	oauth_flow:flow_from_clientsecrets('/var/www/Catalog/catalog/client_secret.json', scope='')
    ```
    
2. Enable virtual host - catalog.conf

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo a2ensite catalog
    ```

3. To get Google+ authorization working, you must add your URL to google authorization just like what was done during the catalog project. (You will be addind to the catalog app):

    (a) - On the Developer Console: http://console.developers.google.com, select your Project.
    
    (b) - Navigate to `Credentials`, and edit your `OAuth 2.0 client ID':
    


----------


### L - Install and Configured Fail2ban intrusion protection that bans suspicious IPs.
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04)
    
1.  Install Fail2ban application:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install fail2ban
    ```
    
2.  Copy the default config file

    ```
    grader@ip-10-20-30-101:~$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    ```
    
3. Open `jail.local` and change the followinf default parameters:

    ```
    grader@ip-10-20-30-101:~$ sudo nano /etc/fail2ban/jail.local
    ```
    
4.  Set the following parameters:

    ```
    set bantime = 1600
    destemail = YOURNAME@DOMAIN or YOUR-EMAIL
    action = %(action_mwl)s
    under [ssh] change port = 2200
    ```

5. Stop the service:

    ```
    grader@ip-10-20-30-101:~$ sudo service fail2ban stop
    ```
    
6. Start the service again:

    ```
    grader@ip-10-20-30-101:~$ sudo service fail2ban start
    ```
    
## FINALLY:
Restart apache2 server, run your app by visiting your URL. 

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

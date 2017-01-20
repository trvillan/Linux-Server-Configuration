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

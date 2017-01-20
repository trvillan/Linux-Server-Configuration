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

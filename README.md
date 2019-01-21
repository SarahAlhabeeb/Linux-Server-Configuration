# Linux Server Configuration
#### Full Stack Web Developer Nano-Degree

### Overview
> This project gives a deep understanding of exactly what web applications are doing, how they are hosted, 
  and how the interactions between multiple systems are done. The developer will learn how to access, secure, and
  perform the initial configuration of a bare-bones Linux server. The developer will then learn how to install and
  configure a web and database server and actually host a web application.

### Item Catalog App
  * IP address: 18.194.129.62
  * SSH Port:  2200
  * URL: http://18.194.129.62.xip.io
  

### Setup
##### 1. Create  an  instance in [Amazon Lightsail](https://lightsail.aws.amazon.com/)

##### 2. Update  and upgrade
  ```
   $ sudo apt-get update 
   $ sudo apt-get upgrade 
  ```

##### 3. Change SSH Port
  ```
   $ sudo nano /etc/ssh/sshd_config
  ```
  \- Change the Port option to: `Port 2200`
  
  \- Then restart SSH:
  ```
   $ sudo service ssh restart
  ```
 
##### 4. Configure firewall

* ```sudo ufw default deny incoming```
* ```sudo ufw default allow outgoing```
* ```$ sudo ufw allow 2200/tcp.```
* ```$ sudo ufw allow 80/tcp.```
* ```$ sudo ufw allow 123/udp.```
* ```$ sudo ufw enable.```

##### 5. Create user grader
\- Create  user
  ```
   $ sudo adduser grader
  ```
\- Give it root privileges
  ```
   $ sudo usermod -aG sudo grader
  ```
\- Generate a Key Pair
*  in your local machine
  ```
   $ ssh-keygen
  ```
*  Copy the public key
* On the server:
1. ```sudo su - grader```
2. ```mkdir ~/.ssh```
3. ```$ sudo chmod 700 ~/.ssh```
4. ```$ sudo nano ~/.ssh/authorized_keys``` Paste the key and save it.
5. ```$ sudo chmod 600 ~/.ssh/authorized_keys```
6. ```$ exit```

* To login as Grader user 
```
ssh -i ~/.ssh/grader_key -p 2200 grader@18.194.129.62
```

##### 6. Change time zone
```
sudo timedatectl set-timezone UTC
```

##### 7. Install and configure Apache to serve a Python mod_wsgi application

* ``` sudo apt-get install apache2 apache2-utils libexpat1 ssl-cert python ```

* ``` sudo apt-get install libapache2-mod-wsgi ```

* ``` sudo service apache2 restart```

* To test Apache: ``` http://18.194.129.62 ```


##### 8. Install Git and clone project
1. Install Git  ``` apt-get install git-core ```

2. Confirm Git the installation ``` git --version ```

3. Configure Git’s settings (for the root user):  
* ``` git config --global user.name "testuser"```
* ``` git config --global user.email "testuser@example.com" ```

4 . ```cd /var/www/ ```

5 . ```sudo mkdir catalog ```

6 . ``` cd catalog``` 

7 .  Clone project: ```git clone https://github.com/SarahAlhabeeb/Item-Catalog.git catalog ```

##### 9. Instal Flask
* ```sudo apt-get install python-pip ```

* ```sudo pip install virtualenv```

* ``` sudo virtualenv Virtenv```
* ``` source Virtenv/bin/activate```
* ``` sudo pip install Flask```

##### 10. Creating Catalog App
1. ```sudo nano /etc/apache2/sites-available/catalogApp.conf ```

 ```
 <VirtualHost *:80>
        ServerName 18.194.129.62
        ServerAlias 18.194.129.62.xip.io
        ServerAdmin admin@18.194.129.62
        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib$
        WSGIProcessGroup catalog
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
 ```

2 . ```sudo a2ensite catalogApp ```

3 . ```sudo nano /var/www/catalog/catalog.wsgi ```
 * Add the following content:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret key'
 ```
 4  . ``` sudo service apache2 restart ```


##### 11. Install and configure PostgreSQL
1. ```sudo apt-get install postgresql postgresql-contrib ```

2. ```sudo su - postgres```

3. ``` psql```
4. ``` CREATE USER catalog WITH PASSWORD 'password';```
4. ``` ALTER USER catalog CREATEDB;```
5. ``` CREATE DATABASE catalog WITH OWNER catalog;```
6. ```\c catalog```

* ```REVOKE ALL ON SCHEMA public FROM public;```

* ```GRANT ALL ON SCHEMA public TO catalog;```

* ```\q```
* ```exit```

* Change create engine line in your \_\_init\_\_.py and database_setup.py to: 

```engine = create_engine('postgresql://catalog:password@localhost/catalog')```

Then setup database:

```python /var/www/catalog/catalog/database_setup.py ```
```python /var/www/catalog/catalog/items.py ```

Restart Apache:

```sudo service apache2 restart```
#### References

This part lists the third-party resources that helped in solving this project:
  * [Configuring SSHD on the Server](https://serversforhackers.com/c/configuring-sshd-on-the-server)
  * [Initial Server Setup with Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04?utm_content=initial-server-setup-with-ubuntu-16-04)
  * [Install and configure Apache to serve a Python mod_wsgi application](https://devops.profitbricks.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/)
  * [How To Install Git on Ubuntu 16.04 ](https://www.liquidweb.com/kb/install-git-ubuntu-16-04-lts/)
  * [Deploy a Flask Application on Ubuntu 14.04](https://devops.profitbricks.com/tutorials/deploy-a-flask-application-on-ubuntu-1404/)
  * [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

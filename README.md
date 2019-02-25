# Linux Server Configuration


## Project Overview
In this project, we need to take a baseline installation of a Linux server and prepare it to host web applications. The target is to secure a Lightail server provided by AWS from a number of attack vectors, install and configure a database server and a web server, and deploy the existing Car Sensor web application onto it.


## Why this Project?
A deep understanding of exactly what our web applications are doing, how they are hosted, and the interactions between multiple systems are what define us as a Full Stack Web Developer. Deploying our web applications to a publicly accessible server is the first step in getting users, and properly securing our applications ensures our applications remain stable and that our users' data are safe. In this project, we’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host our applications need.


## Skills Honed
Learnt how to access, secure, and perform the initial configuration of a bare-bones Linux server. Then learnt how to install and configure a web and database server and actually host a web application.


## Getting Started
### I - Launch an open-to-public virtual machine and connect to it via SSH
1. Launch an AWS Lightsail instance, selecting Linux/Unix as the platform and Ubuntu 18.04 LTS as the blueprint. After choosing the instance plan and identifying the name, wait for the server become "Running" status.

2. The server can be accessed either via a popped-up browser terminal after clicking the "Connect using SSH" button, or via local machine's terminal by providing the default private key.

3. Go to the Account page on LightSail, and download the default private key under the "SSH keys" tab. Rename the default private key to be "LightsailDefaultKey.pem", and move the private key into the folder ```~/.ssh``` (where ~ is the local machine's home directory).

4. Change permissions for the .ssh folder and the private key file on local machine, otherwise permissions are too open and the private key files are accessible by others.  
    ```$ chmod 700 ~/.ssh```  
    ```$ chmod 600 ~/.ssh/LightsailDefaultKey.pem```

5. SSH into the Lightsail server as the default user called ubuntu.  
    ```$ ssh ubuntu@13.115.180.187 -p 22 -i ~/.ssh/LightsailDefaultKey.pem```  
    The p flag stands for port, and i flag means identity. The Lightsail instance's security group provides an SSH port 22 by default.

### II - Create a new user named grader and setup SSH login for grader using key pairs
1. After logging onto the Lightsail server as ubuntu, create a new user grader with a UNIX password.  
    ```$ sudo adduser grader```

2. Edit the sshd_config file on Lightsail server, then login as grader using password is possible.  
    ```$ sudo nano /etc/ssh/sshd_config```

    Change the ```PasswordAuthentication no``` to ```PasswordAuthentication yes```, then restart the SSH service.  
    ```$ sudo service ssh restart```

3. Login as ubuntu and give grader the sudo access. By entering ```$ sudo cat /etc/sudoers```, and reading from the last line we can tell that we need to add grader into a directory called sudoers.d under /etc.  
    ```$ sudo nano /etc/sudoers.d/grader```

    In the file input ```grader ALL=(ALL:ALL) ALL```, then save and quit.

4. Now we are going to enable the key-based authentication for grader. On local machine, generate a new encrypted key pair, and name the key pair as grader. The key pair can be located in the local machine's ```~/.ssh``` folder. Then we need to put the public key on the Lightsail server for grader and put the private key in the ```~/.ssh``` folder on the local machine.  
    ```$ ssh-keygen```

    Exit from ubuntu and relogin as grader using password.  
    ```$ exit```  
    ```$ ssh grader@13.115.180.187```  
    ```$ mkdir .ssh```  
    ```$ touch .ssh/authorized_keys```  
    ```$ sudo nano .ssh/authorized_keys```

    Copy the content in the public key and paste it in this ```authorized_keys``` file, then save and quit. Next step is to change the permissions for grader's ```~/.ssh``` folder and the ```authorized_keys``` file.  
    ```$ chmod 700 .ssh```  
    ```$ chmod 644 .ssh/authorized_keys ```

    We can use ```$ ls -la``` to check whether the permission setting has been completed.

5. Finally we want to enforce the key based authentication and disable login using password.  
    ```$ sudo nano /etc/ssh/sshd_config```

    Change the ```PasswordAuthentication yes``` to ```PasswordAuthentication no```, then restart the SSH service.  
    ```$ sudo service sshd restart```

### III - Login as grader and update all currently installed packages
1. Now we can connect to the Lightsail server as grader using identity. We should put the privated key called grader (saved on the local machine) after the i flag. Amazingly, we can also SSH into the server as grader without any flags.  
    ```$ ssh grader@13.115.180.187 -p 22 -i ~/.ssh/grader``` or  
    ```$ ssh grader@13.115.180.187```

2. Fetch all the package update information, and upgrade all packages.  
    ```$ sudo apt-get update```  
    ```$ sudo apt-get upgrade```

### IV - Configure the SSH port and the Uncomplicated Firewall (UFW)
  _Warining: Do not exit from the server during this configuration stage, otherwise, it is possible to be locked out of the server permanently!_
1. The project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123). Therefore, under the "Networking" tab of the Lightsail web control panel, add custom firewall rules with port range 2200 and 123. Do not delete the SSH port 22 at this moment, otherwise, typing in the terminal will be disabled. This is the AWS's firewall no the server's, so we get double firewall now.

2. Next step is to change the SSH port configuration for the Lightsail server.  
    ```$ sudo nano /etc/ssh/sshd_config```

    Uncomment ```Port 22``` and Add ```Port 2200``` on the next line, save and quit then restart the SSH service.  
    ```$ sudo service sshd restart```

3. Finally, we can configure the UFW for our Lightsail server now. When changing the SSH port, make sure that the firewall is open for port 2200 first, so that we won't lock ourself out of the server. When we change the SSH port, the Lightsail instance will no longer be accessible through the web app "Connect using SSH" button. The button assumes the default port is being used.  
    ```$ sudo ufw status```  
    ```$ sudo ufw default deny incoming```  
    ```$ sudo ufw default allow outgoing```  
    ```$ sudo ufw allow 2200/tcp```  
    ```$ sudo ufw allow www```  
    ```$ sudo ufw allow ntp```  
    ```$ sudo ufw enable```  
    ```$ sudo ufw status```

4. After finishing the UFW setting, we can go back to the Lightsail web app control panel, and delete the SSH 20 port. Also we can go back to the ```/etc/ssh/sshd_config``` file, and delete ```Port 22```, but do not forget to run ```$ sudo service sshd restart``` after save and quit the file. Now we can exit from server and test re-login as grader, but this time we have to use the p flag with port 2200. However, if we want to log in as ubuntu, we need the identity as well.  
    ```$ exit```  
    ```$ ssh grader@13.115.180.187 -p 2200```

### V - Configure the local timezone to UTC
1. Open the time configuration and set it to UTC.  
    ```$ sudo dpkg-reconfigure tzdata```

2. We found that it is already set to UTC by default, so no need to change.

### VI - Install and configure Apache web server and mod_wsgi to serve a Python WSGI App
1. Install Apache.  
    ```$ sudo apt-get install apache2```

2. Install mode_wsgi.  
    ```$ sudo apt-get install libapache2-mod-wsgi```

3. Create the WSGI App, first move to the ```/var/www``` directory and create a folder called FlaskApp, then pull the Car Sensor repository inside the FlaskApp folder and rename it. The FlaskApp folder is the WSGI App folder which will contain the Car Sensor Application folder and a .wsgi file for executing the Car Sensor App.  
    ```$ cd /var/www/```  
    ```$ sudo mkdir FlaskApp```  
    ```$ cd FlaskApp/```  
    ```$ sudo git clone https://github.com/YuezhouLu/Item-Catalog.git```  
    ```$ sudo mv Item-Catalog CarSensor```

4. Move into the CarSensor folder, rename the ```app.py``` file to ```__init__.py```. Install the Python Pip package manager, then use Pip to install all the dependencies of the Car Sensor App (check the import in all files in the Car Sensor App).  
    ```$ cd CarSensor/```  
    ```$ sudo mv app.py __init__.py```  
    ```$ sudo apt-get install python-pip```  
    ```$ sudo pip install sqlalchemy```  
    ```$ sudo pip install datetime```  
    ```$ sudo pip install flask```  
    ```$ sudo pip install oauth2client```  
    ```$ sudo pip install httplib2```  
    ```$ sudo pip install requests```

5. Move back to the FlaskApp folder, and create the .wsgi file.  
    ```$ cd ../```  
    ```$ sudo nano carsensor.wsgi```

    Put the following lines of code into the carsensor.wsgi file.

        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0, "/var/www/FlaskApp/")
        from CarSensor import app as application
        application.secret_key = 'SUPER_SECRET_KEY'

6. Configure and enable a new virtual host by creating a new .conf file. This .conf file tells Apache which WSGI App to run.  
    ```$ sudo nano /etc/apache2/sites-available/FlaskApp.conf```
 
    Add the following content.

        <VirtualHost *:80>
            ServerName 13.115.180.187
            ServerAdmin yeuzhoulu@gmail.com
            WSGIScriptAlias / /var/www/FlaskApp/carsensor.wsgi 
            <Directory /var/www/FlaskApp/CarSensor/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/FlaskApp/CarSensor/static 
            <Directory /var/www/FlaskApp/CarSensor/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log 
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

    Enable the virtual host with the following command.  
    ```$ sudo a2ensite FlaskApp```

### VII - Install and configure the PostgreSQL database server
1. Install PostgreSQL.  
    ```$ sudo apt-get install postgresql```

2. Check if remote connections to PostgreSQL have been blocked. [(Source: DigitalOcean)](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

3. Login as the default user postgres and get into the PostgreSQL shell psql.  
    ```$ sudo -u postgres psql```

4. Create a new database named mygarage and create a new user named catalog.  
    ```$ create database mygarage;```  
    ```$ create user catalog;```

5. Set a password for user catalog, and give user catalog the permission to access mygarage database, then quit PostgreSQL.
    ```$ alter role catalog with password "password";```  
    ```$ grant all privileges on database mygarage to catalog;```  
    ```$ \q```

6. Solve the authentication failure. [(Source: Stack Overflow)](https://stackoverflow.com/questions/18664074/getting-error-peer-authentication-failed-for-user-postgres-when-trying-to-ge)  
    ```$ sudo nano /etc/postgresql/10/main/pg_hba.conf```

    Add a new line ```local all catalog md5 ``` below ```local all postgres peer```, then save the file and quit.

7. Go to the CarSensor App folder and edit files: ```database_setup.py```, ```lotsofcars.py``` and ```__init__.py```.  
    ```$ cd /var/www/FlaskApp/CarSensor/```

    In each of the 3 files, change the code ```engine = create_engine('sqlite:///mygarage.db')``` to ```engine = create_engine('postgresql://catalog:password@localhost/mygarage')```

8. Install psycopg2, create the database (mygarage) and populate seed data.  
    ```$ sudo apt-get install python-psycopg2```  
    ```$ python database_setup.py```  
    ```$ python lotsofcars.py```

9. Update the path of client_secrets.json in \_\_init\_\_.py to be absolute path: ```/var/www/FlaskApp/CarSensor/(fb_)client_secrets.json```.

### VIII - Restart Apache and PostgreSQL server.
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```$ sudo service postgresql restart```  
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;```$ sudo service apache2 restart```


## Important Notes
* Apache Server: ```sudo cat (or tail -20) /var/log/apache2/error.log``` for debugging, this log file is like the console for showing internal errors.
+ Postgresql Database Server: edit ```/etc/postgresql/10/main/pg_hba.conf``` for solving the peer authentication failure. Change the auth-method to trust or md5 (MD5-encrypted password) instead of peer for the record of catalog (not postgres).
+ WSGI Absolute File Path: change file path of client_secrets.json to absolute path: ```/var/www/FlaskApp/CarSensor/(fb_)client_secrets.json```.
+ OAuth2.0 Developer Console Update: add the hostname and the public IP address to the Authorized JavaScript Origins, as well as update the Authorized Redirect URIs on Google's and Facebook's Developer Console.
- IP is the real address, a normal address such as ```www.example.com``` needs to ask a DNS service where it should go, and the DNS will tell the browser the correct IP. If want a normal address, then buy it from [Google Domains](https://domains.google/#/), then on its control panel point the address to the Lightsail IP.

    > P.S.: there are three ways to edit files in a virtual machine:
    > 1. the standard and normal method is to use Git to pull the repositories.
    > 2. use terminal-based text editors such as nano and vim to edit files directly.
    > 3. if for vagrant (not public virtual machine), we can attach files by dragging them into the folder, then define and link them in the Vagrantfile.


## Future Improvements
Need to complete some final updates on OAuth2.0 settings to make the App run properly on AWS Lightsail. The styling of the App has to be refined, especially the mobile version.


## References
1. https://discussions.udacity.com/t/last-exercise-to-configure-wsgi-for-postgresql/45142/8
2. https://medium.com/@mariasurmenok/creating-a-server-with-amazon-lightsail-11c377cf814c
3. https://github.com/kongling893/Linux-Server-Configuration-UDACITY
4. https://libraries.io/github/golgtwins/Udacity-P7-Linux-Server-Configuration
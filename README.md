# Linux Server Configuration

A baseline installation of a Linux server and preparing it to host our web applications, securing it from a number of attack vectors, installing and configuring a database server, and deploying 
our on of your existing web applications onto it.

## How To Run

### Link To Server

* **Public IP Address:** 18.221.232.119
* **Accessible SSH port:** 2200

### Linux Server Configuration and Software Installed

#### 1. Create Development Environment Instance
  * [Create new development environment here.](https://www.udacity.com/account#!/development_environment)
  * Download private key provided and note down your public IP address.
  
#### 2. Launch VM and access SSH to the instance
  * Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory).
  
  ```
    $ mv /(current_private_key_address)/udacity_key.rsa ~/.ssh/
  ```
  * Change the key permission so that only owner can read and write
  
  ```
    $ chmod 600 ~/.ssh/udacity_key.rsa
  ```
  * SSH into the instance
  
  ```
    $ ssh -i ~/.ssh/udacity_key.rsa root@public_IP_address
   ```
   
#### 3. Create New User
  
  * Add User grader
  
  ```
    $ sudo adduser grader
  ```
  * Give Sudo Access to grader
  
  ```
    $ sudo nano /etc/sudoers.d/grader
  ```
  * Add following line to this file
  
  ```
    grader ALL=(ALL:ALL) ALL
  ```
  * To prevent the "sudo: unable to resolve host" error
  
   i. Edit the hosts file:
   
   ```
     $ sudo nano /etc/hosts
   ```
   ii. Add the host:
   
   ```
     $ 127.0.1.1 ip-XX-XX-XX-XX
   ```

#### 4. Configure the key-based authentication for grader user
  *  Generate an encryption key on your local machine
  
   i. Go to the directory where you want to save the Key, and run the following command:
   
   ```
    $ ssh-keygen -t rsa
   ```
      followed by the name of the key. By default, the keys will be stored in the ~/.ssh directory within your user's 
      home directory.
      
   ii. Place the public key on the server that we want to use:
   
   ```
    $ ssh-copy-id grader@XX.XX.XX.XX -i (key_name.pub)
   ```
   iii. Log into remote VM as root user and open the following file:
   
   ```
    $ cat /.ssh/authorized_keys
   ```
      and copy it's content using `$ nano /home/grader/.ssh/authorized_keys`
  
  Now we can log into the remote VM through ssh with the following command: 
  
  ```
   $ ssh -i udacity_key.rsa grader@XX.XX.XX.XX 
  ```

#### 5. Enforce key-based authentication
  * Run `$ sudo nano /etc/ssh/sshd_config`.
  * Find the **PasswordAuthentication** line and edit it to no.
  * Save the file.
  * Run `$ sudo service ssh restart` to restart the service.

#### 6. Change the SSH port from 22 to 2200
  * Find the **Port line** in the same file above, i.e */etc/ssh/sshd_config* and edit it to 2200.
  * Save the file.
  * Run `$ sudo service ssh restart` to restart the service.
  
#### 7. Disable login for root user
  * Find the **PermitRootLogin** line in the same file above, i.e */etc/ssh/sshd_config* and edit it to no.
  * Save the file.
  * Run `$ sudo service ssh restart` to restart the service.

Now we can login into remote VM through SSH with following command:
```
 $ ssh -i ~/.ssh/udacity_key.rsa grader@XX.XX.XX.XX -p 2200
```

Source: [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1739013)

#### 8. Configure local timezone to UTC
  * Change the timezone to UTC using following command: `$ sudo timedatectl set-timezone UTC`.
  * You can also open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.
  * Install ntp daemon ntpd for a better synchronization of the server's time over the network connection:
  
  ```
   $ sudo apt-get install ntp
  ```
 Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)

#### 9. Update all currently installed packages
  * `$ sudo apt-get update`.
  * `$ sudo apt-get upgrade`.

#### 10. Configure the Uncomplicated Firewall (UFW)
  ```
   $ sudo ufw default deny incoming
   $ sudo ufw default allow outgoing
   $ sudo ufw allow 2200/tcp
   $ sudo ufw allow www
   $ sudo ufw allow ntp
   $ sudo ufw enable
  ```

#### 11. Configure cron scripts to automatically manage package updates
  * Install unattended-upgrades if not already installed using command:
  
  ```
   $ sudo apt-get install unattended-upgrades
  ```
  * Enable it using command:
  
  ```
   $ sudo dpkg-reconfigure --priority=low unattended-upgrades
  ```

#### 12. Install and Configure Apache2, mod-wsgi and Git
 ```
  $ sudo apt-get install apache2 libapache2-mod-wsgi git
 ```
 * Enable mod_wsgi:
 
 ```
  $ sudo a2enmod wsgi
 ```
#### 13. Install Flask and other dependencies

```
    $ sudo apt-get install python-pip
    $ sudo pip install Flask
    $ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
    $ sudo pip install requests
  ```
Source: [Flask Documentation](http://flask.pocoo.org/docs/0.12/installation/)

#### 14. Clone the Catalog app from Github

  * Make a *catalog* named directory in */var/www*

    ```
      $ sudo mkdir /var/www/catalog
    ```

  * Change the owner of the directory *catalog*

    ```
     $ sudo chown -R grader:grader /var/www/catalog
    ```

  * Clone the **Priyanka_ItemCatelog** to the catalog directory:

    ```
     $ git clone https://github.com/priyankasingh2210/Priyanka_ItemCatelog
    ```

  * Change the branch of repo **Priyanka_ItemCatelog**  to **deployment**:

    ```
     $ cd catalog && git checkout deployment
    ```

  * Make a flaskApp.wsgi file to serve the application over the mod_wsgi. with content:

    ```
     $ touch flaskApp.wsgi && nano flaskApp.wsgi
    ```

    ```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from Priyanka_ItemCatelog import app as application
    ```
  
    ```
  * Run the database_setup.py and dummyBooks.py once to setup database with dummy data:
  
  ```
   $ python database_setup.py
   $ python dummybooks.py
  ```

#### 15. Edit the default Virtual File with following content:

  ```
    $  sudo nano /etc/apache2/sites-available/000-default.conf
  ```


  ```
  <VirtualHost *:80>
    ServerName XX.XX.XX.XX
    ServerAdmin priyankasingh.ps.90@gmail.com
    WSGIScriptAlias / /var/www/catalog/flaskApp.wsgi
    <Directory /var/www/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/static
    <Directory /var/www/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
  </VirtualHost>
  ```
#### 16. Restart Apache to launch the app

   ```
    $ sudo service apache2 restart
   ```

## Authors

* **Priyanka Singh** 

## Acknowledgments

* A big thank you to Udacity's full stack nanodegree program



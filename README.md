# Linux_Server_Configuration
Configuring a Linux Server to host a web app securely.

# Server Details

**Ip address** : 13.234.38.177

**The url to hosted web page is** : ec2-13-234-38-177.ap-south-1.compute.amazonaws.com

**Port**       : 2200

# Configuration with Amazon Light sail 

### Create an instance with amazon light sail account.
1. Sign up with [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) if yoy don't have an account.
2. After succesfully log into your account click on the button create instance.
3. Then select **os only** and select your ubuntu version, my version was (16.04 lTS) and after that drag to bottom and create the instance.
4. It takes some time to start before we can use it.
5. Click on the three line bar and select manage and a new page will open drag to bottom and select **account page**.
6. Note down your public ip and private ip and downlaod the private default key which placed at little bit below.

# Connect to the instance on Local Machine

1. Save the default key in the shared directory (where vagrant file resides) and save with some name which contains less words. I save my file as **sam.pem**.
2. Now we need to move that file to **.ssh** directory in your vagrant machine which runs ubuntu. By using the command

         1. sudo cp -R /vagrant/sam.pem ~/.ssh/sam.pem
3. We need to make our public key usable and secure. Going back to your terminal, input

         2. chmod 600 ~/.ssh/YourAWSKey.pem
         
4. Now log in with your Amazon light sil ip address.

       3. ssh -i ~/.ssh/sam.pem ubuntu@13.234.38.177(Note that we can only log in as an ubuntu but not as an root as it doesn't permits to login as root)
# Upgrade currently installed packages
1. Notify the system of what package updates are available by running.

        4.sudo apt-get update
2. Download available package updates by running sudo apt-get upgrade

        5. sudo apt-get upgrade


# Configure the firewall
1. Start by changing the SSH port from 22 to 2200 (open up the /etc/ssh/sshd_config file, change the port number on line 5 to 2200, then restart SSH by running sudo service ssh restart; restarting SSH is a very important step!).


2. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running.

        6. sudo ufw status


3.  To set the ufw firewall to block everything coming in

        7. sudo ufw default deny incoming
        
        
4. To set the ufw firewall to allow everything outgoing

        8.  sudo ufw default allow outgoing
        
        
5. To set the ufw firewall to allow SSH

        9. sudo ufw allow ssh
6. To allow all tcp connections for port 2200 so that SSH will work

         10. sudo ufw allow 2200/tcp 
7. To set the ufw firewall to allow a basic HTTP server

         11. sudo ufw allow www
8. To set the ufw firewall to allow NTP
           
         12. ufw allow 123/udp
9. To deny port 22 (deny this port since it is not being used for anything; it is the default port for SSH, but this virtual machine has now been configured so that SSH uses port 2200)

        13. sudo ufw deny 22
10. To enable the ufw firewall

        14. sudo ufw enable

11. To check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

        15. sudo ufw status 
            To                         Action      From
             --                         ------      ----
            22                         DENY        Anywhere
            2200/tcp                   ALLOW       Anywhere
            80/tcp                     ALLOW       Anywhere
            123/udp                    ALLOW       Anywhere
            22 (v6)                    DENY        Anywhere (v6)
            2200/tcp (v6)              ALLOW       Anywhere (v6)
            80/tcp (v6)                ALLOW       Anywhere (v6)
            123/udp (v6)               ALLOW       Anywhere (v6)

12. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed; make sure to deny the default port 22).


13. Now, to login, open up the Terminal and run:
            
            16.ssh -i ~/.ssh/sam.pem -p 2200 ubuntu@13.234.38.177

Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port 22, which is now denied.

# Create a new user named grader
1. To add a new user run

       17. sudo adduser grader
2. Enter in a new UNIX password (twice) when prompted
3. Fill out information for the new grader user
4. To switch to the grader user, run

        18. su - grader, and enter the password
          
# Give grader user sudo permissions
1. Run 
        
        19. sudo visudo
2. Search for a line that looks like this:
    
       root ALL=(ALL:ALL) ALL
3. Add the following line below this one:

       20. grader ALL=(ALL:ALL) ALL
4. Save and close the visudo file.


5. To verify that grader has sudo permissions, su as grader (run su - grader), enter the password, and run sudo -l; after entering in the password (again), a line like the following should appear, meaning grader has sudo permissions:

          [sudo] password for grader:
          Matching Defaults entries for grader on ip-172-26-7-206.ap-south-1.compute.internal:
              env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

          User grader may run the following commands on ip-172-26-7-206.ap-south-1.compute.internal:
              (ALL : ALL) ALL

# Allow grader to log in to the virtual machine
1. Open a new Terminal window (Command+N) and input

        21. ssh-keygen -f ~/.ssh/udacity_key.rsa
2. Stay on the same Terminal window, input
          
         22.cat ~/.ssh/udacity_key.rsa.pub to read the public key. (Copy the public key)
3. Going back to the first terminal window where you are logged into Amazon Lightsail as the root user, move to grader's folder by
          
          23. $ cd /home/grader
4. Create a .ssh directory: 
            
           24. $ mkdir .ssh
5. Create a file to store the public key: 
            
            25. $ touch .ssh/authorized_keys
6. Edit the authorized_keys file 
              
             26. $ nano .ssh/authorized_keys
7. Change the permission: 

             27. $ sudo chmod 700 /home/grader/.ssh 
        and
             28. $ sudo chmod 644 /home/grader/.ssh/authorized_keys
8. Change the owner from root to grader:

              29. $ sudo chown -R grader:grader /home/grader/.ssh
9. Restart the ssh service: 
               
               30.$ sudo service ssh restart
10. Type **$ ~.** to disconnect from Amazon Lightsail server
11. Log into the server as grader: 
        
        31. $ ssh -i ~/.ssh/udacity_key.rsa grader@13.234.38.177
12. We now need to enforce the key-based authentication: 

          32. $ sudo nano /etc/ssh/sshd_config. 
**Find the PasswordAuthentication line and change text after to no**
13. After this, restart ssh again: 
           
           33. $ sudo service ssh restart
14. We now need to change the ssh port from 22 to 2200, as required by Udacity:

          34. $ sudo nano /etc/ssh/ssdh_config Find the Port line and change 22 to 2200.
          35. Restart ssh: $ sudo service ssh restart
15. Disconnect the server by **$ ~.** and then log back through port 2200:

          36. $ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.58.109.116
16. Disable ssh login for root user, as required by Udacity:

          37.$ sudo nano /etc/ssh/sshd_config. Find the PermitRootLogin line and edit to no.
          38. Restart ssh $ sudo service ssh restart
 17. If you are changing the roor login permissions always remember **prohibit password** option doesn't mean to stop ssh login. we need to disable it by replacing it with **no**.
# Configure the local timezone to UTC
1. Run , and follow the instructions (UTC is under the 'None of the above' category)
        
        39. sudo dpkg-reconfigure tzdata
2. Test to make sure the timezone is configured correctly by running
                    
         40.date
# Fix sudo resolve host error
When the grader user issues a sudo command, got the following warning: sudo: unable to resolve host ip-10-20-47-177
To fix this, the hostname was added to the loopback address in the /etc/hosts file so that th first line now reads:

		127.0.0.1 localhost ip-10-20-47-177
This solution was found on the Ubuntu Forms here.
# Install and configure Apache
1. To install Apache
      
       41. sudo apt-get install apache2
2. Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load
# Install mod_wsgi
1. Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) along with python-dev (a package with header files required when building Python extensions); use the following command:


       42.sudo apt-get install libapache2-mod-wsgi python-dev
2. Make sure mod_wsgi is enabled by running 

        43.sudo a2enmod wsgi
# Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
1. Install PostgreSQL by running
        
        44. sudo apt-get install postgresql
2. Open the 
        
        45. sudo nano /etc/postgresql/9.5/main/pg_hba.conf file
            Make sure it looks like this (comments have been removed here for easier reading):
            local   all             postgres                                peer
            local   all             all                                     peer
            host    all             all             127.0.0.1/32            md5
            host    all             all             ::1/128                 md5
# Make sure Python is installed
Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run python. Something like the following should appear:
Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
# Create a new PostgreSQL user named catalog with limited permissions
1. PostgreSQL creates a Linux user with the name postgres during installation; switch to this user by running 

        46.sudo su - postgres (for security reasons, it is important to only use the postgres user for accessing the PostgreSQL software)
2. Connect to psql (the terminal for interacting with PostgreSQL) by running 

         47. psql
3. Create the catalog user by running 
          
         48. CREATE ROLE catalog WITH LOGIN;
4. Next, give the catalog user the ability to create databases: 

         49. ALTER ROLE catalog CREATEDB;
5. Finally, give the catalog user a password by running

          50. \password catalog
6. Check to make sure the catalog user was created by running
            
           60.\du; 
 7.a table of sorts will be returned, and it should look like this:
				   
     Role name | List of roles                     Attributes               | Member of 
    -----------+------------------------------------------------------------+-----------
     catalog   | Create DB                                                  | {}
     postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
8. Exit psql by running 
          
          61. \q
9. Switch back to the ubuntu user by running 
          
          62. exit
# Create a Linux user called catalog and a new PostgreSQL database
1. Create a new Linux user called catalog:
        
        63. sudo adduser catalog
2. Enter in a new UNIX password (twice) when prompted
fill out information for catalog
Give the catalog user sudo permissions:
   
          64. sudo visudo
3. Search for a line that looks like this:

          root ALL=(ALL:ALL) ALL
4. Add the following line below this one: 
          
          65. catalog ALL=(ALL:ALL) ALL
5. save and close the visudo file
6. To verify that catalog has sudo permissions, su as catalog (run sudo su - catalog), and run sudo -l
after entering in the UNIX password, a line like the following should appear (meaning catalog has sudo permissions):
 User catalog may run the following commands on
 	ip-13-234-38-177.ec2.internal:
     (ALL : ALL) ALL
7. While logged in as catalog, create a database called catalog by running 
      
             66. createdb catalog
8. Run **psql** and then run **\l** to see that the new database has been created.
9. Switch back to the ubuntu user by running **exit**
10. Install git and clone the catalog project
       
         67. sudo apt-get install git
11. Create a directory called **'bermuda'** in the /var/www/ directory
12. Change to the 'bermuda' directory, and clone the catalog project:
                  
          68. sudo git clone https://github.com/sampathatmuri/Menu_Item_Catalog.git bermuda
13. Change the ownership of the 'bermuda' directory to ubuntu by running (while in /var/www):

          69. sudo chown -R ubuntu:ubuntu bermuda/
14. Change to the 
    
          70. cd /var/www/bermuda/bermuda directory
15. Change the name of the application.py file to __init__.py by running mv project.py __init__.py
16. In __init__.py,(Open __init__.py)

            app.run(host='0.0.0.0', port=5000)
            Change this line to:
            app.run()

# Set up a vitual environment and install dependencies
1. Start by installing pip (if it isn't installed already) with the following command:
        
        71. sudo apt-get install python-pip
2. Install virtualenv with apt-get by running 
        
        72. sudo apt-get install python-virtualenv
3. Change to the /var/www/nuevoMexico/nuevoMexico/ directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running 
          
         73. virtualenv venv (make sure to not use sudo here as it can cause problems later on)
4. Activate the new environment, venv, by running
          
          74. . venv/bin/activate
5. With the virtual environment active, install the following dependenies (note: with the exception of the libpq-dev package, make sure to not use sudo for any of the package installations as this will cause the packages to be installed globally rather than within the virtualenv):

           75. pip install httplib2
           76. pip install requests
           77. pip install --upgrade oauth2client
           78. pip install sqlalchemy
           79. pip install flask
           80. sudo apt-get install libpq-dev (Note: this will install to the global evironment)
           81. pip install psycopg2
6. In order to make sure everything was installed correctly, run python __init__.py; the following (among other things) should be returned:

          * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
7. Deactivate the virtual environment by running 
            
            83. deactivate
8. Set up and enable a virtual host
9. Create a file in 
          
          84.sudo nano /etc/apache2/sites-available/ called bermuda.conf
Add the following into the file:
                        
                        <VirtualHost *:80>
                            ServerName 13.234.38.177
                            ServerAdmin sm@gmail.com
                            WSGIDaemonProcess catalog python-path=/var/www/bermuda:/var/www/bermuda/venv/lib/python2.7/site-packages
                            WSGIProcessGroup bermuda
                            WSGIScriptAlias / /var/www/bermuda/bermuda.wsgi
                            <Directory /var/www/bermuda/bermuda/>
                              Order allow,deny
                              Allow from all
                              Options -Indexes
                            </Directory>
                            Alias /static /var/www/bermuda/bermuda/static
                            <Directory /var/www/bermuda/bermuda/static/>
                              Order allow,deny
                              Allow from all
                              Options -Indexes
                            </Directory>
                            ErrorLog ${APACHE_LOG_DIR}/error.log
                            LogLevel warn
                            CustomLog ${APACHE_LOG_DIR}/access.log combined
                        </VirtualHost>
10. Note: the Options -Indexes lines ensure that listings for these directories in the browser is disabled.
11. To enable the virtual host
      
        85.sudo a2ensite nuevoMexico
12. The following prompt will be returned:
		
		Enabling site bermuda	
		To activate the new configuration, you need to run:
		service apache2 reload
13. sudo service apache2 reload
# Write a .wsgi file
1. Apache serves Flask applications by using a .wsgi file; create a file called bermuda.wsgi in /var/www/bermuda
Add the following to the file:
activate_this = '/var/www/bermuda/bermuda/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/bermuda/")

	from bermuda import app as application
	application.secret_key = '12345'
14. Resart Apache: 
	
		86. sudo service apache2 restart
# Switch the database in the application from SQLite to PostgreSQL
Replace the engine creation files in __init__.py, databasesetup.py and lotsofmenus.py.
		
		engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')
# Disable the default Apache site
1. At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run
		
		87. sudo a2dissite 000-default.conf
2. The following prompt will be returned:
		Site 000-default disabled.
3. To activate the new configuration, you need to run: sudo service apache2 reload
		
		88. sudo service apache2 reload
# Change the ownership of the project direcotries


1. Change the ownership of the project directories and files to the www-data user (this is done because Apache runs as the www-data user); while in the /var/www directory, run:
		
		89. sudo chown -R www-data:www-data bermuda/
2. Note: if changes need to be made to the project files after the ownership of the directories has been switched to www-data, it is best to edit files as the www-data user; do this with the following command:
		
		90.sudo -u www-data vim INSERT_NAME_OF_FILE
(Note: vim can be replaced here with nano or another text editor.)
3. Set up the database schema and populate the database
While in the /var/www/bermuda/bermuda/ directory, activate the virtualenv by running 
		
		91. . venv/bin/activate
4. Then run python databasesetup.py
5. Deactivate the virtualenv run

		92. deactivate
6. 	Resart Apache again: sudo service apache2 restart

# Update the Google OAuth client secrets file
Fill in the client_id and client_secret fields in the file g_client_secrets.json. Also change the javascript_origins field to the IP address and AWS assigned URL of the host. In this instance that would be: "javascript_origins":["http://13.234.38.177", "http://"ec2-13-234-38-177.ap-south-1.compute.amazonaws.com"]
These addresses also need to be entered into the Google Developers Console -> API Manager -> Credentials, in the web client under "Authorized JavaScript origins".
Update the Facebook OAuth client secrets file
In the file fb_client_secrets.json, fill in the app_id and app_secret fields with the correct values.
In the Facebook developers website, on the Settings page, the website URL needs to read http://ec2-13-234-38-177.ap-south-1.compute.amazonaws.com. Then in the "Advanced" tab, in the "Client OAuth Settings" section, add http://ec2-13-234-38-177.ap-south-1.compute.amazonaws.com and http://13.234.38.177 to the "Valid OAuth redirect URIs" field. Then save these changes.

# Automatic Updates
1. Automatic security updates have been implementing using the instructions in the Ubuntu documentation, Automatic Updates.
2. Check that the unattended-upgrades package is installed:
		
		sudo apt-get install unattended-upgrades

3. Check the configuration file /etc/apt/apt.conf.d/50unattended-upgrades to see which class of updates get installed. It was left at the default of security updates only.
4. Change the configuration file /etc/apt/apt.conf.d/10periodic to specify how often updates and other tasks should be performed. It's contents are now:
		
		APT::Periodic::Update-Package-Lists "1";
		APT::Periodic::Download-Upgradeable-Packages "1";
		APT::Periodic::AutocleanInterval "7";
		APT::Periodic::Unattended-Upgrade "1";
5. So the package list is updated daily and security updates will be downloaded and installed daily. Every week the local download archive will be cleaned.


6. Unattended package installation can be monitiored by reviewing the log file located here: /var/log/apt/unattended-upgrades/unattended-upgrades.log.


# Monitor for Repeated Failed Login Attempts
The program Fail2Ban will be used to block IP addresses from which unsuccessful login attempts have occurred. This is on top of using a non-standard SSH port number and requiring an RSA key to login. The following guide was useful in this task: How To Protect SSH with Fail2Ban on Ubuntu 14.04. Also this guide for setting up Fail2ban when the UFW is being used: UFW with Fail2ban – Quick Secure Setup. ### Although I needed this page to confirm a detail in that post.
1. Install Fail2Ban:

		sudo apt-get install fail2ban
2. Install Sendmail to send email notifications of banned IP addresses:

		sudo apt-get install sendmail
3. Copy the Fail2Ban configuration file to a local config file that will override it. This way, if their updates to the default config file, they will not have to be merged with any changes the user makes.
		
		sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
4. Edit this file with:

		sudo nano /etc/fail2ban/jail.local
5. I set the following settings that differ from the default. Within the [DEFAULT] section:
		bantime = 1800

		destemail = [my email address]

		action = %(action_mwl)s
		This bans an IP address for 1800 seconds and sends an email to the address specified. The action means an email notification will be sent with logging extract included.
		The ssh section is as follows:
		[ssh]

		enabled  = true
		banaction = ufw-ssh
		port     = 2200
		filter   = sshd
		logpath  = /var/log/auth.log
		maxretry = 3
6. The banaction is set to run a custom action ufw-ssh which will be defined below. It uses the UFW frontend to the iptables firewall. This will mean blocked IP address will appear in the output of sudo ufw status. The port is set to our customised SSH port.
Contents of /etc/fail2ban/action.d/ufw-ssh.conf is:
			
			[Definition]
			actionstart =
			actionstop =
			actioncheck =
			actionban = ufw insert 1 deny from <ip> to any port 2200
			actionunban = ufw delete deny from <ip> to any port 2200
7. So Fail2Ban will insert a deny rule as the first rule from the relevant IP address for the custom SSH port 2200. After the 1800 second ban time, this rule will be deleted.

8.To make the new settings take effect, stop and start the fail2ban service with:
		
		sudo service fail2ban stop
		sudo service fail2ban start
# System Monitoring
1. Glances is a full-featured system monitor. It can also monitor processes. Install Glances with:
		
		sudo apt-get install glances
2. To monitor Apache and Postgres, the following lines were added to the Glances config file /etc/glances/glances.conf:
		
		list_1_description=Apache Server
		list_1_regex=.*apache.*
		list_2_description=Postgres
		list_2_regex=.*postgres.*

Now open up a browser and check to make sure the app is working by going to http://13.234.38.177 or http://ec2-13-234-38-177.ap-south-1.compute.amazonaws.com


Linux Server Configuration

1-	Linux Server :

• The Linux distribution is Ubuntu 14 LTS.
• The virtual private server is Amazon Lighsail, IP address : 52.31.34.58  
• The web application is Item
• The database server is PostgreSQL.
• My local machine is Windows.

2- Configuration :
• Go to AWS Lightsail and login using an Amazon Web Services account .
• Choose Linux/Unix platform, OS Only and Ubuntu 14 LTS .
• Choose a instance plan (the cheapest plan is enough) .
• Keep the default AWS name or rename your instance .
• Click Create button to create an instance .

3- SSH connect to the server :
• From the Account menu on Amazon Lightsail, click on SSH keys tab and download the Default Private Key.
• Move this file named LightsailDefaultPrivateKey-*.pem into the local folder and rename it lightsail_key.rsa.
• From that local folder run ssh command ( I am using Git bash ) .
• In your terminal, type: chmod 600 lightsail_key.rsa .
• In your terminal, type: ssh -i lightsail_key.rsa ubuntu@52.209.228.104 and press Enter, which (52.209.228.104) is the public IP.

A - User Management

1- Create a new user account named grader and give sudo access :
• While logged in as Ubuntu .
• Add user sudo adduser grader .
• Enter a password and retype it and complete all information for this new user.
• Give to grader user the permission to sudo. Run: sudo visudo . 
• Add to the lines, like this :
	root    ALL=(ALL:ALL) ALL
	catalog  ALL=(ALL:ALL) ALL
• Save and exit using CTRL+X and confirm with Y.
• Verify that grader has sudo permissions. Run su - grader, enter the password, run sudo -l and enter the password again. The output should be like this:
	User grader may run the following commands on ........
   	 (ALL : ALL) ALL	
• Copy the generated SSH to a virtual environment.
• If you are not login already by user grader, login su – grader .
• Create a new directory  mkdir .ssh .
• Create a new file touch .ssh/authorized_keys .
• nano .ssh/authorized_keys and copy your generated SSH key here.
• Give the permissions: chmod 700 .ssh and chmod 644 .ssh/authorized_keys .

2- Remote login of the root user is disabled :
	Open /etc/ssh/sshd_config and find PermitRootLogin and change it to no.

B- Security

1- firewall configured to only allow for SSH, HTTP, and NTP :
• Edit /etc/ssh/sshd_config file by sudo nano /etc/ssh/sshd_config .
• Change port 22 to 2200 .
• Save the change by Control + X and exit from nano with Y .
• Restart SSH with sudo service ssh restart .
Configure UDW to allow only incoming request from port2200(SSH), port80 (HTTP) and port123 (NTP).

sudo ufw status >>  ufw should be inactive

sudo ufw default deny incoming  >>  deny all incoming requests

sudo ufw default allow outgoing  >>  allow all outgoing requests

sudo ufw allow 2200/tcp  >>  allow incoming ssh request

sudo ufw allow 80/tcp  >>  allow all http request

sudo ufw allow 123/udp  >>  allow ntp request

sudo ufw deny 22  >>  deny incoming request for port 22

sudo ufw enable  >>  enable ufw

sudo ufw status  >>  check current status of ufw

• Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the internal firewall settings above.
• Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.
• Restart SSH with sudo service ssh restart . 
• From your local terminal, run: ssh -i lightsail_key.rsa -p 2200 ubuntu@52.31.34.58  and press Enter, which (52.31.34.58) is the public IP.


2- Users required to authenticate using RSA keys :

	Set option PasswordAuthentication in file /etc/ssh/sshd_config to no .

	
3- applications up-to-date :
            
              Run sudo apt-get update and sudo apt-get upgrade .

4- SSH is hosted on non-default port :Hosted on port 2200 .

5- Set up local time zone :
             Run sudo dpkg-reconfigure tzdata and choose  Asia/Aden .

C- Application Functionality

1- Install Apache application and wsgi module :
• install apache sudo apt-get install apache2 .
• Install the Python 3 mod_wsgi package sudo apt-get install libapache2-mod-wsgi-py3 .
• Enable mod_wsgi using: sudo a2enmod wsgi .
• Start apache sudo service apache2 start .
• Check apache2 status sudo service apache2 status is it active ?
• Open your browser and type ip   34.241.243.81  you should see this :
 

2- Install git :
• Run sudo apt-get install git .
• Configure your username and email. git config --global user.name <username> and git config --global user.email <email> .


3- Install and configure PostgreSQL :
• Install PostgreSQL: sudo apt-get install postgresql .
• PostgreSQL should not allow remote connections. In the /etc/postgresql/9.3/main/pg_hba.conf file .
• Switch to the postgres user: sudo su - postgres.
• Open PostgreSQL interactive terminal with psql .
• Create the user catalog with a password and give him the ability to create databases: 
		postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
		postgres=# ALTER ROLE catalog CREATEDB;
• Check roles : \du .
• Exit psql: \q .
• sign back to grader .
• Create a new Linux user catalog: sudo adduser catalog. Enter password and complete information.
• Give to catalog user the permission to sudo. Run: sudo visudo.
• Add to the lines, like this:
root    ALL=(ALL:ALL) ALL
grader ALL=(ALL:ALL) ALL
catalog  ALL=(ALL:ALL) ALL
• Save and exit by CTRL+X and confirm with Y.
• While logged in as catalog, create a database: createdb catalog .
• Run psql and then run \l to see that the new database has been created.
• Exit psql: \q .
• Go back to the grader user: exit .

4- Clone and setup the Item Catalog project from GitHub :
• While logged in as grader, create /var/www/catalog/ directory, Run : sudo mkdir catalog .
• Change to that directory and clone the catalog project:
		https://github.com/thecoder066/1_Item.git catalog .
• From the /var/www directory, change the owner of the catalog directory to grader sudo chown -R grader:grader catalog .
• Change to the /var/www/catalog/catalog directory.
• Rename the application.py file to __init__.py using: mv project.py__init__.py to deploy on AWS .
• In __init__.py, replace:
		# app.run(host="0.0.0.0", port=8000, debug=True)
		app.run()
• In database.py, replace:
		# engine = create_engine("sqlite:///catalog.db")
		engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
		You will put instead of ( PASSWORD ) your password for postgresql which in our  case is ( catalog)  like this :
		engine = create_engine('postgresql://catalog: catalog@localhost/catalog')

5- Install the virtual environment and dependencies :
• While logged in as grader, install pip: sudo apt-get install python3-pip.
• Install the virtual environment: sudo apt-get install python-virtualenv .
• Change to the /var/www/catalog/catalog/ directory.
• Create the virtual environment: sudo virtualenv -p python3 venv3.
• Change the ownership to grader with: sudo chown -R grader:grader venv3/ .
• Activate the new environment: . venv3/bin/activate .
• Install the following dependencies:
		pip install httplib2
		pip install requests
		pip install --upgrade oauth2client
		pip install sqlalchemy
		pip install flask
		sudo apt-get install libpq-dev
		pip install psycopg2
• Run python3 __init__.py and you should see:
	* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit).

• Deactivate the virtual environment: deactivate .

6- Set up and enable a virtual host :
	•Add the following line in /etc/apache2/mods-enabled/wsgi.conf file to use Python 3 : ( run sudo nano wsgi.conf)
		#WSGIPythonPath directory|directory-1:directory-2:...
		WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
• Create /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host:
	<VirtualHost *:80>
    	ServerName  52.31.34.58 
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
• Enable virtual host: sudo a2ensite catalog .
• Reload Apache: sudo service apache2 reload .

7- Set up the database schema and populate the database :
• Edit /var/www/catalog/catalog/data.py .
• Replace lig.random_para(250) by lig.random_para(240) on lines 86, 143, 191, 234 and 280.
• Add the these two lines at the beginning of the file :
	import sys
	sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.5/site-packages")
• Add the following lines under create_db()  :
	# Create database.
	create_db()

	# Delete all rows.
	session.query(Item).delete()
	session.query(Category).delete()
	session.query(User).delete()
• From the /var/www/catalog/catalog/ directory, activate the virtual environment: . venv3/bin/activate .
• Run: python data.py .
• Deactivate the virtual environment: deactivate .

8- Disable the default Apache site :
• Disable the default Apache site: sudo a2dissite 000-default.conf. The following prompt will be returned:
	Site 000-default disabled.
	To activate the new configuration, you need to run:
  	service apache2 reload
• Reload Apache: sudo service apache2 reload  .

9- Launch the Web Application :
• Go to  /var/www/catalog/ .
• Change the ownership of the project directories: sudo chown -R www-data:www-data catalog/ .
• Restart Apache again: sudo service apache2 restart .
• Open your browser an open http:// 52.31.34.58   .

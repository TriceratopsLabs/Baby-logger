# Baby-logger
This project logs baby functions and related care and displays them on a webpage.
Many pediatricians recommend tracking your baby's feeding patterns, diaper changes, sleep patterns, etc. This is valuable information if there is a problem early on. The doctor can use this information to help with a diagnosis.

Pictures of my build are posted on Imgur: 

[The project was forked from [tommygober's Babby-logger project](https://github.com/tommygober/Baby-logger).]

## Hardware

* 1 - Pi Zero W
* 1 - 8+Gb microSD card
* 1 - MicroUSB Power supply
* 3 - 30mm arcade pushbuttons
* 6 - Jumper wires
* 1 - 



## Updating OS and installing necessary packages

Run
```
sudo apt-get update
sudo apt-get upgrade -y
```

That might take some time.
Install Python 3's PIP tool for installing Python libraries.
```
sudo apt-get install python3-pip
```
Using PIP, install the Python 3 pymysql library.
```
pip install pymysql
```



###  Update Pi to your current time zone

Open the raspi-config Tool: Run the following command in the terminal:
```
sudo raspi-config
```


Navigate to "Localisation Options":

In the raspi-config menu, use the arrow keys to navigate.
Select 5 Localisation Options (or a similar option depending on your version of raspi-config).
Press Enter.


Select "Change Timezone":

From the Localisation Options menu, select Change Timezone.
Press Enter.


Choose Your Geographic Area:

A list of geographic areas (e.g., Europe, America, Asia) will appear.
Use the arrow keys to select your region (e.g., America).
Press Enter.


Choose Your Time Zone:

After selecting the geographic area, you'll see a list of time zones within that area.
Select the appropriate time zone (e.g., New_York for EST in America).
Press Enter.


Finish and Exit:

Once you've set the time zone, you'll be returned to the raspi-config menu.
Select Finish and press Enter.


Verify the correct time zone:
```
timedatectl
```



## Install MariaDB Server

Update your system:
```
sudo apt update && sudo apt upgrade -y
```

Install MariaDB
```
sudo apt install mariadb-server -y
```

Start the MariaDB service:
```
sudo systemctl start mariadb
```

Enable MariaDB to start on boot:
```
sudo systemctl enable mariadb
```

Check the status to confirm it’s running:
```
sudo systemctl status mariadb
```

Run the security script to set a root password and improve security.
When prompted:
    Set a strong root password.
    Remove anonymous users.
    Disallow root login remotely (recommended for security).
    Remove the test database.
    Reload privilege tables.
```
sudo mysql_secure_installation
```

Log in to the MariaDB shell as the root user, use the root password that you just set.
```
sudo mysql -u root -p
```

To verify that MariaDB is working:
```
SHOW DATABASES;
```

### Create a Database and User

Create a new database:
```
CREATE DATABASE babylogger;
```

Create a new user and grant permissions: Replace logger and password with your desired username and password:
```
CREATE USER 'logger'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON babylogger.* TO 'logger'@'localhost';
FLUSH PRIVILEGES;
```






## Install Apache Web Server and PHP

Update your system:
```
sudo apt update && sudo apt upgrade -y
```

Install Apache:
```
sudo apt install apache2 -y
```

Restart Apache to load PHP:
```
sudo systemctl restart apache2
```
Go to your Pi's IP address in a browser on a device connected to the same network as your Pi. You should see the default Apache page if everything is working correctly. 
If you need to find the IP address for the Pi:
```
hostname -I
```

You will want to delete the default index.html file that is generated by Apache setup.
```
sudo rm /var/www/html/index.html
```

Apache's default document root is /var/www/html. You need to place your index.php file in this directory.
```
sudo nano /var/www/html/index.php
```
Paste the contents there and edit the appropriate fields:
* db_host
* db_user
* db_passwd
* database
* GPIO pins
Ctrl-x and then Y to save the file (in nano)/

Set permissions: Ensure the file is readable by the web server:
```
sudo chmod 644 /var/www/html/index.php
```














## Python Script

Now copy the python script to your Pi
```
cd ~
mkdir Logger
cd Logger
```
Use nano or your editor of choice to create the script.
```
nano babylogger.py
```

Paste in the contents of the python script and CTRL-X then Y to exit and save. Don't forget to change GPIO numbers, database name, and password to match what you created. At this point you should also change the button names so that they correspond to the labels whatever you decide those to be.
You can try running the script with
```
sudo python3 babylogger.py
```
Push any of your buttons, if you don't get any error messages - CTRL-C to end the script.

Then open up your MySQL database again and check if anything has been written to the database.
Hopefully it will now show you the date, time, and which button has been pressed. Now you can close the MySQL interface and move on to setting up the webpage.


### Running as Service on boot

Make sure that the script is executable
```
chmod +x /home/pi/Logger/babylogger.py
```
Test the script to confirm that it works
```
/home/pi/Logger/babylogger.py
```

If you want to have the script run automatically whenever your Pi starts up, you can create a Systemd service file.
Create an empty file with nano or your editor of choice:
```
sudo nano /etc/systemd/system/babylogger.service
```
and then paste in the following (changing the username (default is pi) and the path if you've changed any of those)
```
[Unit]
Description=Baby Logger Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/Logger/babylogger.py
Restart=always
User=pi
WorkingDirectory=/home/pi/Logger/
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```
Ctrl-X and then Y to save the file.

Run the following command to reload the systemd manager configuration:
```
sudo systemctl daemon-reload
```
Enable the service so it starts automatically when the system boots:
```
sudo systemctl enable babylogger.service
```
Start the service manually to test it:
```
sudo systemctl start babylogger.service
```
To check on the status of your service:
```
sudo systemctl status babylogger.service
```
You should see something like:
```
● babylogger.service - Baby Logger Service
   Loaded: loaded (/etc/systemd/system/babylogger.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2024-01-01 12:34:56 UTC; 5s ago
 Main PID: 1234 (python3)
    Tasks: 1 (limit: 4915)
   Memory: 10.5M
   CGroup: /system.slice/babylogger.service
           └─1234 /usr/bin/python3 /path/to/babylogger.py
```
This should have you up and running. Reboot and your system should come back up with your ```babylogger``` service running
```
sudo reboot now
```

Step 1: Connect to Your Alibaba Cloud Ubuntu 16.04 Server
Locate the Internet IP address (Public IP address) associated with your Alibaba Cloud ECS Instance. 
If you are running Linux or Mac, use a terminal application to connect to the instance via SSH. If you are on Windows, you can use PuTTy (download here) to connect to your server. You will have to provide the IP address, username and password that you set up when creating your Alibaba Cloud ECS instance to log in via SSH.
There are other ways to connect to your ECS instance as well. Visit the official ECS documentation to learn more. 
Step 2: Change the Hostname on Your Ubuntu 16.04 Server
The hostname is a default identifier when you communicate to a Linux server. It is like a computer name that is associated with your home PC or laptop. Naming your Ubuntu 16.04 server with a descriptive hostname helps you to differentiate your machines especially if you are running a bunch of them.
To begin, ensure your Ubuntu 16.04 system is up-to-date by typing the command below:
$ sudo apt-get update 

And 
$ sudo apt-get upgrade -y
To check your hostname, type the command below on a terminal window:

$ hostname
To change your hostname, first edit the /etc/cloud/cloud.cfg file and find the entry preserve_hostname and change its value from false to true.

$ sudo nano /etc/cloud/cloud.cfg
preserve_hostname true
Press CTRL + X, Y then Enter to save the changes.
Then, edit the /etc/hostname file using a nano editor by typing the command below:

$ sudo nano /etc/hostname
Overwrite the current hostname written at the very top of the file and press CTRL + X, Y then Enter to save the changes.
You will also need to add some entries on the Linux hosts file. Open the file using a text editor:

$ sudo nano /etc/hosts
You will need to add two entries on this file just below the 127.0.0.1 localhost entry. The first entry you are adding uses the loopback interface address 127.0.1.1, please note this is different from the address 127.0.0.1 which have a ‘localhost’ value in the same file.
So assuming your servers public IP address is 111.111.111.111 and your hostname is Miami, your /etc/hosts file should have the below entries at the very top:

127.0.0.1 localhost
127.0.1.1 miami
111.111.111.111 miami

Reboot your Ubuntu 16.04 server for the changes to take effect by typing the command below:
$ sudo reboot

Step 3: Configure Time Zone on Your Ubuntu 16.04 Server
You can check the default date and time zone on your Ubuntu 16.04 server by typing the command below:
$ timedatectl 
You must set the correct time zone especially if you are running cron jobs on your server because they rely heavily on date/time. To change the time zone, use the command below:

$ sudo timedatectl set-timezone 
For instance to set your server time zone to London use the command below

$ sudo timedatectl set-timezone Europe/London
You can run the date command to check if the changes were effected successfully:

$ date
Step 4: Create a Non-Root User with Sudo Privileges on Ubuntu 16.04
Logging on your Ubuntu 16.04 server using a root user can create a lot of problems. For instance, a simple rm command with wrongly typed parameters can wipe your entire production’s server data.
As such, you need to create a non-root user with sudo privileges. You can then temporary elevate privileges by using the sudo command where necessary.
To create the user, use the command below:
$ sudo adduser 
For instance, to add a user identified as james on your server, use the command below:

$ sudo adduser james
Then, add the user to the sudo group by typing the command below:

$ sudo usermod -aG sudo james

Remember to replace james with the correct username.
Step 5: Creating Authentication Key Pair for Logging onto Your Ubuntu 16.04 Server
Logging to your Ubuntu 16.04 server using a private/public key pair is more secure that using a password. In this mode, you keep the private key on your local computer and the public key under the .ssh/authorized_keys file on your Alibaba cloud server.
This technology encrypts data sent from your server via the public key and users can only decrypt it using the correct private key, which is only known to you. Keys used in this manner can’t be guessed even by the most resourceful hackers. You can also add another layer of security by protecting your private key with a passphrase in case it falls to the wrong hands.
You can generate a private/public key pair with a tool like PuTTY key Generator (download here).
Make sure you are logged in as the user who you are generating keys for. Also, the below commands should NOT be run using ‘sudo’.
Copy the public key part to your Ubuntu server using the commands below:
$ mkdir ~/.ssh
Then, use a nano editor to paste your public key on the authorized_keys file by typing:

$ nano ~/.ssh/authorized_keys
Protect the file by typing the commands below

$ chmod 700 -R ~/.ssh && chmod 600 ~/.ssh/authorized_keys
Once the keys are created, you can now login on your Ubuntu 16.04 server using your username and the private key that you have created via a SSH connection.
 

Step 6: Disable Password Authentication
Once you set up the private/public key pair, you should disable password based logins to ensure that only a person with the correct private key can gain access to your Ubuntu 16.04 server.
To do this, edit the SSH configuration file via the command below:
$ sudo nano /etc/ssh/sshd_config
Find the line PasswordAuthentication and change its value from yes to no.

PasswordAuthentication no
Restart the SSH daemon:

$ sudo service ssh restart
Step 7: Disable SSH Root Access on Your Ubuntu 16.04 Server
Once you have created non-root user with sudo privileges and password logins disabled, you can go ahead and disable root login over SSH. This will make sure that no one can login to your Ubuntu server over SSH using the root username.
Any administrative tasks from this point forward will be done by the non-root user with sudo privileges.
To disable root access over SSH, edit the SSH configuration file on more time using a nano editor and look for the directive PermitRootLogin and change its value from yes to no.
$ sudo nano /etc/ssh/sshd_config
Then,

PermitRootLogin no
Restart the SSH daemon by typing the command below for the changes to take effect:

$ sudo service ssh restart

 
Step 8: Install a Firewall on Your Ubuntu 16.04 Server
Ubuntu 16.04 comes with a default interface for interacting with IP tables known as UFW (Uncomplicated Firewall). UFW is a simplified tool which aims towards simplifying the process of setting up IP tables especially for beginners who are new to the Linux environment.
UFW is a right choice for adding another security to your Ubuntu 16.04 server running on Alibaba Cloud.
Although UFW is installed by default, you can use the command below to get it from Ubuntu’s repository if it was uninstalled:
$ sudo apt-get install ufw
Then, type the command below to allow all outgoing calls and deny or incoming calls. 

$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
You can use the UFW command below to allow traffic to a particular port or service:

$ sudo ufw allow 
To avoid completely locking yourself from your Ubuntu server, the first port/service that you should allow on UFW is port 22 which listens for SSH connections. 
To do this, type the command below to add the rule:

$ sudo ufw allow 22
Or

$ sudo ufw allow ssh
Also if you are running a web server, you should enable the http and https port:

$ sudo ufw allow http
$ sudo ufw allow https
Once you have whitelisted the services, run the command below to start UFW

$ sudo ufw enable
You can delete any rule that you have created by first checking its number and then deleting it via the commands below:

$ sudo ufw status numbered
Then

$ sudo ufw delete 
Where is the value that you obtained above from the list of rules available.
Make sure ufw is enabled before checking the list of rules.
You can disable UFW at any time by typing the command below:

$ sudo ufw disable
Or just reset all rules by typing:

$ sudo ufw reset
Step 9: Install Fail2Ban on Your Ubuntu 16.04 Server
Fail2Ban is a tool that adds another layer of security to your Ubuntu 16.04 server by utilizing IP tables. It simply bans users trying to access your Ubuntu server based on the number of failed logged in attempts.
You can Install Fail2Ban by typing the command below.
$ sudo apt-get install fail2ban
You can use your server with the default settings for Fail2Ban but when the need arises, you can edit the configuration file to make changes. All Fail2Ban configuration files are located at /etc/fail2ban/ directory
By default .conf files are read first followed by .local files. So if you want to override settings, you should make changes to .local files and leave .conf files intact.
For instance, you can create your own copy of jail.conf file and create a local file for editing using the commands below:

$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
You can then change any Fail2Ban settings by editing the new file with the command below:

$ sudo nano /etc/fail2ban/jail.local
In most cases, you will be setting the ban time, find time and max retries for SSH connections. This will all depend on the level of security that you need on your Alibaba Ubuntu 16.04 server.
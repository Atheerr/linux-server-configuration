# Linux Server Configuration
> By Dylan Blur

![Part of the Udacity Front-End Web Development Nanodegree](https://img.shields.io/badge/Udacity-Front--End%20Web%20Developer%20Nanodegree-02b3e4.svg)

Project 6 : Linux server configuration

# Server Details
* URL: [x.x.x.x](https://google.com) or [x.amazon.com](https://amazone.com)

* SSH port: changed from ~~**22**~~ to **2200**

* IP adress: `35.176.97.66`

# Configuration
## Instance on [AWS LightSail](https://lightsail.aws.amazon.com)

1. Sign in to Amazon Lightsail using an Amazon Web Services account

2. Follow the 'Create an instance' link

3. Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

4. Choose a payment plan

5. Give the instance a unique name and click 'Create'

6. Wait for the instance to start up

## Connect to the instance on a local machine
>Note: Amazon Lightsail provides a broswer-based connection method and a terminal, but this will no longer work once the SSH port is changed from 22 to 2200. The following steps outline how to connect to the instance via a terminal on a local machine.

1. Download the instance's private key by navigating to the Amazon Lightsail 'Account page'

2. Click on '**Download default key**'

3. A file called `LightsailDefaultPrivateKey.pem` will be downloaded; open it in a text editor

4. Copy the text and put it in a file called lightrail_key.rsa in the local ~/.ssh/ directory

5. Run `chmod 644 ~/.ssh/lightrail_key.rsa` to give permissions to access the file

6. Log in with the following command: `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@aa.aa.aa.aa`, where aa.aa.aa.aa is the public IP address of the instance.

## Update and Upgrade existing packages
`apt-get update` - to update the package indexes

`apt-get upgrade` - to actually upgrade the installed packages

If at login the message *** System restart required *** is display, run `reboot` to restart the machine.

## Configuring UFW (Uncomplicated Firewall)
>Note: The following commands have to be run as administrator so as an alternative to typing `sudo` before every command you can first of all run `sudo su`

0. Run `sudo ufw status` to check the status prior to the configuration

1. Changing the SSH port from 22 to 2200 (open up the /etc/ssh/sshd_config file `sudo nano /etc/ssh/sshd_config`, **change the port number on line 5 to 2200**, then **restart SSH** by running `sudo service ssh restart`

1. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running `sudo ufw status`

2. Run `sudo ufw default deny incoming` this is to set the ufw firewall to block everything coming in

3. Run `sudo ufw default allow outgoing` this is to to set the ufw firewall to allow everything outgoing

4. Run `sudo ufw allow ssh` to set the ufw firewall to allow SSH

5. Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port 2200 so that SSH will work

6. Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server

7. Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP (123/udp is actually the port for NTP)

8. Run `sudo ufw deny 22` to deny port 22 (deny this port since it will be configured to 2200)

9. Run `sudo ufw enable` to enable the ufw firewall

10. Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

```
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
```
12. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed; make sure to deny the default port 22)

13. Now, to login again, open up the Terminal and run:

`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance

>Note: Connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port 22, which is now denied.

## Create a new user named `grader`

1. Run `sudo adduser grader`

2. Enter in a new UNIX password (twice) when prompted
(`pass:12345678`)

3. Fill out information for the new grader user

4. To switch to the grader user, run `su - grader`, and enter the password to test for accessibility

## Give grader user sudo permissions

0. Go back to the main user by typing `exit`

1. Run `sudo visudo`

2. Search for a line that looks like this: `root ALL=(ALL:ALL) ALL`

3. Add the following line below this one:`grader ALL=(ALL:ALL) ALL`

4. Save and close the visudo file

5. To verify that grader has sudo permissions, su as grader (run `su - grader`), enter the password, and run `sudo -l`; 

## Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine

1. Choose a file name for the key pair (such as grader_key)

1. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

1. Log in to the virtual machine

1. Switch to `grader`'s home directory, and create a new directory called `.ssh` (run `mkdir .ssh`)

1. Run `touch .ssh/authorized_keys`

1. On the local machine, run `cat ~/.ssh/insert-name-of-file.pub` 

1. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

1. Run `chmod 700 .ssh` on the virtual machine

1. Run `chmod 644 .ssh/authorized_keys` on the virtual machine

1. Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)

1. Log in as the grader using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`

    `pass:12345678`

> Note that a pop-up window will ask for `grader`'s password.

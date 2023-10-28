# Automating Load Balancer Cofiguration With Shell Scripting
The following steps are taken to automate load balancer configuration with shell scripting.
### Step 1: Provision an EC2 Instance for the 1st Web Server
Use the following parameters when configuring the EC2 Instance:
* Name of Instance: Web Server 1
* AMI: Ubuntu
* Key Pair Name: web11
* Key Pair Type: RSA
* Private Key File Format: .pem
* Security Group: Apache Web Server Security Group
* Inbound Rule: Allow Traffic From Anywhere On Port 8000 and Port 22.

### Step 2: Connect to the Webserver via the terminla using SSH
* Open terminal on your computer.
* Run the following command to go to the directory (i.e. Downloads) where the `.pem` key pair file was downloaded.

```sh
cd Downloads
```

* SSH into the Instance using the command shown below:

* Create and open a file using the command shown below:

```sh
sudo vi install.sh
```

* Paste the script shown below into the file:

```sh
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 18.234.230.94
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=18.234.230.94

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my 1st Web Server EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2
```
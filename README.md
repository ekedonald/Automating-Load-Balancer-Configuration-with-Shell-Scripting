# Automating Load Balancer Configuration With Shell Scripting
The following steps are taken to automate Load Balancer configuration to distribute traffic between 2 Web Servers with shell scripting:

### Step 1: Provision an EC2 Instance for the 1st Web Server
Use the following parameters when configuring the EC2 Instance:
* Name of Instance: Web Server 1
* AMI: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type
* New Key Pair Name: web11
* Key Pair Type: RSA
* Private Key File Format: .pem
* New Security Group: Apache Web Server Security Group
* Inbound Rule: Allow Traffic From Anywhere On Port 8000 and Port 22.

### Step 2: Connect to the 1st Web Server via the terminal using SSH
* Open the terminal on your computer.
* Run the following command to go to the directory (i.e. Downloads) where the `.pem` key pair file was downloaded.

```sh
cd Downloads
```

* SSH into the Instance using the command shown below:

### Step 3: Deployment of the 1st Web Server
* Create and open a file using the command shown below:

```sh
sudo vi install.sh
```

* Paste the script shown below into the file then save and exit the file:

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

* Assign executable permissions on the file using the command shown below:

```sh
sudo chmod +x install.sh
```

* Run the shell script using the command shown below:

```sh
./install.sh
```

* Go to your web browser and paste the following URL to verify the setup:

```sh
http://public_ip_address_of_web_server_2:8000
```

### Step 4: Provision an EC2 Instance for the 2nd Web Server
Use the following parameters when configuring the EC2 Instance:
* Name of Instance: Web Server 2
* AMI: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type
* Key Pair Name: web11
* Security Group: Apache Server Security Group

### Step 5: Connect to the 2nd Web Server via the terminal using SSH
* Open the terminal on your computer.
* Run the following command to go to the directory (i.e. Downloads) where the `.pem` key pair file was downloaded.

```sh
cd Downloads
```

* SSH into the Instance using the command shown below:

### Step 6: Deployment of the 2nd Web Server
* Create and open a file using the command shown below:

```sh
sudo vi install.sh
```
* Paste the script shown below into the file then save and exit the file:

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

PUBLIC_IP=54.175.35.61

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
            <h1>Welcome to my 2nd Web Server EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2
```

* Assign executable permissions on the file using the command shown below:

```sh
sudo chmod +x install.sh
```

* Run the shell script using the command shown below:

```sh
./install.sh
```

* Go to your web browser and paste the following URL to verify the setup:

```sh
http://public_ip_address_of_web_server_2:8000
```

### Step 8: Provision an EC2 Instance for the Load Balancer
Use the following parameters when configuring the EC2 Instance:
* Name of Instance: Load Balancer
* AMI: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type
* New Key Pair Name: web11
* New Security Group: launch-wizard-19
* Inbound Rule: Allow Traffic From Anywhere On Port 80 and Port 22.

### Step 9: Connect to the Load Balancer via the terminal using SSH
* Open the terminal on your computer.
* Run the following command to go to the directory (i.e. Downloads) where the `.pem` key pair file was downloaded.

```sh
cd Downloads
```

* SSH into the Instance using the command shown below:

### Step 10: Deploying and Configuring Nginx Load Balancer
* Create and open a file using the command shown below:

```sh
sudo vi nginx.sh
```
* Paste the script shown below into the file then save and exit the file:

```sh

#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 54.90.83.129 18.234.230.94:8000 54.175.35.61:8000
############################################################################################################# 

PUBLIC_IP=54.90.83.129
firstWebserver=18.234.230.94:8000
secondWebserver=54.175.35.61:8000

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserver 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx
```

* Assign executable permissions on the file using the command shown below:

```sh
sudo chmod +x install.sh
```

* Run the shell script using the command shown below:

```sh
./nginx.sh
```

* Go to your web browser and paste the following URL to verify the setup:

```sh
http://public_ip_address_of_load_balancer:80
```

_Each time you refresh the page, you will see the web page of the 1st and 2nd Web Server._
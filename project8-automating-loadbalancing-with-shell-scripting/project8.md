# Automating Load Balancer configuration with shell scripting By Itoro Ekpo

![cover](./img/cover.png)

In the implementation of our Nginx load balancer, we deployed two backend servers with a load balancer distributing traffic across the webservers. We did this by typing commands right into our terminal.

However, in this project, we will be automating the entire process. We will do this by writing a shell script that when ran, all that we configured manually will be performed automatically.

As DevOps Engineers, automation is at the heart of the work we do. Automation helps us speed the development of services and reduce the chance of making errors in our day to day activities.

This project will give a great introduction into _Automation._

## Deploying and Configuring the Webservers

The process of deploying our webservers has been codified in the shell script below.

```
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

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
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2

```

**Step 1**: Provision an EC2 instance running ubuntu 22.04.

![EC2](./img/1a.instances_EC2.png)

**Step 2**: Open port 8000 to allow traffic from anywhere using the security group.

![port_8000](./img/1b.sec_gr_8000.png)

**Step 3**: Connect to the webserver via a terminal using SSH client.

![ssh](./img/3a.ssh.png)

**Step 4**: Open a file, paste the script above and close the file using the command `sudo vi install.sh`

![innstall](./img/4a.prep_file.png)

**Step 5**: Change the permission on the file to make it executable using the command `sudo chmod +x install.sh`

![chmod](./img/5a.chmod.png)

**Step 6**: Run the script usinfg the command `./install.sh <PUBLIC_IP>`

![run](./img/6.run_script.png)

_script run successfully and both webservers are now active running_

Both apache webservers are now serving webpages on the browser displaying their public IP addresses

![apache1](./img/6a.apache1.png)

![apache2](./img/6b.apache2.png)

## Deployment of Nginx as a Load Balancer using Shell Script
Now we will automate the deployment of Nginx as a load balancer using a shell script. We will provision another EC2 instance running ubuntu 22.04, open port 80 to anywhere using security group and connect to the load balancer via the terminal.

![nginx_inst](./img/7a.nginx_inst.png)

_port 80 opened in security group_

![port80](./img/7b.port_80.png)

_EC2 instance for load balancer connected via ssh_

![ssh_nginx](./img/7c.ssh_nginx.png)

_All the steps followed to implement load balancer with Nginx has been codified in the script below. Read the instructions carefully in the script to learn how to use it._

```
#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

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
            server  "${firstWebserver}"; # public IP and port for webserser 1
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

**Step 1**: On your terminal, open a file `nginx.sh` using the command `sudo vi nginx.sh`

**Step 2**: Copy and paste the script inside the file

![prep_file2](./img/7d.prep_file.png)

**Step 3**: Save and close the file. Type `"esc"` then `":wqa!"`

**Step 4**: Change the permission of the file to make it an executable file using the command `sudo chmod +x nginx.sh`

![nchmod](./img/7e.nginx_chmod.png)

**Step 5**: Run the script with the command `./nginx.sh PUBLIC_IP Webserver-1 Webserver-2`

![run_lb](./img/7f.run_lb.png)

## Verifying the setup
_You can see from the below screenshot the Nginx webserver configured as a load balancer with public IP `54.163.209.112` is serving the webpage of Apache webserver 1 displaying it's public IP address `3.88.47.22`_

![WS1](./img/8a.webserver1.png)

_And when refreshed the Nginx load balancer serves the webpage of Apache webserver 2 displaying it's public IP address `54.160.135.22`

![WS2](./img/8b.webserver2.png)








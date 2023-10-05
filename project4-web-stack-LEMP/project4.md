# Web Stack Implementation (LEMP Stack) Project by Itoro Ekpo

![cover](./img/cover.png
)
> The project LEMP Stack covers similar concepts as that of LAMP and helps to cement my skills of deploying web solutions using LA(E)MP stacks. In this project I will be implementing a similar stack, but with an alternative Web Server-_NGINX_, which is also very popular and widely used by many websites on the internet.

**LEMP** deploys _Linux_, _NGINX_, _MySQL_ and _PHP_ to create dynamic and high-performing websites. We will dive into the architecture of the LEMP stack, understanding how Linux provides a solid foundation , Nginx serves as a powerful webserver, MySQL for the database and PHP empowering server-side functionality.

## Step 0 - Preparing Prerequisites
1. I setup a second [AWS](https://aws.amazon.com/) account and provisioned a virtual server instance with Ubuntu Server OS using _Elastic Compute Cloud (EC2)_

![createinstance](./img/1.instance-create.png)

2. This time around I will be connecting to my EC2 instance using _ssh_ with _Git Bash_. You can download Git [**_here_**](https://git-scm.com) 

>`ssh -i "itoro-web-server.pem" ubuntu@ec2-3.86.35.129.compute-1.amazonaws.com`

![instance connect](./img/2.instance-connect.png)

## Step 1 Installing the Nginx Web Server
I will be employing Nginx, a high performing web server. I will use `apt` package manager to install it but first we must update our server's package index.

> `sudo apt update`
> `sudo apt install nginx`

Verify that nginx was successfully installed and is running as a service in Ubuntu.

> `sudo systemctl status nginx`

![nginx](./img/3.nginx-running.PNG)

Verify TCP port 80 is open

- `curl http://localhost:80`
- `curl http://127.0.0.1:80`

![curl](./img/4.curl.png)

Now we can test that our Nginx server can respond to requests from the internet by entering `http://<public-ip-address>:80` into a web browser.

You can retrieve your public-ip-address from AWS or you can run the command below
`curl -s http://169.254.169.254/latest/meta-data/public-ipv4`

![publicip](./img/6.public-ip.png)

> My public IP address is `3.86.35.129` so `http://3.86.35.129:80`

You should have the below.

![nginxweb](./img/5.nginx-web.png)

## Step 2 Installing MySQL
Now that a webserver is up and running, I need to install a [Database Management System(DBMS)](https://en.wikipedia.org/wiki/Database#Database_management_system) to store and manage data for my site in a [Relational Database](https://cloud.google.com/learn/what-is-a-relational-database#:~:text=A%20relational%20database%20is%20a,structures%20relate%20to%20each%20other.). MySQL is a popular [relational database management system](https://www.techtarget.com/searchdatamanagement/definition/RDBMS-relational-database-management-system) used within PHP environments. I will be installing [_MySQL_](https://www.mysql.com/) using the steps below:

1. Install MySql using command `sudo apt install mysql-server`
2. When installation is complete, login to the MySQL console by typing `sudo mysql`

![mysql](./img/7.mysql.png)

3. set password for root user to `PassWord.1`
- `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

![](./img/8.sql-chnge-passwd.png)

4. Exit the MySQL shell with the command `exit`
5. Start the interactive script by running `sudo mysql_secure_installation`
> VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

> Press y|Y for Yes, any other key for No:

> There are three levels of password validation policy:

> LOW    Length >= 8
> MEDIUM Length >= 8, numeric, mixed case, and special characters
> STRONG Length >= 8, numeric, mixed case, special characters and dictionary file

> Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1

6. Test log in to the MySQL console by typing `sudo mysql -p`
7. Exit the MySQL console by typing `exit`

![mysql-login](./img/9.mysql-exit.png)

MySQL server is now installed and secured. Next step is to install our final component. PHP.

## Step 3 - Installing PHP
Now I have Nginx installed to serve my content and MySQL installed to store and manage my data. [PHP](https://www.php.net/) is the final component of the LEMP technology stack. It will process code and generate dynamic content to the end user. 

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites but it requires additional configuration.

I will need to install `php-fpm` which stands for PHP fast CGI process manager and tell Nginx to pass PHP requests to this software for processing. In addition, I will need `php-mysql`, a PHP module that allows PHP to communicate with MySQL-based databases.

1. Install all 3 packages at once with the command `sudo apt install php-fpm php-mysql`
2. Once installation is complete you can run `php -v` to confirm the php version.

![phpversion](./img/10.php-version.png)

> **LEMP STACK SETUP IS NOW COMPLETE**
> - Linux (Ubuntu)
> - Nginx web server
> - MySQL
> - PHP

## Step 4 Configuring Nginx to use PHP processor



# Implementing Loadbalancing with NGINX

> `Load balancing is the method of distributing network traffic equally across a pool of resources that support an application. Modern applications must process millions of users simultaneously and return the correct text, videos, images, and other data to each user in a fast and reliable manner. To handle such high volumes of traffic, most applications have many resource servers with duplicate data between them. A load balancer is a device that sits between the user and the server group and acts as an invisible facilitator, ensuring that all resource servers are used equally.`

![Nginx](./img/1a.nginx_intro2.png)

## An introduction to load balancing and Nginx

Load balancing is like having a team of helpers working together to make sure a big job gets done smoothly and
efficiently. Imagine you have a lot of heavy boxes to carry, but you can't carry them all by yourself because they are too heavy.

Load balancing is when you call your friends to help you carry the boxes. Each friend takes some of the boxes and carries them to the right place. This way, the work gets done much faster because everyone is working together.

In computer terms, load balancing means distributing the work or tasks among several computers or servers so
that no one computer gets overloaded with too much work. This helps to keep everything running smoothly and
ensures that websites and apps work quickly and don't get too slow. It's like teamwork for computers!

Lets say you have a set of webservers serving your website. In other to distribute the traffic evenly between the webservers, a load balancer is deployed. The load balancer stands in front of the webservers, all
traffic gets to it first, it then distributes the traffic across the set of webservers. This is to ensure no webserver gets over worked, consequently improving system performance.

Nginx is a versatile software, it can act like a webserver, reverse proxy, and a load balancer etc. All that is needed is to configure it properly to server your use case.

In this project we will be working you through how to configure Nginx as a load balancer.

## Setting Up a Basic Load Balancer
1. **Step 1**: Provision two EC2 instances running ubuntu 22.04 operating system and install apache webserver in them. We will open port 8000 to allow traffic from anywhere.

    ![EC2](./img/1a.apache_inst.png)
    _Two EC2 Instances created running ubuntu 22.04 shown above_

    ![apache1](./img/1a.apache1.png)
    _Screenshot of Apache server 1 above showing IP details_

    ![apache2](./img/1a.apache2.png)
    _Screenshot of Apache server 2 above showing IP details_

2. **Step 2**: Open port 8000. We will be running our webservers on port 8000 while the load balancer runs on port 80. We need to open port 8000 to allow traffic from anywhere by adding a rule to the security group of each of our webservers.

    ![port8000](./img/2.open_8000.png)

3. **Step 3**: Install Apache on both servers.

    * SSH into both instances.

        ![ssh_apache1](./img/3a.ssh_apache1.png)

        ![ssh_apache2](./img/3b.ssh_apache2.png)

        Apache installed on both instances as shown in below screenshots.

        ![A1](./img/3c.apache1_installed.png)

        ![A2](./img/3d.apache2_installed.png)

    * Verify apache is running on both instances using `sudo systemctl status apache2`

        ![A1_run](./img/3e.apache1_running.png)

        ![A2_run](./img/3f.apache2_running.png)

4. **Step 4**: Configure Apache to serve a page showing its public IP address.
    * Configure Apache to serve content on port 8000

        ![add_port_8000](./img/4a.port_8000_add.png)

    * Open the file /etc/apache2/sites-available/000-default.conf and change port 80 on the virtualhost to 8000.

        `sudo vi /etc/apache2/sites-available/000-default.conf`

        ![virtualhost](./img/4b.virtual_host.png)

    * Save and Close the file by first pressing `esc` key then the command `:wq!`

    * Restart apache to load the new configuration using the command `sudo systemctl restart apache2`

        ![apache_rstrt](./img/4c.apache_restart.png)

    * Create a new html file `sudo vi index.html`

    * Copy the code below and paste in the new html files. Replace `YOUR_PUBLIC_IP` placeholders with the respective public IP's of the two apache servers.

        ```
        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>
        ```

        As shown below:
        ![html](./img/4d.apache_html_files.png)

    * Change ownership of the `index.html` file with the command `sudo chown www-data:www-data ./index.html`

        ![chown](./img/4e.chown.png)

    * Overide the default html file of the Apache Webserver with the command below  
        `sudo cp -f ./index.html /var/www/html/index.html`

    * Restart the webservers to load the new configuration using the command below  
        `sudo systemctl restart apache2`

    * You should find the page in your browser looking like the below screenshots:

        ![browser1](./img/4f.browser1.png)

        ![browser2](./img/4g.browser2.png)

5. **Step 5**: Configure Nginx as a Load Balancer

    * Provision a new EC2 instance running ubuntu 22.04. Ensure port 80 is opened to accept traffic from anywhere
        ![nginx_inst](./img/5a.nginx_inst.png)

        ![port80](./img/5a.port80.png)

    * SSH into the instance as shown below
        ![ssh_nginx](./img/5b.ssh_nginx.png)

    * Install Nginx on the instance `sudo apt update -y && sudo apt install nginx -y`

    * Verify that Nginx is installed `sudo systemctl status nginx`

        ![nginx_running](./img/5c.nginx_running.png)

    * Open Nginx configuration file with the command `sudo vi /etc/nginx/conf.d/loadbalancer.conf`

    * Paste the below configuration to configure Nginx to act like a load balancer.

        ```
                
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server 3.81.33.174:8000; # public IP and port for apache webserver 1
            server 3.80.251.101:8000; # public IP and port for apache webserver 2

        }

        server {
            listen 80;
            server_name 54.86.25.184; # provide your load balancers public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }

        ```
        ![load_bal](./img/5d.load_bal.png)

        **upstream backend_servers** defines a group of backend servers. The **server** lines inside the **upstream** block list the addresses and ports of your backend servers. **proxy_pass** inside the **location** block sets up the load balancing, passing the requests to the backend servers. The **proxy_set_header** lines pass necessary headers to the backend servers to correctly handle the requests.

    * Test your configuration with the command `sudo nginx -t`
        ![nginx_test](./img/5e.nginx_test.png)

    * If there are no errors as in the screenshot above then restart Nginx to load the new configuration with the command `sudo systemctl restart nginx`

    * Paste the public IP address of Nginx load balancer and you should see the same webpages served by the apache webservers. In the screenshot below you can see the load balancer serves Apache-Server-1's public IP address.

        ![LB](./img/5f.load_balancing_running.png)

    * If you refresh the webpage you will notice that the load balancer serves Apache-Server-2's public IP address as well.

        ![LB2](./img/5g.load_balancer2.png)


> **Implementing Load Balancing with Nginx project completed**

## Notes

### Load Balancing Algorithms

Load balancing algorithms are techniques used to distribute incoming network traffic or workload across multiple servers, ensuring efficient utilization of resources and improving overall system performance, reliability, and availability. Here are some common load balancer algorithms:

1. Round Robin: This algorithm distributes requests sequentially to each server in the pool. It is simple to implement and ensures an even distribution of traffic. It works well when all servers have similar capabilities and resources.

2. Least Connections: This algorithm routes new requests to the server with the least number of active connections. It is effective when servers have varying capacities or workloads, as it helps distribute traffic to the least busy server.

3. Weighted Round Robin: Similar to the Round Robin algorithm, but servers are assigned different weights based on their capabilities. Servers with higher capacities receive more requests. This approach is useful when servers have varying capacities or performance levels.

4. Weighted Least Connections: Similar to the Least Connections algorithm, but servers are assigned different weights based on their capabilities. Servers with higher capacities receive more connections. This approach balances traffic based on server capacities.

5. IP Hash: This algorithm uses a hash function based on the client's IP address to consistently map the client to a specific server. This ensures that the same client always reaches the same server, which can be helpful for maintaining session data or stateful connections.

### SSL Termination and HTTPS Load Balancing

In this module we will be configuring TLS/SSL on our deployed website. But before doing so, let us take a brief moment to understand the purpose of TLS certificate, how they work and the technology behinde it.

### Encryption

Encryption is at the heart of TLS/SSL. Encryption is the process of converting plain, readable data (referred to as plaintext) into an unreadable format called ciphertext. The purpose of encryption is to ensure data confidentiality and protect sensitive information from unauthorized access or interception.

In encryption, an algorithm (known as a cryptographic algorithm) and a secret key are used to transform the plaintext into ciphertext. Only those who possess the correct key can decrypt the ciphertext and convert it back into its original plaintext form.

### Types of Encryption
Encryption can be classified into several types based on various criteria, such as the encryption process, the key used, and the level of security provided. Here are some common types of encryption:

1. Symmetric Encryption
Symmetric encryption, the same key is used for both encryption and decryption processes. Both the sender and the receiver must possess the shared secret key. While symmetric encryption is generally faster than other methods, distributing and managing the secret key securely among all parties can be challenging. Examples of symmetric encryption algorithms include Advanced Encryption Standard (AES) and Data Encryption Standard (DES)

2. Asymmetric Encryption
Asymmetric Encryption (Public-Key Encryption): Asymmetric encryption uses two distinct keys, a public key and a private key. The public key is used for encryption, while the private key is used for decryption. Anyone can use the recipient's public key to encrypt data, but only the recipient with the matching private key can decrypt and read the data. This method eliminates the need for secure key distribution but is computationally more intensive than symmetric encryption. Popular asymmetric encryption algorithms include RSA(Rivest-Shamir-Adleman) and Elliptic Curve Cryptography (ECC).

3. Hybrid Encryption
Hybrid encryption combines both symmetric and asymmetric encryption. In this approach, the sender generates a random symmetric key for each message and encrypts the actual data using this symmetric key (which is efficient for large amounts of data). Then, the sender encrypts the symmetric key using the recipient's public key and sends both the encrypted data and the encrypted symmetric key to the recipient. The recipient can decrypt the symmetric key using their private key and then use the symmetric key to decrypt the actual data. This method leverages the advantages of both symmetric and asymmetric encryption.

### The Purpose of TLS/SSL Certificate
The main purpose of TLS/SSL certificates is to encrypt the data transmitted between the web server and the client. This ensures that sensitive information, such as login credentials, personal data, or credit card details, remains confidential and protected from eavesdropping.

A secondary benefit is to establish trust between webservers and their client. Before data is transmitted between client and sever, the server needs to go through the process of authentication(server proves that its identity is genuine) by presenting its certificate to the cleint which is validate by a trusted Certificate Authority CA.

There are terms such as CA, certificate that you may not understand at the moment. But not to worry, all these will be explained in the next section.

### How TLS/SSL Work
* TLS/SSL works with hybrid encryption. This means that both symmetric and Asymmetric encryption is used in TLS/SSL.

* Before data is transmitted between client and server, the process of TLS Handshake is carried out.

* During TLS handshake, the server shares with the client its digital certificate. The digital certificate contains the public key of the server.

* The client(browser) verifies the validity of the servers public key using the public key of the Certificate Authority CA. If valid, the client encrypts it encryption key using the server's public key. This encrypted key is then sent to the server.

* The client generates its encryption key using symmetric encryption. The implication is that its uses the same key for both encryption and decryption hence the need to encrypt its key using the server public key.

* Since the server is the only entity in possession of its private key, It is able to decrypt the clients encrypted key. 

* After the handshake process is completed, the client encrypts every data it sends to the server. The server is then able to decrypt the data with the client's encryption key.

* This ensures that only the server is able to make sense of the data shared by the client.

### Advanced Load Balancing Features

Advanced features of load balancing enhance the capabilities and efficiency of load balancers in handling complex scenarios and optimizing application performance. Here are some key advanced features:

1. SSL Offloading/Termination: Load balancers can handle Secure Socket Layer (SSL) encryption and decryption on behalf of backend servers. This offloading reduces the computational burden on application servers, enabling them to focus on processing application logic instead of handling SSL/TLS encryption.

2. Session Persistence/Sticky Sessions: Some applications require that a client's requests consistently go to the same backend server to maintain session state. Load balancers can use techniques like cookie-based or IP-based persistence to ensure requests from a specific client are directed to the same server throughout the session.

3. Health Checks and Automatic Server Failover: Load balancers can perform periodic health checks on backend servers to monitor their availability and performance. If a server becomes unresponsive or unhealthy, the load balancer can automatically remove it from the server pool, rerouting traffic to healthy servers, thus ensuring high availability.

4. Global Server Load Balancing (GSLB): GSLB enables load balancing across multiple data centers or geographically distributed server clusters. It helps direct traffic to the nearest or most available data center, optimizing user experience and providing disaster recovery capabilities.

5. Application-Layer Load Balancing: Advanced load balancers can make routing decisions based on application-specific attributes beyond traditional IP and TCP/UDP information. For example, they can inspect HTTP headers or application-layer protocols to direct traffic based on content, URL, or user agent.

6. Dynamic Load Balancing: Some load balancers use real-time analytics and machine learning to dynamically adjust server weights or routing decisions based on current server performance, network conditions, and application demands. This adaptability ensures efficient resource utilization.

7. Anycast Load Balancing: Anycast allows multiple load balancer instances to share the same IP address across different locations. When a client sends a request, it is automatically routed to the nearest load balancer instance, reducing latency and improving performance.

8. Rate Limiting and Traffic Shaping: Load balancers can enforce rate limits on incoming requests from clients, preventing denial-of-service attacks and controlling resource utilization. They can also shape traffic, prioritizing certain types of requests over others based on defined policies.

9. Web Application Firewall (WAF) Integration: Some load balancers offer integrated WAF functionality to protect web applications from common security threats like SQL injection, cross-site scripting (XSS), and other vulnerabilities.

These advanced features make load balancers powerful tools for optimizing application performance, ensuring high availability, and protecting applications from various threats and failures. They are essential components in modern, scalable, and robust IT infrastructures.

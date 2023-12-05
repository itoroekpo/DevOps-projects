# DevOps Tooling Website Solution Project by Itoro Ekpo

In this project, I will implement a tooling website solution making access to DevOps tools within the corporate infrastructure easily accessible. My solution will consist of the following components.

1. Infrastructure: [AWS](https://aws.amazon.com/)
2. Webserver Linux: [Red Hat](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) Enterprise Linux 8
3. Database Server: [Ubuntu](https://ubuntu.com/) 20.04 + [MySQL](https://www.mysql.com/)
4. Storage Server: [Red Hat](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) Enterprise Linux 8 + NFS Server
5. Programming Language: [PHP](https://www.php.net/)
6. Code Repository: [GitHub](https://github.com/)

Implement a business website using NFS for the backend file storage
---

### **Step 1**: Prepare NFS Server
1. Spin up a new EC2 instance with RHEL Linux 8 Operating System.

    ![nfsinst](./img/1.nfs_inst.png)

2. Based on my LVM experience from [Project-9](https://github.com/itoroekpo/DevOps-projects/tree/ab87b204e5ca0d9969c72530b236448ce1346c12/project9-wordpress-website-with-LVM-SM), Configure LVM on the Server.

    * Instead of formatting the disks as `ext4` you will have to format them as `xfs`
    * Ensure there are 3 logical volumes. `lv-opt`, `lv-apps` and `lv-logs`.
    * Create mount points on `/mnt` directory for the logical volumes as follows: 
        - Mount `lv-apps` on `/mnt/apps` - To be used by the webservers
        - Mount `lv-logs` on `/mnt/logs` - To be used by webserver logs
        - Mount `lv-opt` on `/mnt/opt` - To be used by Jenkins server

        `sudo gdisk /dev/xvd[fgh]`  

        Install `LVM2` package using the command `sudo yum install lvm2`. 
        
        Run `sudo lvmdiskscan` command to check for available partitions.


        Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

        ```
        sudo pvcreate /dev/xvdf1
        sudo pvcreate /dev/xvdg1
        sudo pvcreate /dev/xvdh1
        ```

        Verify that your physical volume has been created successfully by running `sudo pvs`

        ![NFS1](./img/NFS1.png)

        Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**

        > `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

        Verify that the VG created successfully by running the command `sudo vgs`

        ![vgs](./img/NFSvgs.png)

        Use `lvcreate` utility to create 3 logical volumes. **lv-apps**, **lv-logs** and **lv-opt** which will be used to store website data, log data and jenkins data respectively.

        > `sudo lvcreate -n lv-apps -L 9G webdata-vg`  
        > `sudo lvcreate -n lv-logs -L 9G webdata-vg`  
        > `sudo lvcreate -n lv-opt -L 9G webdata-vg`

        Verify that your Logical Volume has been created successfully by running `sudo lvs`

        ![lvs](./img/NFSlvs.png)

        **Step 13**: Verify the entire setup  
        > `sudo vgdisplay -v` #view complete setup - VG, PV, and LV  
        > `sudo lsblk`

        ![nfs_lsblk](./img/NFS-lsblk.png)

        Use `mkfs.xfs` to format the logical volumes with **xfs** filesystem

        > `sudo mkfs -t xfs /dev/webdata-vg/lv-apps`  
        > `sudo mkfs -t xfs /dev/webdata-vg/lv-logs`  
        > `sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

        Create directories to mount the logical volumes

        > `sudo mkdir -p /mnt/apps`  
        > `sudo mkdir -p /mnt/logs`  
        > `sudo mkdir -p /mnt/opt`

        Mount the logical drives unto the respective directories.

        > `sudo mount /dev/webdata-vg/lv-apps /mnt/apps/`  
        > `sudo mount /dev/webdata-vg/lv-opt /mnt/opt/`  
        > `sudo mount /dev/webdata-vg/lv-logs /mnt/logs/`

        ![nfs_xfs](./img/1b.nfs_xfs.png)

        ![nfs_comp](./img/1a.nfs_complete.png)

        **Update `/etc/fstab`**
        ---

        Update `/etc/fstab` in this format using your own UUID and remember to remove the leading and ending quotes.

        > `blkid`

        ![nfsblkid](./img/NFS_blkid.png)

        > `sudo vi /etc/fstab`

        ```
        # mounts for NFS Server
        UUID=feb2bb36-9b2d-43a7-b219-708eb1b7479d /mnt/logs xfs defaults 0 0
        UUID=b1c4c331-db12-4639-bb4d-796d9d9bbf7d /mnt/opt xfs defaults 0 0
        UUID=9b5bdef8-fca3-49a8-ba2b-5f77b065a2b1 /mnt/apps xfs defaults 0 0
        ```

        ![nfs_fstab](./img/NFS_fstab.png)

3. Install NFS server, configure it to start on reboot and make sure it is up and running.

    ```
    sudo yum -y update
    sudo yum install nfs-utils -y
    sudo systemctl start nfs-server.service
    sudo systemctl enable nfs-server.service
    sudo systemctl status nfs-server.service
    ```

    ![nfs_serv](./img/2.nfs_service.png)

4. Export the mounts for webservers `subnet cidr` to connect as clients. For simplicity I will be installing all 3 webservers inside the same subnet but in production setup you would probably want to seperate each tier inside it's own subnet for higher level of security. To check your `subnet cidr`, open your EC2 details in **AWS** web console and locate the `Networking` tab and open a subnet link as shown below: 

    _Ensure to set up permissions that will allow our Web servers to read, write and execute files on NFS:_

    ```
    sudo chown -R nobody: /mnt/apps
    sudo chown -R nobody: /mnt/logs
    sudo chown -R nobody: /mnt/opt

    sudo chmod -R 777 /mnt/apps
    sudo chmod -R 777 /mnt/logs
    sudo chmod -R 777 /mnt/opt

    sudo systemctl restart nfs-server.service
    ```

    ![nfs_cidr](./img/3.nfscidr.png)

5. Configure access to NFS for clients within the same subnet (example of subnet CIDR - `172.31.32.0/20`)

    ```
    sudo vi /etc/exports

    /mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
    /mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
    /mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

    Esc + :wq!

    sudo exportfs -arv
    ```

    ![exports](./img/3.exports.png)

6. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)  
    `rpcinfo -p | grep nfs`

    _Important note: In order for NFS server to be accessible from your client, you must also open the following ports:_

    - `TCP 111`
    - `UDP 111`
    - `UDP 2049`

    ![nfs_ports](./img/4.nfsports.png)

    ![sg_ports](./img/5.sg_ports.png)

Configure backend database as part of 3 tier architecture
---

### **Step 2**: Configure the database server. Recall how to install and configure a MySQL DBMS to work with a remote webserver.

1. Install [MySQL](https://www.mysql.com/) server on Ubuntu Linux OS `sudo apt update` and `sudo apt install mysql-server`

    ![mysql](./img/6.mysql.png)

2. Create a database and name it `tooling`
3. Create a database user and name it `webaccess`
4. Grant permission to `webaccess` user on `tooling` database to do anything only from the webservers `subnet cidr` which in this case is `172.31.16.0/20`

    ```
    sudo mysql -p password
    CREATE DATABASE tooling;
    CREATE USER 'webaccess'@'172.31.16.0/20' IDENTIFIED BY 'password';
    GRANT ALL ON tooling.* TO 'webaccess'@'172.31.16.0/20';
    FLUSH PRIVILEGES;
    SHOW DATABASES;
    exit
    ```

    ![cr_db](./img/6a.cr_db.png)

### **Step 3**: Prepare the Web Servers

![ws](./img/6b.webservers.png)

We need to make sure that our web servers can serve the same content from shared storage solutions. In our case - NFS Server and MySQL database. We already know that one DB can be accessed for `reads` and `writes` by multiple clients. For storing files that our web servers will use, we will utilize NFS and mount previously created logical volume `lv-apps` to the folder where Apache stores files to be served to the users `/var/www`

This approach will make our web servers `stateless` which means we will be able to add new ones or remove them whenever we need to and the integrity of the data (in the database and on the NFS) will be preserved. In this step we will do the following:

* Configure NFS client (this step must be done on all 3 webservers).  
* Deploy a Tooling application to our webservers into a shared NFS folder.  
* Configure the web servers to work with a single MySQL database.

1. Launch a new EC2 instance with RHEL 8 OS

2. Install NFS client `sudo yum install nfs-utils nfs4-acl-tools -y`

    ![nfs_install](./img/6c.nfs_install.png)

3. Mount `/var/www/` and target the NFS server's export for apps
    > sudo mkdir /var/www  
    > sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

    ![nfs_mnt](./img/6d.nfs_mnt.png)

4. Verify that NFS was mounted successfully by running `df -h`. Make sure that the changes will persist on web server after reboot:

    ![dfh](./img/6e.dfh.png)

    `sudo vi /etc/fstab`

    add the following line:

    `<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`  
    `<NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd nfs defaults 0 0`

    ![fstab](./img/6f.fstab.png)

5. Install `Remi's Repository`, [Apache](https://httpd.apache.org/) and [PHP](https://www.php.net/).

    ```
    sudo yum install httpd -y

    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

    sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

    sudo dnf module reset php

    sudo dnf module enable php:remi-7.4

    sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

    sudo systemctl start php-fpm

    sudo systemctl enable php-fpm

    sudo setsebool -P httpd_execmem 1
    ```
    **Repeat steps 1-5 for another 2 Web Servers.**

    ![inst_apps](./img/6g.install_apps.png)

6. Verify that Apache files and directories are available on the Web Server in `/var/www/` and also on the NFS server in `/mnt/apps`. If you see the same files, it means NFS is mounted correctly. You can try to create a new file `touch test.txt` from one server and check if the same file is accessible from the other webservers.

    ![files_ws](./img/6h.files_on_ws.png)

    ![files_nfs](./img/6i.files_on_nfs.png)

    Creating a new file `test.txt` on `Webserver1:/var/www/`

    ![test_ws](./img/6jtest_on_ws.png)

    You can see from the screenshot below that the new file `test.txt` is visible in `NFSServer:/mnt/apps/`

    ![test_nfs](./img/6k.test_on_nfs.png)

7. Locate the log folder for Apache on the web server and mount it to NFS server's export for logs. `sudo mount -t nfs -o rw,nosuid 172.31.16.71:/mnt/logs /var/log/httpd`. Repeat step no.4 to make sure the mount point will persist after reboot.

    `sudo vi /etc/fstab`

    `172.31.16.71:/mnt/logs /var/log/httpd defaults 0 0`

    ![fstab_logs](./img/6l.fstab_logs.png)

8. I forked the tooling source code from [Darey.io Github Account](https://github.com/darey-io/tooling) to my GitHub account.

    ![forked](./img/7.forked.png)

9. Deploy the tooling website code to the webserver. Ensure that the **html** folder from the repository is deployed to `/var/www/html`

    `sudo yum install git -y`
    ![gitinstall](./img/7a.install_git.png)

    Registered new SSH key to my GitHub account for the tooling webserver

    ![pubkey](./img/7c.pub_key.png)
    
    ![sshkey](./img/7b.ssh_key.png)

    Now I could successfully clone the forked repo from my github account unto my webserver

    `git clone git@github.com:itoroekpo/tooling.git`
    ![clone](./img/7d.clone.png)

    I ensured the contents of the `html` folder was deployed to `/var/www/html`

    `sudo cp -R html/. /var/www/html`
    ![cp_html](./img/7e.cp_html.png)

    **Note 1:** Do not forget to open TCP port 80 on the webservers.

    ![tcp80](./img/7f.tcp_80.png)

    **Note 2:** If you encounter _403 Error_ - check permissions to your `/var/www/html` folder and also disable SELinux `sudo setenforce 0`. To make this change permanent, open the config file `sudo vi /etc/sysconfig/selinux` and set `SELINUX=disabled`, then restart httpd.

    ![seten](./img/7g.seten.png)

    set `SELINUX=disabled`
    ![selinux](./img/7h.selinux.png)

10. Update the website's configuration to connect to the database in `/var/www/html/functions.php` file. 

    Apply `tooling-db.sql` script to your database using the command `mysql -h <database-private-ip> -u <db-username> -p <db-password> < tooling-db.sql`. This creates a table called `users` with a default `admin` user.

    `sudo vi /var/www/html/functions.php`
    ![db_conn](./img/7i.db_connect.png)

    Apply the `tooling-db.sql` script with the command `sudo mysql -h 172.31.28.136 -u webaccess -p --database=tooling < /home/ec2-user/tooling/tooling-db.sql`

    ![nw_tb](./img/7n.table.users.png)

11. Create in MySQL a new admin user with username: `myuser` and password: `password`:

    `USE tooling;`

    `SHOW tables;`
    
    `INSERT INTO users (id, username, password, email, user_type, status) VALUES (2, 'myuser', 'password', 'itoro@mail.com', 'admin', 1);`

    ![ins](./img/7j.insert_user.png)

    You can see from the below screenshot the newly added admin user `myuser`
    ![sh_users](./img/7k.sh_users.png)

12. Finally, open the website in your browser `http://<Web-Server-Public-IP-Address-OR-Public-DNS-Name>/index.php` and make sure you can login into the website with `myuser`.

    > `http://3.88.224.167/login.php`

    ![ws1_inst](./img/wb1_inst.png)

    Login page
    ![login](./img/7l.login.png)

    Login using the admin account user: `admin`, password: `admin`

    You should have the below webpage visible as seen below.

    ![www](./img/7m.tooling.png)

    The tooling website is also visible from `Webserver2` due to NFS as seen below.

    EC2 instance
    ![wb2](./img/8a.web.png)

    Webpage
    ![websv2](./img/8.web2.png)

    And also visible from `Webserver3`
    
    EC2 instance
    ![wb3](./img/8b.wb3_inst.png)

    Webpage
    ![wbs3](./img/wb3.png)



> PROJECT COMPLETED
> WE HAVE IMPLEMENTED A WEB SOLUTION FOR A DEVOPS TEAM FROM 3 WEBSERVER CLIENT'S USING LAMP STACK WITH REMOTE DATABASE AND NFS SERVERS.










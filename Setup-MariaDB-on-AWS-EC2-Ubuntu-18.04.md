# Setting up MariaDB on AWS EC2 Instance

## Locating Addresses for Configuration

**Web Server** (AWS EC2 Instance) **Public DNS**: `ec2-54-162-133-48.compute-1.amazonaws.com`
**Web Server** (AWS EC2 Instance) **Private DNS**: `ip-172-31-46-106.ec2.internal`
**MariaDB Server** (AWS EC2 Instance) **Public DNS**:  `ec2-54-161-105-107.compute-1.amazonaws.com`
MariaDB Server (AWS EC2 Instance) Private DNS: `ip-172-31-41-184.ec2.internal`

*Note: Public DNS can change when EC2 intances rebooted.*

## Steps to setup MariaDB Server on AWS EC2 Instance Ubuntu 18.04

- Login to MariaDB Server AWS EC instance bash and before installing other software, run the commands below to update Ubuntu

    ```bash
    sudo apt update
    sudo apt dist-upgrade
    sudo apt autoremove

- Install MariaDB server and client applications with following command

    ```bash
    sudo apt-get install mariadb-server mariadb-**client**
    ```

- Start/Stop MariaDB Server, using following commands
  
    ```bash
    sudo systemctl start mariadb.service
    sudo systemctl stop mariadb.service
    ```

- Register MariaDB Server as service so that it starts automatically after reboots, using following command

    ```bash
    sudo systemctl enable mariadb.service
    ```

- Setup security installation of MariaDB, using following command

    ```bash
    sudo mysql_secure_installation
    ```

- When prompted, answer the questions below by following the guide.

    ```bash
    Enter current password for root (enter for none): Just press the Enter
    Set root password? [Y/n]: Y
    New password: Enter password
    Re-enter new password: Repeat password
    Remove anonymous users? [Y/n]: Y
    Disallow root login remotely? [Y/n]: Y
    Remove test database and access to it? [Y/n]: Y
    Reload privilege tables now? [Y/n]: Y
    ```

## Steps to create a database on MariaDB Server

- First we can change the default storage engine to innodb and change the default file format to Barracuda by editing `MariaDB` configuration file.
  
    ```bash
    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
    ```

- Add followings under `# * InnoDB section` and save changes.

    ```bash
    default_storage_engine = innodb
    innodb_file_per_table = 1
    innodb_file_format = Barracuda
    innodb_large_prefix = ON
    ```

- Restart MariaDB Server for changes to take affect

    ```bash
    sudo systemctl restart mariadb.service
    ```

- Logon to MariaDB server, using following command (when prompted enter password for root defined above)

    ```bash
    sudo mysql -u root -p
    ```

- After entering MariaDB CLI, create a database called `your-db` with following command
  
    ```sql
    CREATE DATABASE moodle;
    ```

- Create a database user called `your-username` with a new password. To allow `your-username`  connect from Web Server, insert the Private DNS Name of Web Server as shown below.
  
    ```sql
    CREATE USER 'your-username'@'ip-172-31-46-106.ec2.internal' IDENTIFIED BY 'new-password-here';
    ```

- Grant permission to  `your-username` with full access to  `your-db` database with following command
  
    ```sql
    GRANT ALL ON your-db.* TO 'your-username'@'ip-172-31-46-106.ec2.internal' IDENTIFIED BY 'user_password_here' WITH GRANT OPTION;
    ```

- Save changes to be effective with following command
  
    ```sql
    FLUSH PRIVILEGES;
    EXIT;
    ```

- By default, MariaDB only listens for connections from the localhost. All remote access to the server is denied by default. To enable remote access, run the commands below to editMariaDB configuration file.
  
    ```bash
    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
    ```

- Change `bind-address` from `127.0.0.1` to `0.0.0.0` and save it.

- After making the change above, save the file and run the commands below to restart the server.

    ```bash
    sudo systemctl restart mariadb.service
    ```

- To verify changes, run `sudo netstat -anp | grep 3306` command then following output should be printed if changes effective.

    ```bash
    tcp   0   0 0.0.0.0:3306     0.0.0.0:*    LISTEN    3213/mysqld
    ```

- It may be necessary to open Ubuntu Firewall to allow Web Server to connect on port 3306 with following command

    ```bash
    sudo ufw allow from ip-172-31-46-106.ec2.internal to any port 3306
    ```

## Steps to allow connections from Web Server to MariaDB Server

- Assign both Web Server EC2 Instance and MariaDB Server EC2 Instance to `security groups` on AWS Console
  
- Then, open Security Group of MariaDB Server `inbound tab` and edit.

- Lastly, set up inbound connection rule on  Security Group of MariaDB Server to allow connection from Web Server as shown below

![MariaDB Inbound Rules](/resource/MariaDB_InboundRules.png)

## Steps to setup MariaDB client on Web Server for remote connection

- Login to Web Server bash

- Install MariaDB client application with following command
  
    ```bash
    sudo apt-get install mariadb-client
    ```

- Establish remote connection to MariaDB Server with following command using hostname (`MariaDB Server Private DNS`), username(`your-username`) and password predefined in previous steps.

    ```bash
    sudo mysql -uyour_username -pyour_username_password -h ip-172-31-41-184.ec2.internal
    ```

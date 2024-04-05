# install and configure mysql(master/slave/backup/restore)

**Step-by-step guide for how to install MySQL on Ubuntu:**

## **Step 1: Update package index**

```bash
$ sudo apt update
```

## **Step 2: Install the MySQL server**

```bash
$ sudo apt install mysql-server
```

To check that the server is running, you can start it manually with the command **systemctl**.

```bash
$ sudo systemctl start mysql.service
```

## **Step 3: Configure MySQL**

To do this, start the MySQL command prompt:

```bash
$ sudo mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
mysql> exit
```

**Test MySQL**

```bash
$ systemctl status mysql.service
$ sudo mysqladmin -p -u username version
```

# **How to Set Up MySQL Master-Slave Replication**

## **Step 1: Adjust Firewall Settings**

```bash
sudo ufw allow from 10.0.0.5 to any port 3306
```

## **Step 2: Configure the Source Database**

To set up the replication, you must adjust some settings in the source database configuration file. In Ubuntu, the default location of the MySQL configuration file is */etc/mysql/mysql.conf.d/*. Follow the steps below:

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

bind-address            = 10.0.0.4
server-id             = 1
log_bin                       = /var/log/mysql/mysql-bin.log
#make setlog Database:
binlog_do_db          = setlog
```

After you finish editing the file, save and exit the file.

Restart the MySQL server to apply the changes:

```bash
sudo systemctl restart mysql
```

## **Step 3: Create Replication User:**

```bash
sudo mysql

CREATE USER 'master'@'10.0.0.4' IDENTIFIED WITH mysql_native_password BY 'M@ster1990';
GRANT REPLICATION SLAVE ON *.* TO 'master'@'192.168.64.15';
FLUSH PRIVILEGES;
```

## **Step 4: Retrieve Log File Position**

Open the MySQL prompt and run the following command to lock the database:

```bash
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

## **Step 5: Copy Data or Create New Database**

```bash
UNLOCK TABLES;
```

## **Step 6: Configure Slave Server**

Open the MySQL configuration file on the slave server using a text editor:

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

server-id               = 2
log_bin                 = /var/log/mysql/mysql-bin.log
binlog_do_db            = mysql
```

Save the changes and exit the file.

Restart the MySQL server for the changes to take effect:

```bash
sudo systemctl restart mysql
```

## **Step 7: Start Replication**

After configuring both MySQL instances, you can start the replication process. Open the MySQL shell on the slave server and use the syntax below to instruct the server where to find the binary log file and from which position to start reading it:

```bash
CREATE USER 'slave'@'10.0.0.5' IDENTIFIED BY 'sl@ve1990';
GRANT ALL PRIVILEGES ON *.* TO 'slave'@'10.0.0.5';
```

```bash
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='10.0.0.4',
SOURCE_USER='slave',
SOURCE_PASSWORD='sl@ve1990',
SOURCE_LOG_FILE='mysql-bin.000001',
SOURCE_LOG_POS=558;

START REPLICA;
SHOW REPLICA STATUS\G;
```
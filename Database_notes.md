# Database for SysOps

## Create MySQL database hands on

### Create a Mysql database in RDS console

#### DB instance class

When select instance type, toggle on the Include previous generation instance, so you can choose T2.micro instance to stay in the free tier. 

But when I selected T2.micro, I could not see **Enable encryption** option. According to AWS documentation, you cannot set Encryption and Performance Insight.

#### Public access setting

**Yes** - Amazon EC2 instances and devices outside the VPC can connect to your database. Choose one or more VPC security groups that specify which EC2 instances and devices inside the VPC can connect to the database.

**No** - RDS will not assign a public IP address to the database. Only Amazon EC2 instances and devices inside the VPC can connect to your database.

I choose Yes for public access, because I need to connect and test the database, and let AWS create a security group for MySQL RDS. This means my database will be in public subnet. 

#### Security group details

**Inbound rule**

Type: MySQL/Aurora

Protocol: TCP

Port: 3306

Source: My IP address

### Connect to database using Sqlectron

#### Download sqlectron

sqlectron is sql client for access any type of relational database. 

https://sqlectron.github.io/

Download GUI

I download the zip file for the latest version: [sqlectron-1.37.1-win.7z](https://github.com/sqlectron/sqlectron-gui/releases/download/v1.37.1/sqlectron-1.37.1-win.7z), then unzip it> open sqlectron.exe to use it. 

#### Add database connection to sqlectron

Click Add > Enter server Information:

* Name: can be any name
* Database type: MySQL
* Server address: endpoint address of my created databse
* Initial database: Enter the db instance name I gave when I created the database

Test connection, success. Then click save. 

#### Test to create a table and add record in database instance

Connect to my db instance, then try to create a table and add a row of value. 

```
CREATE TABLE R_Releases(
       Release_ID int NOT NULL,
       Release_Name varchar(255) NOT NULL
)

Insert into R_Releases (Release_ID, Release_Name) values (1, 'Window 10 upgrade')
```

## RDS encryption and security

### Encryption

* Encrypt the master & read replicas with **AWS KMS** 

  AWS KMS key options

  * Default KMS/RDS
  * KMS CMK alias or ARN

* Encryption has to be defined at launch time

### Transparent Data Encryption (TDE)

**Transparent Data Encryption (TDE)** was developed with SQL Server 2008, and it is also available in Oracle database management systems. It is an encryption method that protects the core data in the database.

The encryption method protects the data in the database by encrypting the underlying files of the database, and not the data itself. This prevents the data from being hacked and copied to another server; in order to open the files you have to have the original encryption certificate and a master key.

The actual encryption of the database is done at the **page** level. In the context of the actual database, a **page** refers to the unit of data storage in the server (not a web page). A **page** in SQL server is small (8KB in size); therefore **Transparent Data Encryption (TDE)** operates at the structural level of the database.

Because TDE protects/encrypts the structure of the database, it is considered an **at rest** encryption method.

The keyword in the method is **Transparent**. This means that the encryption method is transparent to authorized users of the database.

**Reference:**

https://study.com/academy/lesson/what-is-transparent-data-encryption-tde.html

## RDS Backups and snapshots

Backups cannot be shared

snapshots can be shared.

Database backup is an automatic process, you can enable/disable during create the database. If you disable it, you can manually create snapshot for backup purpose. 

### Automated backups

* Daily full backup of the database (During the maintenance window)
* Transaction logs are backed-up by RDS every 5 minutes

Have ability to restore any point in time (from oldest backup to 5 minutes ago)

### Take snapshot manually

2 ways

1. RDS console> Databases menu > select you database> Actions> select take snapshot> enter snapshot name> click Take snapshot button
2. RDS console> Snapshot menu > Select your database> enter snapshot name> click Take snapshot button

### CloudWatch alarm for RDS

When you create/modify a database, you can choose to export logs to CloudWatch logs.

Then filter the metric of the CloudWatch logs, to create a cloudwatch alarm.

## Aurora Overview

Writer endpoint: Is a DNS name. Even the master fail over, your client still talk to the writer endpoint, and is automatically redirected to the right instance. 

###  Replication features

**Single-master**

Supports multiple reader instances connected to the same storage volume as a single writer instance. This is a good general-purpose option for most workloads.

**Multi-master**

Supports multiple writer instances connected to the same storage volume. This is a good option for when continuous writer availability is required.

### Writer and reader endpoint

Since you can create multiple writers and readers for Aurora cluster. Therefore, there is writer endpoint for all writers and reader endpoint for all readers. There is elastic loader balancer before the writers and readers to balance the connections. 

But for each writer and reader instance, there is dedicated endpoint url to connect. 


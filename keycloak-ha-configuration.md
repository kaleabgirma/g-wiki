<!-- TITLE: Configuring Keycloak Ha -->
<!-- SUBTITLE: This documentation shows step by step Guid to configure Keycloak HA configuration -->

# Prerequisites
## Requirements before installation 
The following tools needs to be configured before installing and configuring keycloak 
- Java 8 JDK 
- Mysql server 5.6 or later
- Keycloak server zip file 
- Mysql connector.jar file Mysql connector.jar file 

# Installation 
## Step-1 Install and configure MYSQL database 
After you install the MYSQL database you have to create user and database for the keycloak project before running the application 
Start your terminal and run the following commands to complete the database configuration 

Connect to the database server 
```batchfile
mysql -u root -p
```

Create keycloak database 

```sql
Create database keycloak;
```

Create keycloak user and grant privileges  
“Note: change the password with the correct password you want” 

```sql
GRANT ALL PRIVILEGES ON *.* TO ‘keycloak’@’%’ IDENTIFIED BY ‘PASSWORD’;
```

Exit from the database server and reconnect to it with the new keycloak user and provide the password 

```batchfile
mysql -u keycloak -p
```

If you successfully logged in then you have completed step 1. 
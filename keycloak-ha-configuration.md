<!-- TITLE: Configuring Keycloak Ha -->
<!-- SUBTITLE: This documentation shows step by step Guid to configure Keycloak HA configuration -->

## Requirements before installation 
The following tools needs to be configured before installing and configuring keycloak 
- Java 8 JDK 
- Mysql server 5.6 or later
- Keycloak server zip file 
- Mysql connector.jar file Mysql connector.jar file 

## Step-1 Install and configure MYSQL database 
After you install the MYSQL database you have to create user and database for the keycloak project before running the application 
Start your terminal and run the following commands to complete the database configuration 
Connect to the database server 
 
```batchfile
mysql -u root -p
```

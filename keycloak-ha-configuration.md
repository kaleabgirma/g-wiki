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

## Step-2 Install the dependencies and download the keycloak project 
- Download and install Java 8 JDK to you server 
- Download and unzip the keycloak project from the official website to your server preferd directory in this configuration we preferred to put the project in /data/ directory
- Download and unzip the mysql connector from the mysql official site in your proffered directory

## Step-3 configure keycloak mysql connector 
First our keycloak project directory structure looks like 
**/data/keycloak-6.0.1/** 

Create the following directory in the directory */data/keycloak-6.0.1/modules* by running the following command 
```batchfile
mkdir com 
mkdir com/mysql 
mkdir com/mysql/main 
```

Then put the downloaded mysql connector.jar file in the following directory 
*/data/keycloak-6.0.1/modules/com/mysql/main and it looks like 
/data/keycloak-6.0.1/modules/com/mysql/main/mysql-connector-java-8.0.16.jar *
Now create new file called **module.xml** and paste the following code in it 

```batchfile
nano module.xml 
```

Then insert the following code in the file.

```xml
<?xml version="1.0" ?> 
<module xmlns="urn:jboss:module:1.3" name="com.mysql"> 
 <resources> 
  <resource-root path="mysql-connector-java-8.0.16.jar" /> 
 </resources> 
 <dependencies> 
  <module name="javax.api"/> 
  <module name="javax.transaction.api"/> 
 </dependencies> 
</module> 
```

Now go to the following directory and change the **standalone-ha.xml** configuration file 

*cd /data/keycloak-6.0.1/standalone/configuration/ *


```sh
nano standalone-ha.xml 
```

ctr+w and then type the following text to search without the external quote "<driver name="h2" after locating the following string"

```xml
<driver name="h2" module="com.h2database.h2"> 
                        <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</xa-datasource-class> 
</driver>
```
Then add the following mysql driver to make it look like the following 

```xml
<driver name="mysql" module="com.mysql"> 
        <driver-class>com.mysql.jdbc.Driver</driver-class> 
</driver> 
<driver name="h2" module="com.h2database.h2"> 
         <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</xa-datasource-class> 
</driver>
```
Now is the time to add the database connection data source 
In the same file find for the string  
**<datasource jndi-name="java:jboss/datasources/KeycloakDS"**  
Which looks like

```xml
<datasource jndi-name="java:jboss/datasources/KeycloakDS" pool-name="KeycloakDS" enabled="true" use-java-context="true" statistics-enabled="${wildf$ 
    <connection-url>jdbc:h2:${jboss.server.data.dir}/keycloak;AUTO_SERVER=TRUE</connection-url> 
                    <driver>h2</driver> 
                    <security> 
                        <user-name>sa</user-name> 
                        <password>sa</password> 
                    </security> 
                </datasource> 
```

Replace the entire string with the following one

```xml
<datasource jndi-name="java:/jboss/datasources/KeycloakDS" pool-name="KeycloakDS" enabled="true"> 
        <connection-url>jdbc:mysql://10.96.16.225:3306/keycloak?useSSL=false&amp;characterEncoding=UTF-8</connection-url> 
          <driver>mysql</driver> 
          <pool> 
              <min-pool-size>5</min-pool-size> 
              <max-pool-size>15</max-pool-size> 
          </pool> 
          <security> 
              <user-name>keycloak</user-name> 
              <password>password</password> 
          </security> 
          <validation> 
              <valid-connection-checker class-name="org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLValidConnectionChecker"/> 
              <validate-on-match>true</validate-on-match> 
              <exception-sorter class-name="org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLExceptionSorter"/> 
          </validation> 
</datasource> 
```

Next step is to change the binding address to the current public IP address that the server can connect to. 
In the same configuration file standalone-ha.xml find the line **<interface name="public">** 
Then change the IP address from 127.0.0.1 to the public IP that can be seen by remote servers and also the **<interface name="management">** too. 
Do this step in the second server too, give its own IP address to be reached by the other servers. 

## Step-4 Create JDBC-Ping configuration
To configure JDBC-PING create a file with the following name in the directory

```sh
cd /data/keycloak-6.0.1/bin 
nano JDBC_PING.cli 
```

Paste the following source code inside the file 

```batchfile
embed-server --server-config=standalone-ha.xml --std-out=echo 
batch 
/subsystem=infinispan/cache-container=keycloak/distributed-cache=sessions:write-attribute(name=owners, value=${env.CACHE_OWNERS:2}) 
/subsystem=infinispan/cache-container=keycloak/distributed-cache=authenticationSessions:write-attribute(name=owners, value=${env.CACHE_OWNERS:2}) 
/subsystem=infinispan/cache-container=keycloak/distributed-cache=offlineSessions:write-attribute(name=owners, value=${env.CACHE_OWNERS:2}) 
/subsystem=infinispan/cache-container=keycloak/distributed-cache=loginFailures:write-attribute(name=owners, value=${env.CACHE_OWNERS:2}) 
/subsystem=jgroups/stack=tcp:remove() 
/subsystem=jgroups/stack=tcp:add() 
/subsystem=jgroups/stack=tcp/transport=TCP:add(socket-binding="jgroups-tcp") 
/subsystem=jgroups/stack=tcp/protocol=JDBC_PING:add() 
/subsystem=jgroups/stack=tcp/protocol=JDBC_PING/property=datasource_jndi_name:add(value=java:jboss/datasources/KeycloakDS) 
/subsystem=jgroups/stack=tcp/protocol=JDBC_PING/property=initialize_sql:add(value="CREATE TABLE IF NOT EXISTS JGROUPSPING (own_addr varchar(200) NOT NULL, cluster_name varchar(200) NOT NULL, updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, ping_data varbinary(5000) DEFAULT NULL, PRIMARY KEY (own_addr, cluster_name)) ENGINE=InnoDB DEFAULT CHARSET=utf8") 
/subsystem=jgroups/stack=tcp/protocol=MERGE3:add() 
/subsystem=jgroups/stack=tcp/protocol=FD_SOCK:add(socket-binding="jgroups-tcp-fd") 
/subsystem=jgroups/stack=tcp/protocol=FD:add() 
/subsystem=jgroups/stack=tcp/protocol=VERIFY_SUSPECT:add() 
/subsystem=jgroups/stack=tcp/protocol=pbcast.NAKACK2:add() 
/subsystem=jgroups/stack=tcp/protocol=UNICAST3:add() 
/subsystem=jgroups/stack=tcp/protocol=pbcast.STABLE:add() 
/subsystem=jgroups/stack=tcp/protocol=pbcast.GMS:add() 
/subsystem=jgroups/stack=tcp/protocol=pbcast.GMS/property=max_join_attempts:add(value=5) 
/subsystem=jgroups/stack=tcp/protocol=MFC:add() 
/subsystem=jgroups/stack=tcp/protocol=FRAG3:add() 
 
/subsystem=jgroups/stack=udp:remove() 

 
/subsystem=jgroups/channel=ee:write-attribute(name=stack, value=tcp) 
/socket-binding-group=standard-sockets/socket-binding=jgroups-mping:remove() 
run-batch 
try 
    :resolve-expression(expression=${env.JGROUPS_DISCOVERY_EXTERNAL_IP}) 
    /subsystem=jgroups/stack=tcp/transport=TCP/property=external_addr/:add(value=${env.JGROUPS_DISCOVERY_EXTERNAL_IP}) 
catch 
    echo "JGROUPS_DISCOVERY_EXTERNAL_IP maybe not set." 
end-try 
stop-embedded-server
```
Then run the following command in the terminal from the **/data/keycloak-6.0.1/bin** directory

```sh
./jboss-cli --file= JDBC_PING.cli 
```

If you have any errors please try to fix it until you get *command executed successfully* message 

## Step-5 Running keycloak in two servers
- Before running the project copy the source directory to the second server in your preferred directory
- Install java JDK 8 in the second server 

Now you are ready to run the project use the following command to run it 
From */data/keycloak-6.0.1/bin* directory 

```sh
nohup ./standalone.sh --server-config=standalone-ha.xml & 
```

To follow the log type the following command 


```sh
tail -f /data/keycloak-6.0.1/standalone/log/server.log 
```

You should see something similar to the following screenshot on your server log while it is starting.

===================================================================///===================================================
																																					This is the end of the documentation
=========================================================================================================================
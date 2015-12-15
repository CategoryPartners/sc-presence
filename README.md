# sc-presence
Socket Presence Module for SocketCluster

# Features
### Track active users in your SocketCluster based application
### Stores all active socket and channel subscriptions, along with related metadata
### Works across multiple workers or hosts
### Garbage collection process cleans out any abandoned records
### Simple to install and use.  Requires only a MySQL db and a single line of code.
### Lots of configuration and customization options
### Publishes a usercount update whenever someone joins or leaves
### Works with or without authenticated users


# Install

## Create Database

```sql
CREATE DATABASE IF NOT EXISTS `SCPresence`;
USE SCPresence;

DROP USER 'SCP_user'@'localhost';
FLUSH PRIVILEGES;
CREATE USER 'SCP_user'@'localhost' IDENTIFIED BY 'putyourpasswordhere';
GRANT SELECT ON `SCPresence`.* TO 'SCP_user'@'localhost'; 
GRANT INSERT ON `SCPresence`.* TO 'SCP_user'@'localhost'; 
GRANT UPDATE ON `SCPresence`.* TO 'SCP_user'@'localhost'; 
GRANT DELETE ON `SCPresence`.* TO 'SCP_user'@'localhost'; 
GRANT EXECUTE ON `SCPresence`.* TO 'SCP_user'@'localhost'; 


CREATE TABLE IF NOT EXISTS `SCPresence_users` (
  SCP_id INT(11) NOT NULL AUTO_INCREMENT,
  SCP_socket_id VARCHAR(255) DEFAULT NULL,
  SCP_user_id INT(11) DEFAULT NULL,
  SCP_channel VARCHAR(255) DEFAULT NULL,
  SCP_updated DATETIME DEFAULT CURRENT_TIMESTAMP,
  SCP_authToken VARCHAR(2048) DEFAULT NULL,  
  SCP_ip VARCHAR(255) DEFAULT NULL,
  SCP_origin VARCHAR(1024) DEFAULT NULL,
  PRIMARY KEY (SCP_id),
  UNIQUE INDEX IX_unique_user_channel_socket (SCP_user_id, SCP_channel, SCP_socket_id)
)
ENGINE = INNODB;
```

## Install NPM Package
```
npm install sc-presence
```

## Attach sc-presence to your workers
Within worker.js, add the following code:
```javascript
var scPresence = require('./sc-presence/sc-presence.js');
module.exports.run = function (worker) {
    scPresence.attach(worker, options);
};
```


# Options (Only options you want to override are required)

## scpGcWorkerId
The worker id of the worker that will handle sc-presence garbage collection duties			    
Default Value: 0

## scpGcInterval			    
The interval in number of seconds on which the garbate collection process will run
Default Value: 60 

## scpGcThreshold			    
The number of seconds that must pass without an update before the garbage collection process will remove a record
Default Value: 120

## scpBlockUsercountThreshold	
The number of seconds sc-presence will wait after startup before starting to publish usercount updates.
This prevents sc-presence from spamming usercount updates when the system is restarted and sockets are reconnecting.
Default Value: 60

## scpSCPingsPerUpdate         
The number of scServer.pingInterval periods that must pass before sc-presence will fire a database update
Default Value: 5  

## scpUsercountChannel		    
The name of the channel on which sc-presence will publish usercount updates
Default Value: "USERCOUNT"

## scpUsercountType            
The type of usercount update sc-presence will publish when a user joins or leaves.  Options are "SUBSCRIPTIONS", "SOCKETS", or "USERS".
Default Value: "USERS"

## scpPresenceChannel			
The name of the channel that sc-presence will register primary socket presence under
Default Value: "_SCPRESENCE"

## scpDbhost					
The host name of the sc-presence db
Default Value: ""

## scpDbname					
The name of the sc-presence db
Default Value: "SCPresence"

## scpDbTablename	
The name of the db table where sc-presence data is stored			
Default Value: "SCPresence_users"

## scpDbuser	
The name of the db user that will authenticate to the sc-presence db				
Default Value: "SCP_user"

## scpDbpassword
The password for the db user that will authenticate to the sc-presence db				
Default Value: ""  
      
## scpConnectUpdateDelay		
When a new socket connects, sc-presence will wait this many ms before publishing a usercount update.  
This ensures the socket that connected has time to subscribe to the scpUsercountChannel channel before the usercount is published
Default Value: 3000

## scpUserIdField    
The name of the property in the authToken which will be stored in the SCP_user_id field in the database (numeric or string values are ok)          
Default Value: "user_id"
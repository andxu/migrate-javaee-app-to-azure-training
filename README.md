---
services: app-service, PostgreSQL, MySQL, azure-sql-database
platforms: java
author: selvasingh, sadigopu
---

# Quick steps to deploy JBoss sample app to azure webapp using mysql

## Login
```bash
az login
az account show
```

# Create mysql server

```bash
export SUBSCRIPTION=$(az account show --query id --output tsv)
export RESOURCE_GROUP=migrate-javaee-app
export WEBAPP=migrate-petstore-andxu
export REGION=eastus

export DATABASE_SERVER=migrate-petstore-andxu
export DATABASE_ADMIN=andxu
export DATABASE_ADMIN_PASSWORD=SuperS3cr3t


export MYSQL_SERVER_NAME=mysql-${DATABASE_SERVER}
export MYSQL_SERVER_ADMIN_LOGIN_NAME=${DATABASE_ADMIN}
export MYSQL_SERVER_ADMIN_PASSWORD=${DATABASE_ADMIN_PASSWORD}
export MYSQL_DATABASE_NAME=petstore

export MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_NAME}.mysql.database.azure.com
export MYSQL_CONNECTION_URL=jdbc:mysql://${MYSQL_SERVER_FULL_NAME}:3306/${MYSQL_DATABASE_NAME}?ssl=true\&useLegacyDatetimeCode=false\&serverTimezone=GMT
export MYSQL_SERVER_ADMIN_FULL_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME}\@${MYSQL_SERVER_NAME}

export DEVBOX_IP_ADDRESS=167.220.255.76
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home

az mysql server create --resource-group ${RESOURCE_GROUP} \
    --name ${MYSQL_SERVER_NAME}  --location ${REGION} \
    --admin-user ${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
    --admin-password ${MYSQL_SERVER_ADMIN_PASSWORD} \
    --sku-name GP_Gen5_32 \
    --ssl-enforcement Disabled \
    --version 5.7

az mysql server firewall-rule create --name allAzureIPs \
    --server ${MYSQL_SERVER_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

az mysql server firewall-rule create --name myDevBox \
    --server ${MYSQL_SERVER_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --start-ip-address ${DEVBOX_IP_ADDRESS} --end-ip-address ${DEVBOX_IP_ADDRESS}

az mysql server configuration set --name wait_timeout \
    --resource-group ${RESOURCE_GROUP} \
    --server ${MYSQL_SERVER_NAME} --value 2147483


```


## Configure mysql database

1. Connect to mysql server:
```bash
mysql -u ${MYSQL_SERVER_ADMIN_FULL_NAME}  -h ${MYSQL_SERVER_FULL_NAME} -P 3306 -p
```

in the password prompt, input mysql password: `SuperS3cr3t`

2. Create `petstore` database,

```bash
CREATE DATABASE petstore;
CREATE USER 'root' IDENTIFIED BY 'petstore';
GRANT ALL PRIVILEGES ON petstore.* TO 'root';
CALL mysql.az_load_timezone();
SELECT name FROM mysql.time_zone_name limit 10;
quit
```

3. Change timezone

```bash
az mysql server configuration set --name time_zone \
    --resource-group ${RESOURCE_GROUP} \
    --server ${MYSQL_SERVER_NAME} --value "US/Pacific"
```


## Deploy the JBoss sample project


## Change app settings


`mvn package azure-webapp:deploy -Dmaven.test.skip=true -Ddb=mysql`

## Configure SSL/TLS encryption for CDH Services to their respective MySQL Databases

#### This article covers the steps required to enable TLS/SSL encryption for all CDH services(Hive, Sentry, Hue, Oozie) and along with Management Services(Cloudera Manager, Activity Monitor, Reports Manager, Navigator Audit Server, Navigator Metadata Server) and their respective MySQL databases. 


> The article has been tested against the following versions:
> * CM: 5.15.1
> * CDH: 5.15.1
> * MySQL Server: 5.7.21(AWS RDS Instance)
> * MySQL Connector: 8.0.15

**1. Cloudera Manager**

In /etc/cloudera-scm-server/db.properties, add:
```
com.cloudera.cmf.orm.hibernate.connection.url=jdbc:mysql://<db_host>/<database>?useSSL=true&enabledTLSProtocols=TLSv1.2
```
In /etc/default/cloudera-scm-server, add: 
```
export CMF_JAVA_OPTS="-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>"
```
After making the changes, restart cloudera-scm-server.

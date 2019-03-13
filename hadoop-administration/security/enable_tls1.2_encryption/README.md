## Configure SSL/TLS(TLS1.2) encryption for CDH Services to their respective MySQL Databases

#### This article covers the steps required to enable TLS/SSL encryption for all CDH services(Hive, Sentry, Hue, Oozie) and along with Management Services(Cloudera Manager, Activity Monitor, Reports Manager, Navigator Audit Server, Navigator Metadata Server) and their respective MySQL databases. 


> The article has been tested against the following versions:
> * CM: 5.15.1
> * CDH: 5.15.1
> * MySQL Server: 5.7.21(AWS RDS Instance)
> * MySQL Connector: 8.0.15
> * JDK: 1.8.0

**1. Cloudera Manager**

In /etc/cloudera-scm-server/db.properties, add:
```
com.cloudera.cmf.orm.hibernate.connection.url=jdbc:mysql://<db_host>/<db_name>?useSSL=true&enabledTLSProtocols=TLSv1.2
```
In /etc/default/cloudera-scm-server, add: 
```
export CMF_JAVA_OPTS="-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>"
```
After making the changes, restart cloudera-scm-server.

**2. Reports Manager**

In Java Configuration Options for Reports Manager, add:
```
-Dcom.cloudera.enterprise.dbutil.MySqlHandler.EXTRA_PARAMETERS=useSSL=true&requireSSL&enabledTLSProtocols=TLSv1.2
-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>
```

**3. Activity Monitor**

In Activity Monitor Advanced Configuration Snippet (Safety Valve) for cmon.conf, add: 
```
<property> 
<name>db.hibernate.connection.url</name> 
<value>jdbc:mysql://<db_host>/<db_name>?useSSL=true&requireSSL=true&useUnicode=true&characterEncoding=UTF-8&enabledTLSProtocols=TLSv1.2</value>
<description>Enable TLS1.2 encryption</description>
</property>
```

**4. Navigator Audit Server**

In Java Configuration Options for Navigator Audit, add:
```
-Dcom.cloudera.enterprise.dbutil.MySqlHandler.EXTRA_PARAMETERS=useSSL=true&requireSSL&enabledTLSProtocols=TLSv1.2
-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>
```

**5. Navigator Metadata Server**

In Java Configuration Options for Navigator Metadata Server, add:
```
-Dcom.cloudera.enterprise.dbutil.MySqlHandler.EXTRA_PARAMETERS=useSSL=true&requireSSL&enabledTLSProtocols=TLSv1.2
-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>
```



## Configure SSL/TLS(TLS1.2) encryption for CDH Services to their respective MySQL Databases

#### This article covers the steps required to enable TLS/SSL encryption for all CDH services(Hive, Sentry, Hue, Oozie) and along with Management Services(Cloudera Manager, Activity Monitor, Reports Manager, Navigator Audit Server, Navigator Metadata Server) and their respective MySQL databases. 


> The article has been tested against the following versions:
> * CM: 5.15.1
> * CDH: 5.15.1
> * MySQL Server: 5.7.21(AWS RDS Instance)
> * MySQL Connector: 8.0.15
> * JDK: 1.8.0

Before proceeding to making changes in the Cloudera Manager, ensure that your MySQL Server supports TLS 1.2.
Login to the MySQL shell and execute the following: 

```
mysql> SHOW GLOBAL VARIABLES LIKE 'tls_version';
+---------------+-----------------------+
| Variable_name | Value                 |
+---------------+-----------------------+
| tls_version   | TLSv1,TLSv1.1,TLSv1.2 |
+---------------+-----------------------+
```

Additionally if you want to enforce connections only via TLS1.2, you can restrict this in the Java configurations as well. 
```
jdk.tls.disabledAlgorithms=SSLv3, TLSv1, TLSv1.1, RC4, MD5withRSA, DH keySize < 768, 3DES_EDE_CBC, TLSv1, TLSv1.1
```

This will disable the older TLS protocol versions. 

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
After making the changes, restart Reports Manager.

**3. Activity Monitor**

In Activity Monitor Advanced Configuration Snippet (Safety Valve) for cmon.conf, add: 
```
<property> 
<name>db.hibernate.connection.url</name> 
<value>jdbc:mysql://<db_host>/<db_name>?useSSL=true&requireSSL=true&useUnicode=true&characterEncoding=UTF-8&enabledTLSProtocols=TLSv1.2</value>
<description>Enable TLS1.2 encryption</description>
</property>
```

After making the changes, restart Activity Monitor.

**4. Navigator Audit Server**

In Java Configuration Options for Navigator Audit, add:
```
-Dcom.cloudera.enterprise.dbutil.MySqlHandler.EXTRA_PARAMETERS=useSSL=true&requireSSL&enabledTLSProtocols=TLSv1.2
-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>
```

After making the changes, restart Navigator Audit Server. 

**5. Navigator Metadata Server**

In Java Configuration Options for Navigator Metadata Server, add:
```
-Dcom.cloudera.enterprise.dbutil.MySqlHandler.EXTRA_PARAMETERS=useSSL=true&requireSSL&enabledTLSProtocols=TLSv1.2
-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>
```

After making the changes, restart Navigator Metadata Server. 

**6. Oozie**

In Oozie Server Advanced Configuration Snippet (Safety Valve) for oozie-site.xml, add:
```
<property> 
<name>oozie.service.JPAService.jdbc.url</name> 
<value>jdbc:mysql://<db_host>/<db_name>?useUnicode=true&characterEncoding=UTF-8&useSSL=true&ssl_ca=<path_to_ca.pem>&enabledTLSProtocols=TLSv1.2</value>
<description>Enable TLS1.2 encryption</description>
</property> 
```
After making the changes, restart Oozie. 

**7. Sentry**

In Sentry Server Advanced Configuration Snippet (Safety Valve) for sentry-site.xml, add:
```
<property> 
<name>sentry.store.jdbc.url</name> 
<value>jdbc:mysql://<db_host>/<db_name>?useSSL=true&requireSSL=true&useUnicode=true&characterEncoding=UTF-8 &enabledTLSProtocols=TLSv1.2</value>
<description>Enable TLS1.2 encryption</description>
</property> 
```
After making the changes, restart Sentry. 

**8. Hive Metastore Server**: 

In Hive Metastore Server Advanced Configuration Snippet (Safety Valve) for hive-site.xml, add:
```
<property>
<name>javax.jdo.option.ConnectionURL</name>
<value>jdbc:mysql://<db_host>/<db_name>?useSSL=true&requireSSL=true&enabledTLSProtocols=TLSv1.2</value>
<description>Enable TLS1.2 encryption</description>
</property> 
```

In Java Configuration Options for Hive Metastore Server add the following:
```
-Djavax.net.ssl.trustStore=<path_to_truststore> -Djavax.net.ssl.trustStorePassword=<password>
```

After making the changes, restart Hive. 

**9. Hue**:

In Hue Server Advanced Configuration Snippet (Safety Valve) for hue_safety_valve_server.ini:
```
[desktop] 
[[database]] 
options='{"ssl": {"ca": "/opt/cloudera/security/pki/<path-to-root-ca>.pem"}}' 
[[session]] 
secure=true
```

After making the changes, restart Hue. 

**Testing the connection**

In order to test whether all services are using SSL or not, login to MySQL shell and execute the following command:
```
mysql> SELECT sbt.variable_value AS tls_version,t2.variable_value AS cipher,processlist_user AS user,processlist_host AS host
       FROM performance_schema.status_by_thread  AS sbt
       JOIN performance_schema.threads AS t ON t.thread_id = sbt.thread_id
       JOIN performance_schema.status_by_thread AS t2 ON t2.thread_id = t.thread_id
       WHERE sbt.variable_name = 'Ssl_version' and t2.variable_name = 'Ssl_cipher' ORDER BY tls_version;
```

Verify the TLS version for all connections. It should state TLSv1.2. 

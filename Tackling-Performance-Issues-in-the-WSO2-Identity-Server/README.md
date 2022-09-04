# How to Tackle Performance Issues in the WSO2 Identity Server

## Table of Contents

- [Table of Contents](#table-of-contents)
- [General Notes about this Documentation](#general-notes-about-this-documentation)
- [High Amounts of Records in the Database Table](#high-amounts-of-records-in-the-database-table)
    - [Overview](#overview)
    - [Investigation and Solutions](#investigation-and-solutions)
        - [Cleanup Procedure Locations](#cleanup-procedure-locations)
- [Indexes not Added](#indexes-not-added)
    - [Overview](#overview)
    - [Investigation and Solutions](#investigation-and-solutions)
    - [Additional Information](#additional-information)
- [Unsuitable Database Pooling Configurations](#unsuitable-database-pooling-configurations)
    - [Overview](#overview)
    - [Solution](#solution)
- [Already Enabled Debug Logs](#already-enabled-debug-logs)
    - [Overview](#overview)
    - [Procedure](#procedure)
    - [Default Log4j Property Files](#default-log4j-property-files)
- [Calling External Services](#calling-external-services)
    - [Overview](#overview)
    - [Solution](#solution)
- [LDAP/LDAPS Connection Pool Issues](#ldapldaps-connection-pool-issues)
    - [Overview and Current Solution](#overview-and-current-solution)
    - [Additional Configurations](#additional-configurations)
- [LDAP Referrals](#ldap-referrals)
    - [Overview](#overview)
    - [Solution](#solution)
- [Insufficient Resources to Run the Server](#insufficient-resources-to-run-the-server)
    - [Overview](#overview)
    - [Solution](#solution)
- [Network Delays between Clients and the WSO2 IS](#network-delays-between-clients-and-the-wso2-is)
    - [Overview](#overview)
    - [Investigation](#investigation)
- [Performance Troubleshooting Methods](#performance-troubleshooting-methods)
    - [Slow Query Reports](#slow-query-reports)
        - [Overview](#overview)
        - [How to Enable Slow Query Reports](#how-to-enable-slow-query-reports)
        - [How to Analyze Slow Query Reports](#how-to-analyze-slow-query-reports)
    - [Correlation Logs](#correlation-logs)
        - [Overview](#overview)
        - [How to Enable and Analyze Correlation Logs](#how-to-enable-and-analyze-correlation-logs)
    - [Thread and Heap Dumps](#thread-and-heap-dumps)
        - [Overview](#overview)
            - [Thread Dump](#thread-dump)
        - [Heap Dump](#heap-dump)
        - [How to Extract Thread/Heap Dumps](#how-to-extract-threadheap-dumps)
            - [Extract Thread Dump](#extract-thread-dump)
            - [Extract Thread Dump](#extract-thread-dump)
        - [How to Analyze Thread/Heap Dumps](#how-to-analyze-threadheap-dumps)
            - [Analyze Thread Dump](#analyze-thread-dump)
            - [Analyze Heap Dump](#analyze-heap-dump)

## General Notes about this Documentation

* The documentation is constructed as a key reference point of view for both the WSO2 Customer Success Engineers and the
  customers.
* The documentation will only contain the vital information that is required to tackle performance issues hence there
  can be places where minor information is left off. However, we have tried to cover them as much as possible within
  this document. If you feel like we have missed out on vital information please reach out to us and let us know how to
  improve the documentation.
* The documentation will point out certain issues in the official documentation site. These will most probably be fixed
  by the documentation team hence your patience on it is appreciated.
* A few trusted external documentation sites are referred throughout this document, but you should be cautious while
  visiting those sites and while following those instructions since they are not handled/updated by the WSO2 team.
* This documentation considers that you have already updated the product to the latest update level that is available so
  that most of the bugs/security fixes are already in place. Also, please make sure that you have properly read the
  update summary documentation since there can be configuration changes that might be important for your product.
* The documentation might update once in a while hence it would be appreciated if you could keep a note on that.

## High Amounts of Records in the Database Table

### Overview

High amounts of data are specifically seen in the following scenarios irrespective of the WSO2 Identity Server (WSO2 IS)
version.

* The cleanup tasks are not properly configured to get rid of the unwanted records from the database tables.
* There is a high load of incoming traffic towards the WSO2 IS.

### Investigation and Solutions

To check if the database tables contain a lot of records, you can browse through the database and extract the number of
records stored in the following tables that are responsible for storing access tokens and sessions accordingly. Please
note that there can be other tables as well.

* `SELECT COUNT(*) IDN_OAUTH2_ACCESS_TOKEN;`
* `SELECT COUNT(*) IDN_AUTH_SESSION_STORE;`

If you see there is a massive number of records in any of the above-mentioned tables, please check whether you have
configured the cleanup tasks according to the type of the database. The names of the cleanup stored procedures are as
follows.

* `CLEANUP_SESSION_DATA`
* `WSO2_TOKEN_CLEANUP_SP`

After observing the above, if you did not find the relevant stored procedures, please make sure to add them properly as
explained in the following sections according to the version of the WSO2 IS.

**WSO2 IS < 5.9.0 Configurations**

Open the identity.xml file located in the _<IS_HOME>/repository/conf/identity_ directory and update the following tag
values as mentioned below.

```
<EnableTokenCleanup>false</EnableTokenCleanup>

<SessionDataCleanUp>
    <Enable>false</Enable>
</SessionDataCleanUp>

<OperationDataCleanUp>
    <Enable>false</Enable>
</OperationDataCleanUp>

<TempDataCleanup>
    <Enable>false</Enable>
</TempDataCleanup>
```

**WSO2 IS >= 5.9.0 Configurations**

Apply the following configuration blocks to the deployment.toml file located in the _<IS_HOME>/repository/conf_
directory. These are added in order to disable the internal cleanup tasks (token and session) done by the WSO2 IS that
are not efficient. After applying the configurations, please make sure to restart the WSO2 IS so that the configurations
are applied correctly.

```
[oauth.token_cleanup]
enable = false

[session_data.cleanup]
enable_expired_data_cleanup = false
clean_logged_out_sessions_at_immediate_cycle = false
enable_pre_session_data_cleanup = false
```

#### Cleanup Procedure Locations

The up-to-date cleanup procedures can be found in the following locations based on the type of the database that you are
currently using.

| Database Type            | Token Cleanup Procedure (TCP)                                                                                                                                                                                                                                               | Session Cleanup Procedure (SCP)                                                                                                                                                                                                                                                           |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| DB2                      | [DB2 TCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/db2/token-cleanup/oauth2-token-cleanup.sql)                                      | [DB2 SCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/db2/sessiondata-cleanup/db2-session-data-cleanup.sql)                                          |
| MSSQL                    | [MSSQL TCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/mssql/token-cleanup/mssql-tokencleanup.sql)                                    | [MSSQL SCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/mssql/sessiondata-cleanup/mssql-session-data-cleanup.sql)                                    |
| MySQL                    | [MySQL TCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/mysql/token-cleanup/mysql-token-cleanup.sql)                                   | [MySQL SCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/mysql/sessiondata-cleanup/mysql-session-data-cleanup.sql)                                    |
| Oracle                   | [Oracle TCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/oracle/token-cleanup/oracle-token-cleanup.sql)                                | [Oracle SCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/oracle/sessiondata-cleanup/oracle-sessiondata-cleanup.sql)                                  |
| PostgreSQL 9.X and 10.x  | [PostgreSQL 9.X TCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/postgresql/postgre-9x/token-cleanup/postgresql-token-cleanup.sql)     | [PostgreSQL 9.X SCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/postgresql/postgre-9x/sessiondata-cleanup/postgresql-session-data-cleanup.sql)      |
| PostgreSQL 11.X or newer | [PostgreSQL 11.X TCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/postgresql/postgre-11x/token-cleanup/postgresql_11-tokencleanup.sql) | [PostgreSQL 11.X SCP Location](https://github.com/wso2/carbon-identity-framework/blob/master/features/identity-core/org.wso2.carbon.identity.core.server.feature/resources/dbscripts/stored-procedures/postgresql/postgre-11x/sessiondata-cleanup/postgresql_11-session-data-cleanup.sql) |

Ideally, the recommendation is for them to be scheduled daily in an off-peak time. But if your session data table/token
data table is growing too fast (there is a high load of incoming traffic towards the WSO2 IS) please increase the
frequency of the session data cleanup/token data cleanup task hourly or every 03 hours, etc…

The official [documentation](https://is.docs.wso2.com/en/latest/setup/removing-unused-tokens-from-the-database/)
explains the token cleanup process further. Please note that there is no documentation for the session data cleanup
process.

## Indexes not Added

### Overview

One of the most important things when it comes to performance is database indexes. The WSO2 IS is shipped with most of
the required database indexes that are available in the default product database scripts. Hence, creating the database
using the default DDLs available that are residing at the _<IS_HOME>/dbscripts_ directory would be sufficient to create
the required indexes. However, there are certain situations where these indexes get updated through Update 2.0 (U2)/WSO2
Update Manager (WUM) updates hence this requires a change based on the new configurations. Furthermore, these changes
should be handled by the Database Administrator (DBA) hence he/she should propagate the changes to the database in order
to achieve optimum performance.

### Investigation and Solutions

To validate if the mandatory indexes are already existing, you can inspect the database for it. The indexes mentioned in
[documentation](https://docs.google.com/document/d/1PJ-EM3aQArxeYtzIr_pnc2xjH4Iwo-bupPoMVA5nppQ/edit#) are the mandatory
indexes to improve the overall performance of the WSO2 IS. If the indexes mentioned in the above-listed documentation
is not available, please make sure to include them as they are mandatory as explained above. To check the indexes
applied on a specific table you can use the commands mentioned in the below table according to the type of the database.
If you are using a different type of database, please use your search engine and find out a command.

| Database Type                                                                                                                          | Command to list indexes of a table                                                                                                                                                                                                                                                                                                                                                                                                                                             | 
|----------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [DB2](https://www.ibm.com/docs/en/sdse/6.4.0?topic=caches-db2-indexes)                                                                 | ```SELECT NAME, UNIQUERULE, CLUSTERING FROM SYSIBM.SYSINDEXES WHERE TBNAME=’TABLE_NAME’```                                                                                                                                                                                                                                                                                                                                                                                     |
| [MSSQL](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-indexes-transact-sql?view=sql-server-ver16) | ```SELECT i.name AS index_name,i.type_desc,is_unique,ds.type_desc AS filegroup_or_partition_scheme,ds.name AS filegroup_or_partition_scheme_name,ignore_dup_key,is_primary_key,is_unique_constraint,fill_factor,is_padded,is_disabled,allow_row_locks,allow_page_locks FROM sys.indexes AS i INNER JOIN sys.data_spaces AS ds ON i.data_space_id = ds.data_space_id WHERE is_hypothetical = 0 AND i.index_id <> 0 AND i.object_id = OBJECT_ID(‘DATABASE_NAME.TABLE_NAME’');``` |
| [MySQL](https://dev.mysql.com/doc/refman/8.0/en/show-index.html)                                                                       | ```SHOW EXTENDED INDEX from TABLE_NAME;```                                                                                                                                                                                                                                                                                                                                                                                                                                     | 
| [Oracle](https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_1069.htm#i1578369)                                           | ```select * from dba_indexes where table_name='TABLE_NAME';```                                                                                                                                                                                                                                                                                                                                                                                                                 |
| [PostgreSQL](https://www.postgresqltutorial.com/postgresql-indexes/postgresql-list-indexes/)                                           | ```SELECT * FROM pg_indexes WHERE tablename = 'TABLE_NAME';```                                                                                                                                                                                                                                                                                                                                                                                                                 |

### Additional Information

In addition to the above-mentioned points. Please keep note of the points below as well.

* If the user store is configured to use case-insensitive usernames with the databases like Oracle, PostgreSQL,
  SQLServer, the DBA should create the respective indexes with `LOWER(UM_USER_NAME)` functionality.
* If the running database is a case-sensitive database, the following indexes should be added to the database and the
  configurations mentioned after the third point should not be applied at all. Furthermore, please note that the
  following might not represent all the indexes required.

```
CREATE INDEX IDX_AT_CK_AU_LO ON IDN_OAUTH2_ACCESS_TOKEN(CONSUMER_KEY_ID, LOWER(AUTHZ_USER), TOKEN_STATE, USER_TYPE);
CREATE INDEX IDX_AT_TI_UD_LO ON IDN_OAUTH2_ACCESS_TOKEN(LOWER(AUTHZ_USER), TENANT_ID, TOKEN_STATE, USER_DOMAIN);
CREATE INDEX IDX_AT_AU_TID_UD_TS_CKID_LO ON IDN_OAUTH2_ACCESS_TOKEN(LOWER(AUTHZ_USER), TENANT_ID, USER_DOMAIN, TOKEN_STATE, CONSUMER_KEY_ID);
CREATE INDEX IDX_AT_AU_CKID_TS_UT_LO ON IDN_OAUTH2_ACCESS_TOKEN(LOWER(AUTHZ_USER), CONSUMER_KEY_ID, TOKEN_STATE, USER_TYPE);
CREATE INDEX IDX_AT_CIDAUTID_UD_TSH_TS_LO ON IDN_OAUTH2_ACCESS_TOKEN(CONSUMER_KEY_ID, LOWER(AUTHZ_USER), TENANT_ID, USER_DOMAIN, TOKEN_SCOPE_HASH, TOKEN_STATE);
CREATE INDEX IDX_AUTH_CODE_AU_TI_LO ON IDN_OAUTH2_AUTHORIZATION_CODE (LOWER(AUTHZ_USER), TENANT_ID, USER_DOMAIN, STATE);
CREATE INDEX IDX_AUTH_USER_UN_TID_DN_LO ON IDN_AUTH_USER (LOWER(USER_NAME), TENANT_ID, DOMAIN_NAME);
CREATE INDEX IDX_OCA_UM_TID_UD_APN_LO ON IDN_OAUTH_CONSUMER_APPS(LOWER(USERNAME),TENANT_ID,USER_DOMAIN, APP_NAME);
CREATE INDEX INDEX_IDN_USER_DK_LO_UNIQUE ON IDN_IDENTITY_USER_DATA (TENANT_ID, LOWER(USER_NAME), DATA_KEY);
CREATE INDEX INDEX_IDN_USER_LO_UNIQUE ON IDN_IDENTITY_USER_DATA (TENANT_ID, LOWER(USER_NAME));

CREATE INDEX IDX_UU_LO_UI_UUN_TI ON UM_USER(UM_ID,LOWER(UM_USER_NAME),UM_TENANT_ID);
CREATE INDEX INDEX_UM_USER_LO_UNIQUE ON UM_USER (LOWER(UM_USER_NAME), UM_TENANT_ID);
CREATE INDEX INDEX_UM_SYSTEM_USER_LO_UNIQUE ON UM_SYSTEM_USER (LOWER(UM_USER_NAME), UM_TENANT_ID);
CREATE INDEX INDEX_UM_ACC_MAPPING_LO_UNIQUE ON UM_ACCOUNT_MAPPING (LOWER(UM_USER_NAME), UM_TENANT_ID, UM_USER_STORE_DOMAIN, UM_ACC_LINK_ID);
CREATE INDEX INDEX_UM_HYBRID_UR_LO_UNIQUE ON UM_HYBRID_USER_ROLE (LOWER(UM_USER_NAME), UM_ROLE_ID, UM_TENANT_ID);
CREATE INDEX INDEX_UM_SYSTEM_UR_LO_UNIQUE ON UM_SYSTEM_USER_ROLE (LOWER(UM_USER_NAME), UM_ROLE_ID, UM_TENANT_ID);
```

* If the running database is a case-insensitive database such as MySQL or MSSQL, the following configuration should be
  added in order to avoid executing queries with the LOWER function. Please note that the configuration should be
  applied to all user stores that adhere as a case-insensitive database and the WSO2 IS should be restarted after
  applying the configurations.

**WSO2 IS < 5.9.0 Configurations**

Set the following property to false based on the user store in the user-mgt.xml file that is located in the _<IS_HOME>
/repository/conf_ directory.

```
<Property name="CaseInsensitiveUsername">false</Property>
<Property name="UseCaseSensitiveUsernameForCacheKeys">false</Property>
```

**WSO2 IS >= 5.9.0 Configurations**

Set the following property to false based on the user store in the deployment.toml file that is located in the _<
IS_HOME>
/repository/conf_ directory.

```
[user_store]
properties.CaseInsensitiveUsername = false
properties.UseCaseSensitiveUsernameForCacheKeys = false
```

* The usage of the indexes can change according to the business use cases. In such cases, based on the DBA suggestion,
  if such indexes have not any usage. it's safe to drop the same.

## Unsuitable Database Pooling Configurations

### Overview

Pooling configurations are applied in order to reduce the cost by maintaining a pool of open connections that can be
transferred from one database operation to another as needed. This way, we can spare the expense of having to open and
close a new connection for each operation the database requires to perform. You may now think “isn’t a single database
connection inexpensive?”. A single database connection is not expensive at all, but as things scale up like in the WSO2
IS, many problems can emerge.

When it comes to the WSO2 IS, it has the ability to read these pool configurations if they are defined in the required
configuration files. The following table provides a brief overview on the available configurations with regard to any
version of the WSO2 IS.

| Configuration Name | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | 
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| maxActive          | The maximum number of active connections that can be allocated at the same time from this pool. Enter any negative value to denote an unlimited number of active connections.                                                                                                                                                                                                                                                                                                                                                                                                               |
| maxWait            | The maximum number of milliseconds that the pool will wait (when there are no available connections) for a connection to be returned before throwing an exception. You can enter zero or a negative value to wait indefinitely.                                                                                                                                                                                                                                                                                                                                                             |
| minIdle            | The minimum number of active connections that can remain idle in the pool without extra ones being created, or enter zero to create none.                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 
| testOnBorrow       | Whether objects will be validated before being borrowed from the pool. If the object fails to validate, it will be dropped from the pool, and another attempt will be made to borrow another.                                                                                                                                                                                                                                                                                                                                                                                               |
| defaultAutoCommit  | Whether to commit database changes automatically or not. This property is not applicable to the Carbon database in WSO2 products because auto committing is usually handled at the code level, i.e., the default auto commit configuration specified for the RDBMS driver will be effective instead of this property element. Typically, auto committing is enabled for RDBMS drivers by default. When auto committing is enabled, each SQL statement will be committed to the database as an individual transaction, as opposed to committing multiple statements as a single transaction. |
| validationInterval | The indication to avoid excess validation, and only run validation at the most, at this frequency (time in milliseconds). If a connection is due for validation, but has been validated previously within this interval, it will not be validated again.                                                                                                                                                                                                                                                                                                                                    | 
| commitOnReturn     | If `defaultAutoCommit = false`, then you can set `commitOnReturn = true`, so that the pool can complete the transaction by calling the commit on the connection as it is returned to the pool. However, If `rollbackOnReturn = true` then this attribute is ignored. The default value is false.                                                                                                                                                                                                                                                                                            |
| rollbackOnReturn   | If `defaultAutoCommit = false`, then you can set `rollbackOnReturn = true` so that the pool can terminate the transaction by calling rollback on the connection as it is returned to the pool. The default value is false.                                                                                                                                                                                                                                                                                                                                                                  |

### Solution

To ensure that the WSO2 IS is not experiencing any performance issues on the fact that database pooling configurations
are unavailable, please make sure to add them according to the type of the database and the type/version of the
database. The configurations mentioned below applied the pooling configurations to the identity and the shared database.

**WSO2 IS < 5.9.0 Configurations**

Add the required configurations to the master-datasources.xml file located in the path _<IS_HOME>
/repository/conf/datasources_. A sample configuration is as follows.

```
<datasource>
   <name>WSO2_CARBON_DB</name>
   <description>The datasource used for registry and user manager</description>
   <jndiConfig>
      <name>jdbc/WSO2CarbonDB</name>
   </jndiConfig>
   <definition type="RDBMS">
      <configuration>
         <url>jdbc:mysql://localhost:3306/regdb</url>
         <username>regadmin</username>
         <password>regadmin</password>
         <driverClassName>com.mysql.jdbc.Driver</driverClassName>
         <maxActive>80</maxActive>
         <maxWait>60000</maxWait>
         <minIdle>5</minIdle>
         <testOnBorrow>true</testOnBorrow>
         <validationQuery>SELECT 1</validationQuery>
         <validationInterval>30000</validationInterval>
         <defaultAutoCommit>false</defaultAutoCommit>
      </configuration>
   </definition>
</datasource>
```

**WSO2 IS >= 5.9.0 Configurations**

Add the required configurations to the newly introduced deployment.toml file located in the path _<IS_HOME>
/repository/conf_. A sample configuration is as follows.

```
[database.identity_db.pool_options]
maxActive = "80"
maxWait = "60000"
minIdle = "5"
testOnBorrow = true
validationQuery="SELECT 1"
validationInterval="30000"
defaultAutoCommit=false

[database.shared_db.pool_options]
maxActive = "80"
maxWait = "60000"
minIdle = "5"
testOnBorrow = true
validationQuery="SELECT 1"
validationInterval="30000"
defaultAutoCommit=false
```

Please make sure to follow the official documentation based on the WSO2 IS/database type/version that you are using for
further information regarding this matter. You can find the documentation for the WSO2 IS 5.11.0
at [link](https://is.docs.wso2.com/en/latest/setup/working-with-databases/) and for the versions below the WSO2 IS 5.9.0
at [link](https://docs.wso2.com/display/ADMIN44x/Changing+the+Carbon+Database).

## Already Enabled Debug Logs

### Overview

Debug logs are useful resources that could help us when there is an error/problem in the WSO2 IS that requires further
information. By default, we have not enabled debug logs in the product, and it is done to avoid the occurrences of
unwanted performance issues. Even though we have not enabled debug logs, we have enabled info level logs by default as a
mechanism to monitor the WSO2 IS. Furthermore, when debug logs are enabled a large number of logs are written to the log
files hence it is not recommended to enable debug logs in production environments unless it is necessary.

### Procedure

During support cases, the customer success engineers will provide information to enable logs. However, these changes
should be reverted to the previous state as soon as the analysis is completed.

### Default Log4j Property Files

The default log4j.properties/log4j2.properties files for the relevant WSO2 IS versions are linked in the table below.

| WSO2 IS Version | Default log4j/log4j2.properties Location                                               | 
|-----------------|----------------------------------------------------------------------------------------|
| 5.2.0           | https://drive.google.com/drive/folders/12mPKCXz20D_A7fwWPllPO9b8oSHSW2I6?usp=sharing   |
| 5.3.0           | https://drive.google.com/drive/folders/16KoaSPxS2DcCfGUcJnO-VgL0YKL94HCU?usp=sharing   |
| 5.4.0           | https://drive.google.com/drive/folders/1B_mPrKw6BuqJ1dunosseqPHLbHI-idVi?usp=sharing   | 
| 5.5.0           | https://drive.google.com/drive/folders/1fvWZPbrt6QcVUMDHpArwaF6OeVkZb6fo?usp=sharing   |
| 5.6.0           | https://drive.google.com/drive/folders/1pjbGh_4dveIcyqDLiO-PvDCqhzI3bo11?usp=sharing   |
| 5.7.0           | https://drive.google.com/drive/folders/19__OKpysUqfWKGC9T_Oe8wl1qNFZIvEO?usp=sharing   | 
| 5.8.0           | https://drive.google.com/drive/folders/1pwzZKmQTUbSTg44HiqUNC27YUHNn0ViW?usp=sharing   |
| 5.9.0           | https://drive.google.com/drive/folders/13V3IPDwi3ZTHSAYJeM8wjFK7ByO0tYMY?usp=sharing   |
| 5.10.0          | https://drive.google.com/drive/folders/1InRAZNQwW1M050rEu1pxk9bD47yaIvou?usp=sharing   |
| 5.11.0          | https://drive.google.com/drive/folders/1zVedcsjYVuRWCv-QvrtkAet6Vqv3HfZH?usp=sharing   |

## Calling External Services

### Overview

The WSO2 IS is designed to be extensible hence it allows calling external services in order to capture vital information
that are required for the user. While calling these external services, there could be situations where the service might
be slow due to some unknown reason because of this the WSO2 IS will also get slow since it is waiting for the response
from the external service. Take the following as an example.

Let's say that you have a custom Just-In-Time (JIT) provisioning handler that calls an API service external to the WSO2
IS in order to retrieve claims for a user and update them in the WSO2 IS. Furthermore, let’s also consider that the
custom source code is not written to handle this scenario asynchronously. In this scenario, if the API is slow to
respond, the WSO2 IS will wait until a response is received hence there will be a few performance degradations. Another
common example when it comes to the WSO2 IS would be when there is a delay in the federated identity provider’s token
endpoint that will in turn cause the WSO2 ISto slow down. If this is the case, it would be better to inspect what is
wrong with the federated identity provider before relying on the WSO2 IS for debugging.

### Solution

The ideal solution would be to either call the external service asynchronously or add retry attempts and fail the
scenario after a certain threshold. This will in turn reduce the performance degradation to a certain extent.

## LDAP/LDAPS Connection Pool Issues

### Overview and Current Solution

The WSO2 IS OOTB has an LDAP as the user store where connection pooling is not enabled. For more information regarding
connection pooling, please refer to
the [external documentation](https://www.cockroachlabs.com/blog/what-is-connection-pooling/). Due to these kinds of
issues, there can be instances where you might face slowness in a few scenarios like during user login. The
recommendation is to allow connection pooling as mentioned in
the [official documentation](https://is.docs.wso2.com/en/latest/setup/performance-tuning-recommendations/#pooling-ldaps-connections)
. However, the official documentation is a bit wayward hence, the proper configurations should be as follows.

**WSO2 IS < 5.9.0 Configurations**

Add the required configurations to the user-mgt.xml file located in the directory _<IS_HOME>/repository/conf_. A sample
configuration is as follows.

```
<UserStoreManager class="org.wso2.carbon.user.core.ldap.ReadWriteLDAPUserStoreManager">
   <Property name="ConnectionPoolingEnabled">true</Property>
</UserStoreManager>
```

**WSO2 IS >= 5.9.0 Configurations**

Set the following property to true based on the user store in the deployment.toml file that is located in the _<IS_HOME>
/repository/conf_ directory.

```
[user_store]
connection_pooling_enabled = true
```

Please note that if you are using a secondary user store, the configuration can be changed from the Management Console
itself or from the secondary user store XML file located in the _<IS_HOME>/repository/deployment/server/userstores_
directory. The XML property is the same as the one mentioned in the WSO2 IS < 5.9.0 section.

In addition to the above configuration the following parameter should be added to the wso2server.sh/wso2server.bat file
to define the protocol that is being used. The following parameter will allow both plain and SSL connections to be
pooled.

```
-Dcom.sun.jndi.ldap.connect.pool.protocol=”plain ssl”
```

### Additional Configurations

In addition to the parameter `-Dcom.sun.jndi.ldap.connect.pool.protocol=”plain ssl”`, there are a lot more parameters
that can be used to tune the connection pooling configurations such as the following.

* Initial pool size = `-Dcom.sun.jndi.ldap.connect.pool.initsize=1`
* Preferred pool size = `-Dcom.sun.jndi.ldap.connect.pool.prefsize=10`
* Maximum pool size = `-Dcom.sun.jndi.ldap.connect.pool.maxsize=0`

You can refer to the external documentation
at [Oracle](https://docs.oracle.com/javase/jndi/tutorial/ldap/connect/config.html)
or [Atlassian](https://confluence.atlassian.com/doc/configuring-the-ldap-connection-pool-229838451.html) for more
information regarding the same.

## LDAP Referrals

### Overview

LDAP referrals provide a reference to an alternate location in which an LDAP request may be processed. The WSO2 IS
supports this functionality Out Of The Box (OOTB) and it can be configured within the user store properties. A referral
is useful for allowing an object to be identified by different names. Referrals can be used to accommodate the namespace
changes and mergers that are inevitable as organizations evolve. Furthermore, they allow directory administrators to set
up search locations for collecting results from multiple servers as mentioned
in [Oracle documentation](https://docs.oracle.com/javase/jndi/tutorial/ldap/referral/overview.html). Even if this
feature is useful, there are several issues embedded while using it. The following points represent the main issues.

* If an invalid referral is pointed for the LDAP, there would be error logs printed in the console saying that the
  connection could not be established with the said referral.
* When the pointed referral is slow to respond, the WSO2 IS will also become slow since it is waiting for a response
  from the referral location.

### Solution

The recommended approach to overcome this problem is to let the WSO2 IS ignore these referral locations so that no
requests would get redirected to those referrals. To do that the following configuration should be added to all the LDAP
user stores.

**WSO2 IS < 5.9.0 Configurations**

Add the required configurations to the user-mgt.xml file located in the directory _<IS_HOME>/repository/conf_. A sample
configuration is as follows.

```
<UserStoreManager class="org.wso2.carbon.user.core.ldap.ReadWriteLDAPUserStoreManager">
   <Property name="Referral">ignore</Property>
</UserStoreManager>
```

**WSO2 IS >= 5.9.0 Configurations**

Add the required configurations on the user store in the deployment.toml file that is located in the <IS_HOME>
/repository/conf directory.

```
[user_store]
referral = “ignore”
```

Please note that if you are using a secondary user store, the configuration can be changed from the Management Console
itself or from the secondary user store XML file located in the _<IS_HOME>/repository/deployment/server/userstores_
directory. The XML property is the same as the one mentioned in the WSO2 IS < 5.9.0 Configurations section.

## Insufficient Resources to Run the Server

### Overview

As almost all the software applications available in the market, the WSO2 IS has a set of prerequisites that are
mandatory for it to be functionality at the level best. If these prerequisites are not noted beforehand, it would cause
several performance issues such as a slow server startup, etc… On top of that, there will be some occasions where you
will need to redo the deployment in order to adhere to these prerequisites, which makes this a tedious effort if your
current deployment is up and running.

As an example for the above paragraph, let's say that you are using a single Central Processing Unit (CPU) core for the
WSO2 IS container/Virtual Machine (VM) while the official documentation suggests that a 4 vCPUs (x86_64 Architecture) is
required. In this scenario, the startup of the WSO2 IS will be really slow and these issues cannot be resolved from the
WSO2 IS’s perspective. With that, you will need to reconsider and redo deployment to increase the number of CPUs which
is a tedious task.

There are several aspects of prerequisites required for the WSO2 IS, and we will be talking these through in the
Solution
section.

### Solution

As said in the previous section, there are several prerequisites required for the WSO2 IS and the following bullet
points represent the mandatory ones from those.

* Memory. The memory recommendations given in the official documentation can change based on the expected concurrency
  and performance.
* Storage. There is no recommendation for the space required for logs since it can change based on the amount of logs
  created.
* Operating System (OS). The WSO2 IS requires an operating system that is compliant with JDK.
* Databases. All WSO2 Carbon-based products are made compatible with most of the common DBMSs around. The embedded H2
  database is the default database, and it is only suitable for development and testing.
* User Stores. The user store is the database where information about the users and user roles is stored. By default,
  the WSO2 IS has an LDAP user store.
* Java SE Development Kit (JDK). Since the WSO2 IS is a Java application, to launch the product JDK is required.
* Web Browser. The web browser is required to access the Management Console and test out applications as well. The Web
  Browser must be JavaScript enabled to take full advantage of the Management Console.
* Build Automation Tools. These are required if you are building components related to the product or any
  customizations.

**Official Documentation for the Recommendations based on the above criterias**

| WSO2 IS Version | Default log4j/log4j2.properties Location                                                                                                                                                              | 
|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 5.2.0           | [Installation Prerequisites](https://docs.wso2.com/display/IS520/Installation+Prerequisites), [Environment Compatibility](https://docs.wso2.com/display/compatibility/Compatibility+of+WSO2+Products) |
| 5.3.0           | [Installation Prerequisites](https://docs.wso2.com/display/IS530/Installation+Prerequisites), [Environment Compatibility](https://docs.wso2.com/display/compatibility/Compatibility+of+WSO2+Products) |
| 5.4.0           | [Installation Prerequisites](https://docs.wso2.com/display/IS540/Installation+Prerequisites), [Environment Compatibility](https://docs.wso2.com/display/compatibility/Compatibility+of+WSO2+Products) | 
| 5.5.0           | [Installation Prerequisites](https://docs.wso2.com/display/IS550/Installation+Prerequisites), [Environment Compatibility](https://docs.wso2.com/display/compatibility/Compatibility+of+WSO2+Products) |
| 5.6.0           | [Installation Prerequisites](https://docs.wso2.com/display/IS560/Installation+Prerequisites), [Environment Compatibility](https://docs.wso2.com/display/compatibility/Compatibility+of+WSO2+Products) |
| 5.7.0           | [Installation Prerequisites](https://docs.wso2.com/display/IS570/Installation+Prerequisites), [Environment Compatibility](https://docs.wso2.com/display/compatibility/Compatibility+of+WSO2+Products) | 
| 5.8.0           | [Installation Prerequisites](https://docs.wso2.com/display/IS580/Installation+Prerequisites), [Environment Compatibility](https://docs.wso2.com/display/compatibility/Compatibility+of+WSO2+Products) |
| 5.9.0           | [Installation Prerequisites](https://is.docs.wso2.com/en/5.9.0/setup/installation-prerequisites/), [Environment Compatibility](https://is.docs.wso2.com/en/5.9.0/setup/environment-compatibility/)    |
| 5.10.0          | [Installation Prerequisites](https://is.docs.wso2.com/en/5.10.0/setup/installation-prerequisites/), [Environment Compatibility](https://is.docs.wso2.com/en/5.10.0/setup/environment-compatibility/)  |
| 5.11.0          | [Installation Prerequisites](https://is.docs.wso2.com/en/5.11.0/setup/installation-prerequisites/), [Environment Compatibility](https://is.docs.wso2.com/en/5.11.0/setup/environment-compatibility/)  |

## Network Delays between Clients and the WSO2 IS

### Overview

There are times which the clients connecting to the WSO2 IS are not responding in the expected time period, this could
be most probably due to network delays in connecting to the WSO2 IS.

For example, let’s say that you are accessing the Management Console’s login page, and it is taking a long time to
respond. However, you are not sure whether there is a latency in the network or in the browsers side. Another example
would be when you are running load testing using an external tool such as Apache JMeter where the tool seems to be
taking a long time.

From the WSO2 IS’s point of view, there are no solutions to overcome this situation. The only possibility is to do an
investigation to check whether the relevant client's request is received by the WSO2 IS end so that investigation can be
isolated properly.

### Investigation

In order to explain the investigation process, I would be taking the first example provided in the previous subsection.
As said earlier, let’s say that you are trying to access the Management Console from the
URL https://localhost:9443/carbon/admin/login.jsp, but it is taking a long time to load. To check and confirm that there
is no latency issue with the WSO2 IS’s end, you can check the log file named http_access.log that is located in the _<
IS_HOME>/repository/logs_ directory. You have to check whether the relevant request is received by the WSO2 IS, if it is
not received, it confirms that there is a network delay at the client’s end. Based on the scenario, if there is no
access log similar to the following line according to the time of the Management Console access, it confirms that there
is no issue with the WSO2 IS hence further analysis steps should be applied in the client’s end to find the root cause
of the issue.

```
127.0.0.1 - - [11/Nov/2021:21:11:59 +0530] GET /carbon/admin/login.jsp HTTP/1.1 200 3431 - Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:94.0) Gecko/20100101 Firefox/94.0 1.705
```

The explanations for the above access log’s attributes are as follows.

* `127.0.0.1` - The first part of the access log line depicts the IP address from which the WSO2 IS received the
  request.
* `[11/Nov/2021:21:11:59 +0530]` - The second part of the access log line displays the date and the time at which the
  WSO2 IS received the request.
* `GET` - The third part of the access log line presents the type of request which the WSO2 IS has received.
* `/carbon/admin/login.jsp` - The fourth part of the access log line provides the request path which the WSO2 IS has
  received.
* `HTTP/1.1 200 3431` - The fifth part of the access log line shows the status of the request which the WSO2 IS has
  received.
* `Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:94.0) Gecko/20100101 Firefox/94.0` - The sixth part of the access log line
  exhibits the user agent who sent the request to the WSO2 IS.
* `1.705` - The final part of the access log line unveils the time taken (in seconds) by the WSO2 IS to process the
  request and send back the response.

If the above-mentioned log is available in the http_access.log and the time taken to process the request and send the
response back from the WSO2 IS is large, you might need to further look into the latency issue from the WSO2 IS’s end.
To do that, you can follow the stepwise debugging topics mentioned in the “_Performance Troubleshooting Methods_”
section.

In WSO2 IS versions that are lower than 5.9.0, the above said log pattern is not available by default hence please
follow the steps below to add the pattern to the product.

* To add the new pattern to HTTP access logs, please open the catalina-server.xml file located in the _<IS_HOME>
  /repository/conf/tomcat_ directory.
* Comment out the existing "AccessLogValve" and add the valve with a new log pattern (last one) as below. Save the file
  and restart the WSO2 IS.

```
<!--Valve className="org.apache.catalina.valves.AccessLogValve" directory="${carbon.home}/repository/logs"
      prefix="http_access_" suffix=".log"
      pattern="combined"/-->
```

```
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="${carbon.home}/repository/logs"
      prefix="http_access_" suffix=".log"
      pattern="%h %l %u %t %r %s %b %{Referer}i %{User-Agent}i [%T]"/>
```

## Performance Troubleshooting Methods

### Slow Query Reports

#### Overview

Slow query reports are generally enabled to check whether there are slow database level queries. When you enable slow
query reports, you can observe that the Structured Query Language (SQL) statements which are slower than the threshold
mentioned in the slow query report configuration would get printed in the console or in the wso2carbon.log file. With
the help of the slow query report, it is possible to improve those queries in order for them to be executed a bit
faster. An example for it would be adding indexes to a table where the query execution is slow.

#### How to Enable Slow Query Reports

In order to generate the slow query report, you need to add the following configurations based on the version of the
WSO2 IS which you are using.

**WSO2 IS < 5.9.0 Configurations**

Add the required configurations to the master-datasources.xml file located in the path _<IS_HOME>
/repository/conf/datasources_. A sample configuration is as follows.

```
<UserStoreManager class="org.wso2.carbon.user.core.jdbc.JDBCUserStoreManage">
   <jdbcInterceptors>SlowQueryReport(threshold=20)</jdbcInterceptors>
</UserStoreManager>
```

**WSO2 IS >= 5.9.0 Configurations**

Add the required configurations into the deployment.toml file that is located in the _<IS_HOME>/repository/conf_
directory. A sample configuration is as follows.

```
[database.shared_db.pool_options]
jdbcInterceptors = "org.apache.tomcat.jdbc.pool.interceptor.SlowQueryReport(threshold=20);"

[database.identity_db.pool_options]
jdbcInterceptors = "org.apache.tomcat.jdbc.pool.interceptor.SlowQueryReport(threshold=20);"
```

Please note that if you are using secondary user stores, the configurations mentioned in the WSO2 IS < 5.9.0 section
should be applied to the secondary user store XML file located in the _<IS_HOME>
/repository/deployment/server/userstores_ directory. The XML property differs from the one mentioned in the WSO2 IS <
5.9.0 Configurations section hence please use the below sample configuration for the secondary user stores.

```
<UserStoreManager class="org.wso2.carbon.user.core.jdbc.JDBCUserStoreManage">
   <Property name="jdbcInterceptors">SlowQueryReport(threshold=20)</Property>
</UserStoreManager>
```

#### How to Analyze Slow Query Reports

Since most of the slow queries are already printed out into the console/wso2carbon.log file, it is a matter of filtering
out the slower queries and settling into a decision on how to improve the execution time of those queries. The decisions
can range from adding indexes to improving the pooling configurations.

### Correlation Logs

#### Overview

Correlation logs help to find the time taken for LDAP and JDBC database calls. This process helps in tracking down any
slowness caused by database calls in an instance. The request calls and response calls are correlated via a unique ID
named “correlation ID” that is sent in the request call.

#### How to Enable and Analyze Correlation Logs

The process of enabling and analyzing correlation logs is thoroughly explained in the WSO2 IS’s official documentation
site. The following table contains product version specific links to the official documentation.

| WSO2 IS Version | Default log4j/log4j2.properties Location                                     | 
|-----------------|------------------------------------------------------------------------------|
| 5.2.0           | https://docs.wso2.com/display/IS520/Working+with+Product+Observability       |
| 5.3.0           | https://docs.wso2.com/display/IS530/Working+with+Product+Observability       |
| 5.4.0           | https://docs.wso2.com/display/IS540/Working+with+Product+Observability       | 
| 5.5.0           | https://docs.wso2.com/display/IS550/Working+with+Product+Observability       |
| 5.6.0           | https://docs.wso2.com/display/IS560/Working+with+Product+Observability       |
| 5.7.0           | https://docs.wso2.com/display/IS570/Working+with+Product+Observability       | 
| 5.8.0           | https://docs.wso2.com/display/IS580/Working+with+Product+Observability       |
| 5.9.0           | https://is.docs.wso2.com/en/5.9.0/setup/working-with-product-observability/  |
| 5.10.0          | https://is.docs.wso2.com/en/5.10.0/setup/working-with-product-observability/ |
| 5.11.0          | https://is.docs.wso2.com/en/5.11.0/setup/working-with-product-observability/ |

Please take note of the following points when correlation log configurations are added to the WSO2 IS.

* The correlation log will be generated in a folder named ${instance.log} inside the _<IS_HOME>/repository/logs_
  directory. If you want to change this location you can modify the following values based on the product version.

    * WSO2 IS < 5.9.0 - `log4j.appender.CORRELATION.File=${carbon.home}/repository/logs/${instance.log}/correlation.log`
    * WSO2 IS >= 5.9.0
        - `appender.CORRELATION.fileName =${sys:carbon.home}/repository/logs/${instance.log}/ccorrelation.log
          appender.CORRELATION.filePattern =${sys:carbon.home}/repository/logs/correlation-%d{MM-dd-yyyy}.%i.log`

* The file size mentioned in the official documentation is 10MB but this is not enough for a production environment
  since this will only log 200MB of data that is totaled by the generation of 20 files containing 20 MB each. The
  recommendation is to increase the file size to 100MB but please keep note that this will require 2000MB of space that
  is totaled by the generation of 20 files containing 100 MB each. The configuration changes required based on the
  product version.

    * WSO2 IS < 5.9.0 - `log4j.appender.CORRELATION.MaxFileSize=100MB`
    * WSO2 IS >= 5.9.0 - `appender.CORRELATION.policies.size.size=100MB`

### Thread and Heap Dumps

#### Overview

##### Thread Dump

A thread dump is an overview of the state of all the threads within a Java process. The state of each thread is
represented with a stack trace, showing the content of a thread's stack as mentioned in
the [external documentation](https://www.baeldung.com/java-thread-dump). A thread dump is useful for diagnosing
problems, as it displays the thread's activity. A thread dump can be used for analyzing performance degradation (
slowness), finding the root cause of an application becoming unresponsive, or for diagnosing deadlock situations.

#### Heap Dump

A heap dump is an overview of all the objects in the Java Virtual Machine (JVM) heap at a given point in time. The Java
Virtual Machine (JVM) software is responsible for allocating memory for objects from the heap for the use of all class
instances and arrays as mentioned in
the [Oracle external documentation](https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/heapdump.html). The
heap memory is reclaimed by the garbage collector (GC) when it thinks that an object is no longer needed and when there
are no references to the object. By analyzing the heap it is possible to locate where objects are created and find the
references to those objects in the source. However, it is rare to observe that there is an issue with the GC process by
nature. Furthermore, whenever there are slowness issues, out of memory (OOM) issues, deadlock issues it would be ideal
to generate both thread and heap dumps to find the root cause of them.

#### How to Extract Thread/Heap Dumps

##### Extract Thread Dump

A thread dump can be extracted using the
shell [script tool](https://github.com/wso2-cs/troubleshoot-kit/tree/master/scripts-and-commands/thread-dump) that is
already available in the wso2-cs GitHub organizations’s
troubleshoot-kit [repository](https://github.com/wso2-cs/troubleshoot-kit). The instructions are clearly provided in the
README.md file that is available in the same location. For the ease of understanding, you can keep note of the following
information.

* The Process Identifier is referred to as PID, the No:of Thread Dumps is referred to as NoTD, the Thread Dump Interval
  is referred to as TDI. Furthermore, the TDI contains the following time units.

    * s for seconds (the default)
    * m for minutes.
    * h for hours.
    * d for days.

* The tool can be executed with the below commands depending on the Operating System type.

    * Linux - `sh thread-analyze.sh <PID> <NoTD> <TDI>`
    * Windows - `thread-analyze-windows.bat <PID>`

* The script available for windows will extract 4 thread dumps within a one-minute interval while you can define them
  according to the scenario in the Linux script.

If the above-mentioned scripts are not working, please use one of the methods mentioned in
the [external blog]( https://www.baeldung.com/java-thread-dump).

##### Extract Thread Dump

By default, the WSO2 IS generates heap dumps whenever the server goes OOM. However, it is possible to generate heap
dumps according to your requirements as well. The blogs at [Baeldung](https://www.baeldung.com/java-heap-dump-capture)
and [IBM](https://www.ibm.com/docs/en/was-nd/8.5.5?topic=generation-generating-heap-dumps-manually) represent a few
methods of extracting heap dumps, hence it is possible for you to achieve the requirement using one of those mentioned
methods.

#### How to Analyze Thread/Heap Dumps

##### Analyze Thread Dump

The most important thing to do while analyzing thread dumps is to filter out the results. If you have generated the
thread dump from the tool that was available in the GitHub troubleshoot-kit repository, there is
another [tool](https://github.com/wso2-cs/troubleshoot-kit/tree/master/scripts-and-commands/thread-analysis) that helps
in analyzing the thread dump by sorting out the OS threads by the CPU percentage and mapping them with the corresponding
Java thread. However, there are also external tools that you can use to analyze the thread dumps as mentioned in
the [external blog](https://www.baeldung.com/java-analyze-thread-dumps). Furthermore, the blog explains how to analyze
certain types of issues through thread dumps.

##### Analyze Heap Dump

As said for the thread dumps, the first thing when it comes to analyzing heap dumps is to narrow down the search space.
For this, there are several tools that you can use but the Eclipse Memory Analyzer (EMA) is preferred by the community
hence it could be the perfect tool for this task. The blogs
at [DZone](https://dzone.com/articles/java-heap-dump-analyzer-1)
and [Atlassian](https://confluence.atlassian.com/jirakb/how-to-analyze-performance-diagnostics-thread-dumps-heap-dumps-garbage-collection-logs-973502191.html)
provide an overview on how to use that tool to analyze heap dumps.

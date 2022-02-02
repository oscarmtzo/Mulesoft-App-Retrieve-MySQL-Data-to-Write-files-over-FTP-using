# Mule app to query MySQL database and write output data to Excel file using FTP Write connector and FileZilla Server

This Mule application allows to easy interact with a MySQL database which to retrieve data about actors, their first name, last name and last update of their data, to later write it to a new local file.

### Previous MySQL configuration
- Use database `sakila` from *Samples and Examples* MySQL installer files.
- Once open the connection of the database on *MySQL Workbench*, create a user to grant access for CRUD database operations.
    ```sql
    -- Creation of a test user to use at the configuration XML for Mule application before pushing to github
    CREATE USER 'test'@'localhost' IDENTIFIED BY 'newpassword';
    -- Grant access to our new user for all operations with "SELECT"
    GRANT SELECT ON * . * TO 'test'@'localhost';
    -- Grant access to our new user for all operations with "DELETE"
    GRANT DELETE ON * . * TO 'test'@'localhost';
    ```
### Access from FTP - Write connector
- Create a directory  `./retrieved-data` inside the project directory.
1. Instalation of <a src="https://filezilla-project.org/download.php?type=server">FileZilla Server</a> - as an FTP server.
    ``port: 14148`
    - **Add a user**, *Server -> Configure -> Users -> Add*
        - `ftp_test_user` is the name for the testing user for the mule application to access server files.
        - Set Credentials to:
        ```
        option: Require a password to log in.
        password: ftp_test_user 
        ```
    - **Add a directory to access from server**, `./ftp-files-retrieved-data`, these directory will be the storage area for the FTP Mule application to write data located at: `./ftp-files-retrieved-data/actorInfo.xlsx` that Excel file will be created when running the Mule app inside of `./ftp-files-retrieved-data`.
        - Add 1 mount points for FileZilla Server configuration test user directories: 
            -  *Server -> Configure -> Users -> ftp_test_user ->Mount points -> Add*
            1. Set Virtual path to `/retrieved-data`, and Native Path to `./ftp-files-retrieved-data`.
               
## Mule event connectors
- **Scheduler** connector, triggers the execution of the mule app to be done on a establish interal of time.
```xml
<scheduler doc:name="Scheduler" doc:id="d8ddbef1-62a9-4d79-95be-96d07ca760ef" >
    <scheduling-strategy >
        <fixed-frequency frequency="1" timeUnit="MINUTES"/>
    </scheduling-strategy>
</scheduler>
```
- Scheduling Strategy:

| Frecuency | Start delay | Time unit | Mule XML description | 
| --- | --- | --- | --- |
| 1 | 0 | Minutes | Every 1 minute trgiggers the flow execution.  |


- **Database - Select** connector, retrieves data from `sakila` database, also includes the configuration to retrieve data from an specific table like `actors`.

```xml
<db:select doc:id="b08d5543-fa6d-4f5a-8193-84f859faec7f" config-ref="Database_Config">
    <db:sql ><![CDATA[SELECT * FROM actor]]></db:sql>
</db:select>
```
| config-ref | SQL Operation | MySQL table |Mule XML description | 
| --- | --- | --- | --- |
| is the reference to global configuration which is set to  Database_Config | SELECT | actor | Retrieve all records from database `sakila` table `actor`; Database_Config: port: 3306, host: localhost, user: test, password: newpassword, database: sakila, MySQL JDBC Driver  |

- **Transform Message** connector, allow us to use *DataWeave 2.0 transformation language* to parse Java - array hash Map to CSV file comma separated Values file.
```xml
<ee:transform doc:name="Transform Message" doc:id="79133dea-acde-4265-a5f9-45c4f2fb4279" >
    <ee:message >
        <ee:set-payload ><![CDATA[%dw 2.0
output application/csv
---
payload]]></ee:set-payload>
    </ee:message>
</ee:transform>
```
| Payload typeOf() | input data | output data |
| --- | --- | --- |
| Array<Object> | application/java | application/csv |

- **FTP - Write** connector, writes data could be payload or metadata retrieved, for this purpose is written to  a new CSV file located at `./ftp-files-retrieved-data`.

```xml
<ftp:write doc:id="38149880-bb8f-4cc9-9f8a-9d6e0b6b43d6" config-ref="FTP_Config" path="actorsInfo"/>
```

| config-ref | input data | output data | path |
| --- | --- | --- | --- |
| ``FTP_config`` | ``application/java`` | ``application/csv`` | ``actorsInfo.xlsx`` *sets payload  or retrieved data to a new xlsx file to be created if does not exists* |

*FTP_Config*
| Working Directory | Host | Port | username | password |
| --- | --- | --- | --- | --- |
| retrieved-data *Is the virtual directory configured at FileZilla server* | localhost | 21 | ftp_test_user | ftp_test_user |

*Result: multiple records wirtten on the new `./ftp-files-retrieved-data/actorsInfo.xlsx` as shown here:*
|id	|firstName | lastName |	lastUpdate|
|---	|--- | --- |	--- |
|1|	PENELOPE|	GUINESS |2006-02-15T04:34:33|
|2 |	NICK |	WAHLBERG	|2006-02-15T04:34:33|
|... |	... |	...	|...|
|... |	... |	...	|...|
| 200 |	THORA |	TEMPLE |	2006-02-15T04:34:33 |
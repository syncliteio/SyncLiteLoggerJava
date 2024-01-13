# SyncLite Platform

SyncLite is a no-code, real-time, relational data consolidation platform empowering developers to rapidly build data-intensive applications for desktops, edge devices, smartphones, with the capability to enable in-app data management, in-app analytics and perform real-time data consolidation from numerous application instances into one or more databases, data warehouses, or data lakes of your choice.

{Edge/Desktop/Phone Apps} + {SyncLite Logger} ------> {Staging Storage} ------> { SyncLite Consolidator} -----> {Destination DB/DW/DataLakes}

SyncLite is scalable, secure, extensible, fault-tolerant, enabling a wide range of use-cases, including rapidly building smart resource monitors, building native SQL (hot) data stores on top of cloud hosted databases, building data mesh architectures, database migration/replication pipelines, deploying pluggable IoT data stacks, enabling cloud databases at the edge, creating OLTP + OLAP solutions, setting up software telemetry pipelines, among others.

For more details, check out 
Website : https://www.synclite.io
YouTube : https://www.youtube.com/@syncliteplatform


# SyncLite Components

SyncLite Platform comprises of two components : SyncLite Logger and SyncLite Consolidator.

1. SyncLite Logger : SyncLite Logger is a jar file built on top of SQLite, providing a SQL interface over JDBC for user applications, enabling them for in-app data management/analytics while logging all the SQL transactional activity into log files and shipping them to one of more configured staging storages like SFTP/S3/MinIO/Kafka/GoogleDrive/MSOneDrive/NFS etc. 

2. SyncLite Consolidator : SyncLite Consolidator is a Java application deployed on an on-premise host or a cloud VM is configured to scan thousands of SyncLite devices/databases and their logs continously from the configured staging storage which are uploaded by numerous edge/desktop applications, performs real-time data replication/consolidation into one or more configured databases, data warehouses or data lakes of user's choice.
   
# Using SyncLite Logger

This repository has been created to distribute the SyncLite logger jar file as updated in src/main/resources. You can use the following maven dependency in your edge/desktop applications for creating and operating edge databases/devices.

```
<dependency>
    <groupId>io.synclite</groupId>
    <artifactId>synclite-logger</artifactId>
    <version>2024.01.13</version>
</dependency>
```

# Configuration File

Refer src/main/resources/synclite_logger.conf file for all available configuration options for SyncLite Logger. Refer "SyncLite Logger Configuration" section in the documentation at https://www.synclite.io/resources/documentation for more details about all configuration options. 

# Application Code Samples

SyncLite Platform allows applications to create three types of devices:

1. Transactional Device : Transcational device supports all database operations as supported by SQLite and performs transactional logging of all the DDL and DML operations performed by the application. It empowers developers to build use cases such as native SQL (hot) hot data stores, SQL application caches, edge enablement of cloud databases, building OLTP + OLAP solutions etc.

## Java   
```
package testApp;

import java.nio.file.Path;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import io.synclite.logger.*;


public class TestTransactionalDevice {
	
	public static Path syncLiteDBPath;
	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.Transactional");
		Path dbPath = syncLiteDBPath.resolve("t.db");
		SyncLite.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
	}	
	
	public void myAppBusinessLogic() throws SQLException {
		//
		//Some application business logic
		//
		//Perform some database operations		
		try (Connection conn = DriverManager.getConnection("jdbc:synclite:" + syncLiteDBPath.resolve("t.db"))) {
			try (Statement stmt = conn.createStatement()) { 
				//Example of executing a DDL : CREATE TABLE. 
				//You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
				stmt.execute("CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)");
				
				//Example of performing INSERT
				stmt.execute("INSERT INTO feedback VALUES(3, 'Good product')");				
			}
			
			//Example of setting Auto commit OFF to implement transactional semantics
			conn.setAutoCommit(false);
			try (Statement stmt = conn.createStatement()) { 
				//Example of performing basic DML operations INSERT/UPDATE/DELETE
				stmt.execute("UPDATE feedback SET comment = 'Better product' WHERE rating = 3");
				stmt.execute("INSERT INTO feedback VALUES (1, 'Poor product')");
				stmt.execute("DELETE FROM feedback WHERE rating = 1");
			}
			conn.commit();
			conn.setAutoCommit(true);
			
			//Example of Prepared Statement functionality for bulk insert.			
			try(PreparedStatement pstmt = conn.prepareStatement("INSERT INTO feedback VALUES(?, ?)")) {
				pstmt.setInt(1, 4);
				pstmt.setString(2, "Excellent Product");
				pstmt.addBatch();
				
				pstmt.setInt(1, 5);
				pstmt.setString(2, "Outstanding Product");
				pstmt.addBatch();
				
				pstmt.executeBatch();			
			}
		}
		//Close SyncLite database/device cleanly.
		SyncLite.closeDevice(Path.of("t.db"));
		//You can also close all open databases in a single SQL : CLOSE ALL DATABASES
	}	
	
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		appStartup();
		TestTransactionalDevice testApp = new TestTransactionalDevice();
		testApp.myAppBusinessLogic();
	}

}

```
## Python   

```
import jaydebeapi

props = {
  "config": "synclite_logger.conf",
  "device-name" : "transactional1"
}
conn = jaydebeapi.connect("io.synclite.logger.Transactional",
                           "jdbc:synclite:c:\\synclite\\python\\data\\t.db",
                           props,
                           "synclite-logger-<version>.jar",)

curs = conn.cursor()

#Example of executing a DDL : CEATE TABLE.
#You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
curs.execute('CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)')

#Example of performing basic DML operations INSERT/UPDATE/DELETE
curs.execute("insert into feedback values (3, 'Good product')")

#Example of setting Auto commit OFF to implement transactional semantics
conn.jconn.setAutoCommit(False)
curs.execute("update feedback set comment = 'Better product' where rating = 3")
curs.execute("insert into feedback values (1, 'Poor product')")
curs.execute("delete from feedback where rating = 1")
conn.commit()
conn.jconn.setAutoCommit(True)


#Example of Prepared Statement functionality for bulk insert.
args = [[4, 'Excellent product'],[5, 'Outstanding product']]

#Close SyncLite database/device cleanly.
curs.execute("close database c:\\synclite\\python\\data\\t.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```

2. Telemetry Device : Telemetry device supports all DDL operations as supported by SQLite and Prepared Statement based INSERT operation to allow high speed batched data ingestion, performing logging of the ingested data. It empowers developers to build data-intensive streaming, IOT etc. use cases

## Java

```
package testApp;

import java.nio.file.Path;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import io.synclite.logger.*;

public class TestTelemetryDevice {
	public static Path syncLiteDBPath;

	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.Telemetry");
		Path dbPath = syncLiteDBPath.resolve("t_tel.db");
		SyncLiteTelemetry.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
	}

	public void myAppBusinessLogic() throws SQLException {
		//
		// Some application business logic
		//
		// Perform some database operations
		try (Connection conn = DriverManager
				.getConnection("jdbc:synclite_telemetry:" + syncLiteDBPath.resolve("t_tel.db"))) {
			try (Statement stmt = conn.createStatement()) {
				// Example of executing a DDL : CREATE TABLE.
				// You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
				stmt.execute("CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)");
			}

			// Example of Prepared Statement functionality for bulk insert.
			try (PreparedStatement pstmt = conn.prepareStatement("INSERT INTO feedback VALUES(?, ?)")) {
				pstmt.setInt(1, 4);
				pstmt.setString(2, "Excellent Product");
				pstmt.addBatch();

				pstmt.setInt(1, 5);
				pstmt.setString(2, "Outstanding Product");
				pstmt.addBatch();

				pstmt.executeBatch();
			}
		}
		// Close SyncLite database/device cleanly.
		SyncLiteTelemetry.closeDevice(Path.of("t_tel.db"));
		// You can also close all open databases/devices in a single SQL : CLOSE ALL
		// DATABASES
	}

	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		appStartup();
		TestTelemetryDevice testApp = new TestTelemetryDevice();
		testApp.myAppBusinessLogic();
	}
}
```
## Python

```
import jaydebeapi
props = {
  "config": "synclite_logger.conf",
  "device-name" : "telemetry1"
}
conn = jaydebeapi.connect("io.synclite.logger.Telemetry",
                           "jdbc:synclite_telemetry:c:\\synclite\\python\\data\\t_tel.db",
                           props,
                           "synclite-logger-<version>.jar",)

curs = conn.cursor()

#Example of executing a DDL : CEATE TABLE.
#You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
curs.execute('CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)')

#Example of Prepared Statement functionality for bulk insert.
args = [[4, 'Excellent product'],[5, 'Outstanding product']]
curs.executemany("insert into feedback values (?, ?)", args)

#Close SyncLite database/device cleanly.
curs.execute("close database c:\\synclite\\python\\data\\t_tel.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```

3. Appender Device : Appander device provides similar capabilities as Telemetry device with an additional capability to also mantain a copy of the ingested data on the edge device which can be leveraged for in-app edge computing/analytics.

## Java

```
package testApp;

import java.nio.file.Path;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import io.synclite.logger.*;

public class TestAppenderDevice {
	public static Path syncLiteDBPath;

	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.Telemetry");
		Path dbPath = syncLiteDBPath.resolve("t_app.db");
		SyncLiteAppender.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
	}

	public void myAppBusinessLogic() throws SQLException {
		//
		// Some application business logic
		//
		// Perform some database operations
		try (Connection conn = DriverManager
				.getConnection("jdbc:synclite_appender:" + syncLiteDBPath.resolve("t_app.db"))) {
			try (Statement stmt = conn.createStatement()) {
				// Example of executing a DDL : CREATE TABLE.
				// You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
				stmt.execute("CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)");
			}

			// Example of Prepared Statement functionality for bulk insert.
			try (PreparedStatement pstmt = conn.prepareStatement("INSERT INTO feedback VALUES(?, ?)")) {
				pstmt.setInt(1, 4);
				pstmt.setString(2, "Excellent Product");
				pstmt.addBatch();

				pstmt.setInt(1, 5);
				pstmt.setString(2, "Outstanding Product");
				pstmt.addBatch();

				pstmt.executeBatch();
			}
		}
		// Close SyncLite database/device cleanly.
		SyncLiteAppender.closeDevice(Path.of("t_app.db"));
		// You can also close all open databases/devices in a single SQL : CLOSE ALL
		// DATABASES
	}

	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		appStartup();
		TestAppenderDevice testApp = new TestAppenderDevice();
		testApp.myAppBusinessLogic();
	}

}
```

## Python

```
import jaydebeapi
props = {
  "config": "synclite_logger.conf",
  "device-name" : "appender1"
}
conn = jaydebeapi.connect("io.synclite.logger.Appender",
                           "jdbc:synclite_appender:c:\\synclite\\python\\data\\t_appender.db",
                           props,
                           "synclite-logger-<version>.jar",)

curs = conn.cursor()

#Example of executing a DDL : CEATE TABLE.
#You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
curs.execute('CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)')

#Example of Prepared Statement functionality for bulk insert.
args = [[4, 'Excellent product'],[5, 'Outstanding product']]
curs.executemany("insert into feedback values (?, ?)", args)

#Close SyncLite database/device cleanly.
curs.execute("close database c:\\synclite\\python\\data\\t_appender.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```

# Deploying SyncLite Consolidator

Refer https://hub.docker.com/r/syncliteio/synclite-consolidator for setting up SyncLite Consolidator.


# Support

Contact support@synclite.io for support and feedback.

# SyncLite - Build Anything Sync Anywhere

SyncLite is a no-code, real-time, relational data consolidation platform empowering developers to rapidly build data-intensive applications for desktops, edge devices, smartphones, with the capability to enable in-app data management, in-app analytics and perform real-time data consolidation from numerous application instances into one or more databases, data warehouses, or data lakes of your choice.

```
{Edge/Desktop/Phone Apps} + {SyncLite Logger} ------> {Staging Storage} ------> { SyncLite Consolidator} -----> {Destination DB/DW/DataLakes}
```

SyncLite is scalable, secure, extensible, fault-tolerant, enabling a wide range of use-cases, including rapidly building smart resource monitors, building native SQL (hot) data stores on top of cloud hosted databases, building data mesh architectures, database migration/replication pipelines, deploying pluggable IoT data stacks, enabling cloud databases at the edge, creating OLTP + OLAP solutions, setting up software telemetry pipelines, among others.

SyncLite excels at performing real-time data replication and consolidation from a myriad of sources including edge devices, mobile apps, desktop software, IoT devices, message brokers, and traditional databases. It empowers seamless integration into diverse databases, data warehouses, and data lakes.

More specifically, it enables following scenarios for a wide range of database, data warehouse or a data lakes.

## Edge/Mobile/Desktop Applications: 
SyncLite's pioneering CDC replication framework for embedded databases, is designed to empower data-intensive edge, desktop, and mobile applications. It seamlessly integrates with embedded databases like SQLite, DuckDB, Apache Derby, H2, HyperSQL(HSQLDB), enabling transactional, real-time data replication and consolidation from them. SyncLite EdgeDB, aka SyncLite Logger, efficiently performs Change Data Capture (CDC) from numerous application instances built on top of embedded databases. The generated log files are synchronized with SyncLite Consolidator, which replicates and consolidates them into a diverse range of industry leading databases, data warehouses, and data lakes.

## Streaming Applications: 
SyncLite facilitates development of large-scale data streaming applications through SyncLite Logger, which offers both a Kafka Producer API and SQL API. This allows for the ingestion of massive amounts of data and provides the capability to query the ingested data using the SQL API within applications. Together, SyncLite Logger and SyncLite Consolidator enable seamless last-mile data integration from thousands of streaming application instances into a diverse array of final data destinations, including all the industry leading databases, data warehouses, and data lakes.

## Smart Database Replication/ETL Pipelines: 
Set up replication, migration, or incremental ETL pipelines from a diverse range of source databases and raw data files into a diverse range of destinations.

## IoT Applications: 
Effortlessly connect hundreds of MQTT brokers (IoT gateways) to one or more final data destinations.

SyncLite continues to introduce new data integration options, expanding our range to offer hundreds of data pipeline options and countless data integration architecture possibilities. Supported systems include PostgreSQL, MySQL, SQLite, MongoDB, Apache Iceberg tables, DuckDB, Microsoft SQL Server, Oracle, IBM Db2, ClickHouse, Snowflake, Redshift, S3 Data Lake, etc.

For more details, check out 
Website : https://www.synclite.io
YouTube : https://www.youtube.com/@syncliteplatform


# SyncLite Components

SyncLite Platform comprises of two components : SyncLite Logger and SyncLite Consolidator.

```SyncLite Logger```: is a lighweight JDBC wrapper built on top of SQLite, providing a SQL interface over JDBC for user applications, enabling them for in-app data management/analytics while logging all the SQL transactional activity into log files and shipping them to one of more configured staging storages like SFTP/S3/MinIO/Kafka/GoogleDrive/MSOneDrive/NFS etc. (Note: Extended SYncLite logger extends this functionality to other embedded databases : DuckDB, Apache Derby, H2, HyperSQL. Refer : https://github.com/syncliteio/SyncLiteLoggerJavaExtended)

```SyncLite Consolidator```: is Java application deployed on an on-premise host or a cloud VM is configured to scan thousands of SyncLite devices/databases and their logs continously from the configured staging storage which are uploaded by numerous edge/desktop applications, performs real-time transactional data replication/consolidation into one or more configured databases, data warehouses or data lakes of user's choice. Refer : https://hub.docker.com/r/syncliteio/synclite-consolidator

```SyncLite DBReader```: enables data teams and data engineers effortlessly configure and orchestrate many-to-many, highly scalable database replication/migration/ETL jobs across a diverse array of databases, data warehouses and data lakes.

```SyncLite QReader```: enables IoT developers integrate vast amounts of data published to message queue brokers, into a diverse array of databases, data warehouses and data lakes, enabling real-time analytics and AI use cases at all three levels: edge, fog and cloud.

```SyncLite JobMonitor```: enables managing, scheduling and monitoring all the SyncLite jobs created on a given host.

```SyncLite Client```: is a command line tool to operate SyncLite devices, to execute SQL queries and workloads.

SyncLite Consolidator is the centralized application to all the reader/producer tools mentioned above, which receives and consolidates the incoming data streams into one or more databases, data warehouses and data lakes of user’s choice.


# Using SyncLite Logger

This repository has been created to distribute the SyncLite logger jar file as updated in src/main/resources. You can use the following maven dependency in your edge/desktop applications for creating and operating edge databases/devices.

```
<dependency>
    <groupId>io.synclite</groupId>
    <artifactId>synclite-logger</artifactId>
    <version>2024.07.31</version>
</dependency>
```

## Configuration File

Refer src/main/resources/synclite_logger.conf file for all available configuration options for SyncLite Logger. Refer "SyncLite Logger Configuration" section in the documentation at https://www.synclite.io/resources/documentation for more details about all configuration options. 

## Application Code Samples (SQL API)

SyncLite Platform allows applications to create three types of devices:

### 1. SQLite Transactional Device : 

SQLite transactional device supports all database operations as supported by SQLite and performs transactional logging of all the DDL and DML operations performed by the application. It empowers developers to build use cases such as building data-intensive sync-ready applications, Edge + Cloud GenAI search and RAG applications, native SQL (hot) hot data stores, SQL application caches, edge enablement of cloud databases etc.

#### Java
```
package testApp;

import java.nio.file.Path;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import io.synclite.logger.*;


public class TestSQLiteDevice {
	
	public static Path syncLiteDBPath;
	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.SQLite");
		Path dbPath = syncLiteDBPath.resolve("t.db");
		SQLite.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
	}	
	
	public void myAppBusinessLogic() throws SQLException {
		//
		//Some application business logic
		//
		//Perform some database operations		
		try (Connection conn = DriverManager.getConnection("jdbc:synclite_sqlite:" + syncLiteDBPath.resolve("t.db"))) {
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
		SQLite.closeDevice(Path.of("t.db"));
		//You can also close all open databases in a single SQL : CLOSE ALL DATABASES
	}	
	
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		appStartup();
		TestSQLiteDevice testApp = new TestTransactionalDevice();
		testApp.myAppBusinessLogic();
	}

}

```
#### Python   

```
import jaydebeapi

props = {
  "config": "synclite_logger.conf",
  "device-name" : "sqlite1"
}
conn = jaydebeapi.connect("io.synclite.logger.SQLite",
                           "jdbc:synclite_sqlite:c:\\synclite\\python\\data\\t.db",
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
curs.executemany("insert into feedback values (?, ?)", args)

#Close SyncLite database/device cleanly.
curs.execute("close database c:\\synclite\\python\\data\\t.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```

### 2. Streaming Device : 
Streaming device supports all DDL operations as supported by SQLite and Prepared Statement based INSERT operation to allow high speed concurrent batched data ingestion, performing logging of the ingested data. It empowers developers to build high volumne streaming applications enabled with last mile data integration from thousands of edge points into centralized database destinations.

#### Java

```
package testApp;

import java.nio.file.Path;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import io.synclite.logger.*;

public class TestStreamingDevice {
	public static Path syncLiteDBPath;

	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.Streaming");
		Path dbPath = syncLiteDBPath.resolve("t_str.db");
		Streaming.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
	}

	public void myAppBusinessLogic() throws SQLException {
		//
		// Some application business logic
		//
		// Perform some database operations
		try (Connection conn = DriverManager
				.getConnection("jdbc:synclite_streaming:" + syncLiteDBPath.resolve("t_str.db"))) {
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
		Streaming.closeDevice(Path.of("t_str.db"));
		// You can also close all open databases/devices in a single SQL : CLOSE ALL
		// DATABASES
	}

	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		appStartup();
		TestStreamingDevice testApp = new TestStreamingDevice();
		testApp.myAppBusinessLogic();
	}
}
```
#### Python

```
import jaydebeapi
props = {
  "config": "synclite_logger.conf",
  "device-name" : "streaming1"
}
conn = jaydebeapi.connect("io.synclite.logger.Streaming",
                           "jdbc:synclite_streaming:c:\\synclite\\python\\data\\t_str.db",
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
curs.execute("close database c:\\synclite\\python\\data\\t_str.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```

### 3. SQLite Appender Device : 
SQLite appender device provides similar capabilities as Streaming device but it allows a single writer at any point (unlike a Streaming device which supports concurrent data ingestion). It has an additional capability to also mantain a copy of the ingested data (in a local SQLite database file) on the edge/mobile/desktop device which can be leveraged for in-app edge computing/analytics.

#### Java

```
package testApp;

import java.nio.file.Path;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import io.synclite.logger.*;

public class TestSQLiteAppenderDevice {
	public static Path syncLiteDBPath;

	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.SQLiteAppender");
		Path dbPath = syncLiteDBPath.resolve("test_sqlite_appender.db");
		SQLiteAppender.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
	}

	public void myAppBusinessLogic() throws SQLException {
		//
		// Some application business logic
		//
		// Perform some database operations
		try (Connection conn = DriverManager
				.getConnection("jdbc:synclite_sqlite_appender:" + syncLiteDBPath.resolve("test_sqlite_appender.db"))) {
			try (Statement stmt = conn.createStatement()) {
				// Example of executing a DDL : CREATE TABLE.
				// You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
				stmt.execute("CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)");
			}

			// Example of Prepared Statement functionality for bulk insert.
			// Note that Appender Devices only support DDLs, INSERT INTO DML operations and SELECT queries.
			//
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
		SQLiteAppender.closeDevice(Path.of("test_sqlite_appender.db"));
		// You can also close all open databases/devices in a single SQL : CLOSE ALL
		// DATABASES
	}

	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		appStartup();
		TestSQLiteAppenderDevice testApp = new TestSQLiteAppenderDevice();
		testApp.myAppBusinessLogic();
	}

}
```

#### Python

```
import jaydebeapi
props = {
  "config": "synclite_logger.conf",
  "device-name" : "sqliteapp1"
}
conn = jaydebeapi.connect("io.synclite.logger.Appender",
                           "jdbc:synclite_sqlite_appender:c:\\synclite\\python\\data\\test_sqlite_appender.db",
                           props,
                           "synclite-logger-<version>.jar",)

curs = conn.cursor()

#Example of executing a DDL : CREATE TABLE.
#You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
curs.execute('CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)')

#Example of Prepared Statement functionality for bulk insert.
args = [[4, 'Excellent product'],[5, 'Outstanding product']]
curs.executemany("insert into feedback values (?, ?)", args)

#Close SyncLite database/device cleanly.
curs.execute("close database c:\\synclite\\python\\data\\test_sqlite_appender.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```


## Application Code Samples (Kafka API)

```
package testApp;

import io.synclite.logger.*;

public class TestKafkaProducer {

	public static void main(String[] args) throws Exception {

		Properties props = new Properties();
	    
		//
		//Set properties to use a staging storage of your choice e.g. S3, MinIO, SFTP etc. 
		//where SyncLite logger will ship log files continuously for consumption by SyncLite consolidator
		//
		
        	Producer<String, String> producer = new io.synclite.logger.KafkaProducer(props);

		ProducerRecord<String, String> record = new ProducerRecord<>("test", "key", "value");
        
		//
		//You can use same or different KafkaProducer objects to ingest data concurrently over multiple theads.
		//
        	producer.send(record);
		
		produer.close();

	}
```


# Deploying SyncLite Consolidator

Refer https://hub.docker.com/r/syncliteio/synclite-consolidator for setting up SyncLite Consolidator.


# Support

Contact support@synclite.io for support and feedback.

# SyncLite - Build Anything Sync Anywhere

SyncLite is open-source no-code, real-time, relational data consolidation platform empowering developers to rapidly build data-intensive applications for desktops, edge devices, smartphones, with the capability to enable in-app data management, in-app analytics and perform real-time data consolidation from numerous application instances into one or more databases, data warehouses, or data lakes of your choice.

```
{Edge/Desktop Apps} + {SyncLite Logger} ---> {Staging Storage} ---> { SyncLite Consolidator} ---> {Destination DB/DW/DataLakes}
```

SyncLite is scalable, secure, extensible, fault-tolerant, enabling a wide range of use-cases, including rapidly building smart resource monitors, building native SQL (hot) data stores on top of cloud hosted databases, building data mesh architectures, database migration/replication pipelines, deploying pluggable IoT data stacks, enabling cloud databases at the edge, creating OLTP + OLAP solutions, setting up software telemetry pipelines, among others.

SyncLite excels at performing real-time data replication and consolidation from a myriad of sources including edge devices, mobile apps, desktop software, IoT devices, message brokers, and traditional databases. It empowers seamless integration into diverse databases, data warehouses, and data lakes.

More specifically, it enables following scenarios for a wide range of database, data warehouse or a data lakes.

## Build Sync-Ready Applications with Zero-Coding: 
SyncLite's novel CDC replication framework for embedded databases, is designed to empower general purpose data-intensive applications, Gen AI Search/RAG applications for edge, desktop, and mobile environments. It seamlessly integrates with embedded databases like SQLite, DuckDB, Apache Derby, H2, HyperSQL(HSQLDB), enabling Change Data Capture + transactional, real-time data replication and consolidation from them into a diverse range of industry leading databases, data warehouses, and data lakes, enabling global analytics, AI Search and RAG applications.

```
{Edge/Desktop Apps} + {SyncLite Logger + Embedded Databases} ---> {Staging Storage} ---> {SyncLite Consolidator} ---> {Destination DB/DW/DataLakes}
```
Learn more: 

https://www.synclite.io/synclite/sync-ready-apps

https://www.synclite.io/solutions/gen-ai-search-rag


## Build Streaming Applications For Last Mile Data Integration: 
SyncLite facilitates development of large-scale data streaming applications through SyncLite Logger, which offers both a Kafka Producer API and SQL API. This allows for the ingestion of massive amounts of data and provides the capability to query the ingested data using the SQL API within applications. Together, SyncLite Logger and SyncLite Consolidator enable seamless last-mile data integration from thousands of streaming application instances into a diverse array of final data destinations.

```
{Data Streaming Apps} + {SyncLite Logger} ---> {Staging Storage} ---> {SyncLite Consolidator} ---> {Destination DB/DW/DataLakes}
```

Learn more: https://www.synclite.io/synclite/last-mile-streaming


## Deploy Smart Database ETL/Replication/Migration Pipelines:
Set up replication, migration, or incremental ETL pipelines from a diverse range of source databases and raw data files into a diverse range of destinations.

```
{Source Databases} ---> {SyncLite DBReader} ---> {Staging Storage} ---> {SyncLite Consolidator} ---> {Destination DB/DW/DataLakes}
```

Learn More: https://www.synclite.io/solutions/smart-database-etl

## Setup Rapid IoT Data Connectors:
Effortlessly connect numerous MQTT brokers (IoT gateways) to one or more final data destinations.

```
{IoT Message Brokers} ---> {SyncLite QReader} ---> {Staging Storage} ---> {SyncLite Consolidator} ---> {Destination DB/DW/DataLakes}
```
Learn More: https://www.synclite.io/solutions/iot-data-connector

SyncLite continues to introduce new data integration options, expanding our range to offer hundreds of data pipeline options and countless data integration architecture possibilities. Supported systems include PostgreSQL, MySQL, SQLite, MongoDB, Apache Iceberg tables, DuckDB, Microsoft SQL Server, Oracle, IBM Db2, ClickHouse, Snowflake, Redshift, S3 Data Lake, etc.

For more details, check out 
Website : https://www.synclite.io
YouTube : https://www.youtube.com/@syncliteplatform


# SyncLite Components

```SyncLite Logger``` is a JDBC driver, enables developers to rapidly build 
	
-sync-enabled, robust, responsive, high-performance, low-latency, transactional, data intensive applications for edge/mobile/desktop platforms using their favorite embedded databases (SQLite, DuckDB, Apache Derby, H2, HyperSQL)
  	
-large scale data streaming solutions for last mile data integrations into a wide range of industry leading databases, while offering ability to perform real-time analytics using the native embedded databases over streaming data, at the producer end of the pipelines.

```SyncLite DB``` is a sync-enabled, single-node database server that wraps popular embedded databases like SQLite, DuckDB, Apache Derby, H2, and HyperSQL. Unlike the embeddable SyncLite Logger library for Java and Python applications, SyncLite DB acts as a standalone server, allowing your edge or desktop applications—regardless of the programming language—to connect and send SQL requests (wrapped in JSON format) over HTTP. 

```SyncLite Client``` is a command line tool to operate SyncLite devices, to execute SQL queries and workloads.

```SyncLite DBReader``` enables data teams and data engineers to configure and orchestrate many-to-many, highly scalable, incremental/log-based database ETL/replication/migration jobs across a diverse array of databases, data warehouses and data lakes.

```SyncLite QReader``` enables developers to integrate IoT data published to message queue brokers, into a diverse array of databases, data warehouses and data lakes, enabling real-time analytics and AI use cases at all three levels: edge, fog and cloud.

```SyncLite Consolidator``` is the centralized application to all the reader/producer tools mentioned above, which receives and consolidates the incoming data and log files in real-time into one or more databases, data warehouses and data lakes of user’s choice. SyncLite Consolidator also offers additional features: table/column/value filtering and mapping, data type mapping, database trigger installation, fine-tunable writes, support for multiple destinations and more.

```SyncLite JobMonitor``` enables managing, scheduling and monitoring all SyncLite jobs created on a given host.

```SyncLite Validator``` is an E2E integration testing tool for SyncLite.


# Using SyncLite Logger

- This repository has been created to distribute the SyncLite logger jar file as updated in src/main/resources. 

- OR You can use the following maven dependency in your edge/desktop applications for creating and operating edge databases/devices.

```
<dependency>
    <groupId>io.synclite</groupId>
    <artifactId>synclite-logger</artifactId>
    <version>2024.10.02</version>
</dependency>
```

## Configuration File

Refer src/main/resources/synclite_logger.conf file for all available configuration options for SyncLite Logger. Refer "SyncLite Logger Configuration" section in the documentation at https://www.synclite.io/resources/documentation for more details about all configuration options. 

## Application Code Samples (SQL API)

SyncLite Logger facilitates applications to create various kinds of embedded devices/databases :

### Transactional Devices : 

Transactional devices (SQLite, DuckDB, Apache Derby, H2, HyperSQL) support all database operations and performs transactional logging of all the DDL and DML operations performed by the application. These empower developers to build use cases such as building data-intensive sync-ready applications, Edge + Cloud GenAI search and RAG applications, native SQL (hot) hot data stores, SQL application caches, edge enablement of cloud databases etc.

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


public class TestTransactionalDevice {
	
	public static Path syncLiteDBPath;
	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.SQLite");
		//
		//////////////////////////////////////////////////////
		//For other types of transactional devices : 
		//DuckDB : Class.forName("io.synclite.logger.DuckDB");
		//Apache Derby : Class.forName("io.synclite.logger.Derby");
		//H2 : Class.forName("io.synclite.logger.H2");
		//HyperSQL : Class.forName("io.synclite.logger.HyperSQL");
		//////////////////////////////////////////////////////
		//

		Path dbPath = syncLiteDBPath.resolve("test_tran.db");
		SQLite.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
		//
		//////////////////////////////////////////////////////
		//For other types of transactional devices : 
		//DuckDB : DuckDB.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
		//Apache Derby : Derby.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
		//H2 : H2.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
		//HyperSQL : HyperSQL.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
		//////////////////////////////////////////////////////
		//
	}	
	
	public void myAppBusinessLogic() throws SQLException {
		//
		//Some application business logic
		//
		//Perform some database operations		
		try (Connection conn = DriverManager.getConnection("jdbc:synclite_sqlite:" + syncLiteDBPath.resolve("test_sqlite.db"))) {
			//
		        //////////////////////////////////////////////////////////////////
			//For other types of transactional devices use following connection strings :
			//For DuckDB : jdbc:synclite_duckdb:<db_path>
			//For Apache Derby : jdbc:synclite_derby:<db_path>
			//For H2 : jdbc:synclite_h2:<db_path>
			//For HyperSQL : jdbc:synclite_hsqldb:<db_path>
			///////////////////////////////////////////////////////////////////
			//
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
		SQLite.closeDevice(Path.of("test_sqlite.db"));
		//
		///////////////////////////////////////////////////////
		//For other types of transactional devices :
		//DuckDB : DuckDB.closeDevice
		//Apache Derby : Derby.closeDevice
		//H2 : H2.closeDevice
		//HyperSQL : HyperSQL.closeDevice
		//////////////////////////////////////////////////////
		//
		//You can also close all open databases in a single SQL : CLOSE ALL DATABASES
	}	
	
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		appStartup();
		TestTransactionalDevice testApp = new TestTransactionalDevice();
		testApp.myAppBusinessLogic();
	}
}

```
#### Python   

```
import jaydebeapi

props = {
  "config": "synclite_logger.conf",
  "device-name" : "tran1"
}
conn = jaydebeapi.connect("io.synclite.logger.SQLite",
                           "jdbc:synclite_duckdb:c:\\synclite\\python\\data\\test_sqlite.db",
                           props,
                           "synclite-logger-<version>.jar",)
#//
#////////////////////////////////////////////////////////////////
#For other types of transactional devices use following are the class names and connection strings :
#For DuckDB - Class : io.synclite.logger.DuckDB, Connection String : jdbc:synclite_duckdb:<db_path>
#For Apache Derby - Class : io.synclite.logger.Derby, Connection String : jdbc:synclite_derby:<db_path>
#For H2 - Class : io.synclite.logger.H2, Connection String : jdbc:synclite_h2:<db_path>
#For HyperSQL - Class : io.synclite.logger.HyperSQL, Connection String : jdbc:synclite_hsqldb:<db_path>
#/////////////////////////////////////////////////////////////////
#//

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
curs.execute("close database c:\\synclite\\python\\data\\test_sqlite.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```

### Appender Devices :

Appender devices (SQLiteAppender, DuckDBAppender, DerbyAppender, H2Appender, HyperSQLAppender) allow all DDL operations and Prepared Statement based INSERT operations and are highly optimized for high speed concurrent batched data ingestion, performing logging of the ingested data. Unlike transactional devices, appender devices only allow INSERT DML operations (UPDATE and DELETE are not allowed). Appender devices empower developers to build high volume streaming applications enabled with last mile data integration from thousands of edge points into centralized database destinations as well as in-app analytics by enabling fast read access to ingested data from the underlying local embedded databases storing the ingested/streamed data.

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

public class TestAppenderDevice {
	public static Path syncLiteDBPath;

	public static void appStartup() throws SQLException, ClassNotFoundException {
		syncLiteDBPath = Path.of(System.getProperty("user.home"), "synclite", "db");
		Class.forName("io.synclite.logger.SQLiteAppender");
		//
		//////////////////////////////////////////////////////
		//For other types of appender devices : 
		//DuckDB : Class.forName("io.synclite.logger.DuckDBAppender");
		//Apache Derby : Class.forName("io.synclite.logger.DerbyAppender");
		//H2 : Class.forName("io.synclite.logger.H2Appender");
		//HyperSQL : Class.forName("io.synclite.logger.HyperSQLAppender");
		//////////////////////////////////////////////////////
		//
		Path dbPath = syncLiteDBPath.resolve("test_appender.db");
		SQLiteAppender.initialize(dbPath, syncLiteDBPath.resolve("synclite_logger.conf"));
	}

	public void myAppBusinessLogic() throws SQLException {
		//
		// Some application business logic
		//
		// Perform some database operations
		try (Connection conn = DriverManager.getConnection("jdbc:synclite_sqlite_appender:" + syncLiteDBPath.resolve("test_appender.db"))) {
			//
		        //////////////////////////////////////////////////////////////////
			//For other types of appender devices use following connection strings :
			//For DuckDBAppender : jdbc:synclite_duckdb_appender:<db_path>
			//For DerbyAppender : jdbc:synclite_derby_appender:<db_path>
			//For H2Appender : jdbc:synclite_h2_appender:<db_path>
			//For HyperSQLAppender : jdbc:synclite_hsqldb_appender:<db_path>
			///////////////////////////////////////////////////////////////////
			//
			try (Statement stmt = conn.createStatement()) {
				// Example of executing a DDL : CREATE TABLE.
				// You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
				stmt.execute("CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)");
			}

			//
			// Example of Prepared Statement functionality for bulk insert.
			// Note that Appender Devices allows all DDL operations, INSERT INTO DML operations (UPDATES and DELETES are not allowed) and SELECT queries.
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
		SQLiteAppender.closeDevice(Path.of("test_appender.db"));
		//
		///////////////////////////////////////////////////////
		//For other types of appender devices :
		//DuckDBAppender : DuckDBAppender.closeDevice
		//DerbyAppender : DerbyAppender.closeDevice
		//H2Appender : H2Appender.closeDevice
		//HyperSQLAppender : HyperSQLAppender.closeDevice
		//////////////////////////////////////////////////////
		//
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

#### Python

```
import jaydebeapi
props = {
  "config": "synclite_logger.conf",
  "device-name" : "appender1"
}
conn = jaydebeapi.connect("io.synclite.logger.SQLiteAppender",
                           "jdbc:synclite_sqlite_appender:c:\\synclite\\python\\data\\test_appender.db",
                           props,
                           "synclite-logger-<version>.jar",)
#//
#////////////////////////////////////////////////////////////////
#For other types of appender devices use following are the class names and connection strings :
#For DuckDBAppender - Class : io.synclite.logger.DuckDBAppender, Connection String : jdbc:synclite_duckdb_appender:<db_path>
#For DerbyAppender - Class : io.synclite.logger.DerbyAppender, Connection String : jdbc:synclite_derby_appender:<db_path>
#For H2Appender - Class : io.synclite.logger.H2Appender, Connection String : jdbc:synclite_h2_appender:<db_path>
#For HyperSQLAppender - Class : io.synclite.logger.HyperSQLAppender, Connection String : jdbc:synclite_hsqldb_appender:<db_path>
#/////////////////////////////////////////////////////////////////
#//

curs = conn.cursor()

#Example of executing a DDL : CREATE TABLE.
#You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
curs.execute('CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)')

#Example of Prepared Statement functionality for bulk insert.
args = [[4, 'Excellent product'],[5, 'Outstanding product']]
curs.executemany("insert into feedback values (?, ?)", args)

#Close SyncLite database/device cleanly.
curs.execute("close database c:\\synclite\\python\\data\\test_appender.db");

#You can also close all open databases in a single SQL : CLOSE ALL DATABASES
```

### 2. Streaming Device : 
Streaming device allows all DDL operations as supported by SQLite and Prepared Statement based INSERT operations (UPDATE and DELETE not allowed) to allow high speed concurrent batched data ingestion, performing logging and streaming of the ingested data. Streaming device empowers developers to build high volume data streaming applications enabled with last mile data integration from thousands of edge applications into centralized database destinations.

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


# Open Source Software

SyncLite is available as open source software under Apache 2.0 license. You can clone the repository and build SyncLite Logger jar yourself along with other SyncLite tools.
Refer: https://github.com/syncliteio/SyncLite


# Support

Contact support@synclite.io for support and feedback.

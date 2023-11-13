# SyncLite Platform

SyncLite is a no-code, real-time, relational data consolidation platform empowering developers to rapidly build data-intensive applications for desktops, edge devices, smartphones, with the capability to enable in-app data management, in-app analytics and perform real-time data consolidation from numerous application instances into one or more databases, data warehouses, or data lakes of your choice.

{Edge/Desktop/Phone Apps} + {SyncLite Logger} ------> {Staging Storage} ------> { SyncLite Consolidator} -----> {Destination DB/DW/DataLakes}

SyncLite is scalable, secure, extensible, fault-tolerant, enabling a wide range of use-cases, including rapidly building smart resource monitors, native SQL data stores, building data mesh architectures, database migration/replication pipelines, deploying pluggable IoT data stacks, enabling cloud databases at the edge, creating OLTP + OLAP solutions, setting up software telemetry pipelines, among others.

For more details, check out https://www.synclite.io

# SyncLite Components

SyncLite Platform comprises of two components : SyncLite Logger and SyncLite Consolidator.

1. SyncLite Logger : SyncLite Logger is a jar file built on top of SQLite, providing a JDBC SQL interface for user applications, enabling them for in-app data management and in-app edge analytics while logging all the SQL transactional activity into log files and shipping them to one of more configured staging storages like SFTP/S3/MinIO/Kafka/GoogleDrive/MSOneDrive/NFS etc. 

2. SyncLite Consolidator : SyncLite Consolidator is a Java application deployed on an on-premise host or a cloud VM is configured to scan thousands of  SyncLite devices/databases and their logs continously from the configured stating storage which are uploaded by numerous edge/desktop applications, performs real-time data replication/consolidation into one or more configured databases/data warehouses/data lakes.

   
# Using SyncLite Logger

This repository has been created to distribute the SyncLite Logger jar file as updated in src/main/resources. You can use the following maven dependency in your edge/desktop applications for creating and operating edge databases/devices.
```
<dependency>
    <groupId>io.synclite</groupId>
    <artifactId>synclite-logger</artifactId>
    <version>2023.11.11</version>
</dependency>
```
# Configuration File

Refer src/main/resources/synclite_logger.conf file for all available configuration options for SyncLite Logger. Refer "SyncLite Logger Configuration" section in the documentation at https://www.synclite.io/resources/documentation for more details about all configuration options. 

# Application Code Samples

SyncLite Platform allows applications to create three types of devices:

1. Transactional Device : Transcational device supports all database operations as supported by SQLite and performs transactional logging of all the DDL and DML operations performed by the application. It empowers developers to build use cases such as hot data stores, SQL application caches, edge enablement of cloud databases, OLTP + OLAP data stores etc.
```
	public static void appStartup() throws SQLException, ClassNotFoundException {
		Class.forName("io.synclite.logger.Transactional");
		Path dbPath = Path.of("t.db");
		SyncLite.initialize(dbPath, Path.of("synclite_logger.conf"));
	}	
	
	public void myAppBusinessLogic() throws SQLException {
		//
		//Some application business logic
		//
		//Perform some database operations		
		try (Connection conn = DriverManager.getConnection("jdbc:synclite:t.db")) {
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
```
   
2. Telemetry Device : Telemetry device supports all DDL operations as supported by SQLite and Prepared Statement based INSERT operation to allow high speed batched data ingestion, performing logging of the ingested data. It empowers developers to build data-intensive streaming, IOT etc. use cases

```
	public static void appStartup() throws SQLException, ClassNotFoundException {
		Class.forName("io.synclite.logger.Telemetry");
		Path dbPath = Path.of("t_tel.db");
		SyncLiteTelemetry.initialize(dbPath, Path.of("synclite_logger.conf"));
	}	
	
	public void myAppBusinessLogic() throws SQLException {
		//
		//Some application business logic
		//
		//Perform some database operations		
		try (Connection conn = DriverManager.getConnection("jdbc:synclite_telemetry:t_tel.db")) {
			try (Statement stmt = conn.createStatement()) { 
				//Example of executing a DDL : CREATE TABLE. 
				//You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
				stmt.execute("CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)");				
			}			
		
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
		SyncLiteTelemetry.closeDevice(Path.of("t_tel.db"));
		//You can also close all open databases/devices in a single SQL : CLOSE ALL DATABASES
	}	
```
3. Appender Device : Appander device provides similar capabilities as Telemetry device with an additional capability to also mantain a copy of the ingested data on the edge device which can be leveraged for in-app edge computing/analytics.
```
	public static void appStartup() throws SQLException, ClassNotFoundException {
		Class.forName("io.synclite.logger.Appender");
		Path dbPath = Path.of("t_appender.db");
		SyncLiteAppender.initialize(dbPath, Path.of("synclite_logger.conf"));
	}	
	
	public void myAppBusinessLogic() throws SQLException {
		//
		//Some application business logic
		//
		//Perform some database operations		
		try (Connection conn = DriverManager.getConnection("jdbc:synclite_appender:t_appender.db")) {
			try (Statement stmt = conn.createStatement()) { 
				//Example of executing a DDL : CREATE TABLE. 
				//You can execute other DDL operations : DROP TABLE, ALTER TABLE, RENAME TABLE.
				stmt.execute("CREATE TABLE IF NOT EXISTS feedback(rating INT, comment TEXT)");				
			}			
		
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
		SyncLiteAppender.closeDevice(Path.of("t_appender.db"));
		//You can also close all open databases/devices in a single SQL : CLOSE ALL DATABASES
	}	

```

# Deploying SyncLite Consolidator

Refer https://hub.docker.com/repository/docker/syncliteio/synclite-consolidator/general for setting up SyncLite Consolidator.


# Support

Contact support@synclite.io for support and feedback.

Yes. Based on our db-summarized.txt lab setup, here is the clean sequence to reproduce the SQL Server container, database, Users table, 2M-row data, and indexes.

I couldn't locate the exact db-summarized.txt file in your saved files, so I'm using the commands we established in this lab.

1. Pull SQL Server image
docker pull mcr.microsoft.com/mssql/server:2022-latest
2. Create Docker network
docker network create app-network

If it already exists, Docker will tell you; that's okay.

3. Create persistent SQL Server volume
docker volume create sqlserver-data
4. Start SQL Server container
docker run -d --name db --network app-network -p 1433:1433 -e ACCEPT_EULA=Y -e MSSQL_PID=Developer -e MSSQL_SA_PASSWORD='' -v sqlserver-data:/var/opt/mssql mcr.microsoft.com/mssql/server:2022-latest
5. Check SQL Server
docker ps

Then:

docker logs db

Look for SQL Server being ready for connections.

6. Connect to SQL Server
docker exec -it db /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'SqlLab@2026Strong!' -C

Now you're inside SQL Server.

7. Create database

SQL commands are kept on one line as you prefer:

CREATE DATABASE ExperimentDB;

Then:

USE ExperimentDB;
8. Create Users table
CREATE TABLE Users (UserId BIGINT NOT NULL, Name VARCHAR(100), Address VARCHAR(200));
9. Create primary key
ALTER TABLE Users ADD CONSTRAINT PK_Users PRIMARY KEY (UserId);

This gives us the clustered primary-key structure on UserId.

10. Create Name index
CREATE INDEX IX_Users_Name ON Users(Name);
11. Create Address index
CREATE INDEX IX_Users_Address ON Users(Address);
12. Verify indexes
SELECT name, type_desc FROM sys.indexes WHERE object_id = OBJECT_ID('Users');

You should see approximately:

PK_Users
IX_Users_Name
IX_Users_Address
13. Verify table
SELECT COUNT(*) FROM Users;

For the completed lab, we had approximately 2 million rows (2,000,001 after the final test insert).

14. Test the indexes

Name lookup:

SET STATISTICS IO ON; SELECT * FROM Users WHERE Name = 'User1900000';

Address lookup:

SET STATISTICS IO ON; SELECT * FROM Users WHERE Address = 'Address-1900000';

Primary-key range:

SET STATISTICS IO ON; SELECT * FROM Users WHERE UserId BETWEEN 1900000 AND 1900100;
Important: your 2M records

The commands above create the database/table/index structure, but they don't recreate the 2M rows.

Your Git repo should contain the scripts needed to generate/load those 2M records. If you want to reproduce the entire lab from Git on another machine, the next architectural step should be:

Git clone
   ↓
docker pull SQL Server
   ↓
docker run SQL Server
   ↓
create DB
   ↓
create table
   ↓
create indexes
   ↓
load 2M rows
   ↓
run performance experiments

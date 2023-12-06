# Customer Support Ticketing System

## Part One: Creating Tables and Tablespaces

### Tablespace

```sql
CREATE TABLESPACE Ticketing_System
DATAFILE 'Ticketing_System.dbf'
SIZE 100M
AUTOEXTEND ON
NEXT 50M;
```

### Customer Table
```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY, 
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    Phone VARCHAR(20)
) TABLESPACE Ticketing_System;
```

### SupportTickets Table
```sql
CREATE TABLE SupportTickets (
    TicketID INT PRIMARY KEY,
    CustomerID INT, 
    Description VARCHAR(500),
    Priority VARCHAR(20),
    Status VARCHAR(20),
    CreatedAt TIMESTAMP,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
) TABLESPACE Ticketing_System;
```

### SupportAgents Table
```sql
CREATE TABLE SupportAgents (
    AgentID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    Skills VARCHAR(255),
    Availability CHAR(1) 
) TABLESPACE Ticketing_System;
```

### TicketAssignments Table
```sql
CREATE TABLE TicketAssignments (
    AssignmentID INT PRIMARY KEY,
    TicketID INT, 
    AgentID INT,
    AssignedAt TIMESTAMP,
    FOREIGN KEY (TicketID) REFERENCES SupportTickets(TicketID),
    FOREIGN KEY (AgentID) REFERENCES SupportAgents(AgentID)
) TABLESPACE Ticketing_System;
```

### Interactions Table
```sql
CREATE TABLE Interactions (
    InteractionID INT PRIMARY KEY,
    TicketID INT,
    InteractionText VARCHAR(500),
    InteractionTime TIMESTAMP,
    FOREIGN KEY (TicketID) REFERENCES SupportTickets(TicketID)
) TABLESPACE Ticketing_System;
```

## Part Two: Creating Users and Inserting Data

### Creating Users
```sql
conn / as sysdba
CREATE USER CustomersUser IDENTIFIED BY cust;
CREATE USER SupportTicketsUser IDENTIFIED BY supp;
CREATE USER SupportAgentsUser IDENTIFIED BY suppag;
CREATE USER TicketAssignmentsUser IDENTIFIED BY tick;
CREATE USER InteractionsUser IDENTIFIED BY inter;
```
### Inserting Data

```sql
-- Insert statements for Customers Table
INSERT INTO Customers (CustomerID, FirstName, LastName, Email, Phone) VALUES (3, 'Bader', 'Shamlan', 'bader.shamlan@example.com', '111-222-3333');
INSERT INTO Customers (CustomerID, FirstName, LastName, Email, Phone) VALUES (4, 'Sultan', 'Alkharmani', 'sultan.alkharmani@example.com', '444-555-6666');
INSERT INTO Customers (CustomerID, FirstName, LastName, Email, Phone) VALUES (5, 'Mohammed', 'Alnomees', 'mohammed.alnomees@example.com', '777-888-9999');

-- Insert statements for SupportTickets Table
INSERT INTO SupportTickets (TicketID, CustomerID, Description, Priority, Status, CreatedAt) VALUES (3, 3, 'Issue reported by Bader Shamlan', 'Medium', 'Open', SYSTIMESTAMP);
INSERT INTO SupportTickets (TicketID, CustomerID, Description, Priority, Status, CreatedAt) VALUES (4, 4, 'Support request from Sultan Alkharmani', 'High', 'In Progress', SYSTIMESTAMP);
INSERT INTO SupportTickets (TicketID, CustomerID, Description, Priority, Status, CreatedAt) VALUES (5, 5, 'Problem reported by Mohammed Alnomees', 'Low', 'Closed', SYSTIMESTAMP);

-- Insert statements for SupportAgents Table
INSERT INTO SupportAgents (AgentID, FirstName, LastName, Email, Skills, Availability) VALUES (101, 'Bader', 'Shamlan', 'bader@example.com', 'Technical Support', 'Y');
INSERT INTO SupportAgents (AgentID, FirstName, LastName, Email, Skills, Availability) VALUES (102, 'Sultan', 'Alkharmani', 'sultan@example.com', 'Billing Support', 'Y');
INSERT INTO SupportAgents (AgentID, FirstName, LastName, Email, Skills, Availability) VALUES (103, 'Mohammed', 'Alnomees', 'mohammed@example.com', 'Technical Support', 'Y');

-- Insert statements for TicketAssignments Table
INSERT INTO TicketAssignments (AssignmentID, TicketID, AgentID, AssignedAt) VALUES (201, 3, 101, SYSTIMESTAMP);
INSERT INTO TicketAssignments (AssignmentID, TicketID, AgentID, AssignedAt) VALUES (202, 4, 102, SYSTIMESTAMP);
INSERT INTO TicketAssignments (AssignmentID, TicketID, AgentID, AssignedAt) VALUES (203, 5, 103, SYSTIMESTAMP);

-- Insert statements for Interactions Table
INSERT INTO Interactions (InteractionID, TicketID, InteractionText, InteractionTime) VALUES (301, 3, 'Resolved issue for Bader Shamlan', SYSTIMESTAMP);
INSERT INTO Interactions (InteractionID, TicketID, InteractionText, InteractionTime) VALUES (302, 4, 'Provided support for Sultan Alkharmani', SYSTIMESTAMP);
INSERT INTO Interactions (InteractionID, TicketID, InteractionText, InteractionTime) VALUES (303, 5, 'Closed problem for Mohammed Alnomees', SYSTIMESTAMP);
```

### Creating another user and grant Select and Insert privileges
```sql
CREATE USER anotherUser IDENTIFIED BY user;
-- Granting SELECT and INSERT permissions to anotherUser
GRANT SELECT, INSERT ON Customers TO anotherUser WITH GRANT OPTION;
GRANT SELECT, INSERT ON SupportTickets TO anotherUser WITH GRANT OPTION;
GRANT SELECT, INSERT ON Interactions TO anotherUser WITH GRANT OPTION;
```

### Fine-Grained Auditing (FGA)
```sql
-- Adding FGA policy for Customers table
BEGIN
  DBMS_FGA.ADD_POLICY (
    object_schema   => 'CustomersUser',
    object_name     => 'Customers',
    policy_name     => 'Customers_FGA',
    audit_condition => 'SELECT, DELETE',
    statement_types => 'SELECT, DELETE',
    audit_column    => NULL,
    handler_schema  => 'CustomersUser',
    handler_module  => 'AuditHandlerPackage.HandleAuditRecord',
    enable          => TRUE
  );
END;
/
```
#### Perform SELECT and INSERT operations on the "Customers" table:
```sql
-- SELECT operation
SELECT * FROM Customers;

-- INSERT operation
INSERT INTO Customers (CustomerID, FirstName, LastName, Email, Phone)
VALUES (7, 'Ahmad', 'Alzahrani', 'Ahmad.Alzahrani@example.com', '123-456-7890');
```
#### Check the Audit Trail
```sql
SELECT * FROM DBA_AUDIT_TRAIL WHERE TABLE_NAME = 'CUSTOMERS';
```

## Part Three: Archive log and backup job

### Assessing Available Binary Logs
```mysql
SHOW BINARY LOGS;
```
![image](https://github.com/BwShamlan/test/assets/98660242/71adc5b6-74a6-4413-8ddf-5781978b9726)
![image](https://github.com/BwShamlan/test/assets/98660242/ea113ae0-2a8a-4459-b167-0a60f141895b)


### Backup job using sqlbak
#### Connection steps:
![Screenshot 2023-12-02 223902](https://github.com/BwShamlan/test/assets/98660242/468bb2f9-8b86-443f-acf0-58404766b697)
![Screenshot 2023-12-02 223919](https://github.com/BwShamlan/test/assets/98660242/9ae5b55a-1b7d-40bb-9227-5bd5bcb1eb6c)
#### Backup settings:
![incfinal](https://github.com/BwShamlan/test/assets/98660242/22cd77a7-e1ba-4179-bfe0-b6b036cc1523)

#### Backup files for incremental and full
![Screenshot 2023-12-02 220200](https://github.com/BwShamlan/test/assets/98660242/f5a62906-39fa-41c0-86eb-3e3586a9f962)

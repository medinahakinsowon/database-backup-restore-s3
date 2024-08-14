# database-backup-restore-s3



DB Backup and Restore from S3 for Microsoft SQL 


1 Create DB - Microsoft Sql Server (No Public Access, it will be accessed via a DB Bastion using SSM Fleet Manager)

2 Create EC2 - SSM Access Role which will be used during instance Creation

3 Create sql server backup role for RDS s3 access that will be attached to our option group

4 Launch a windows 2022 EC2 Instance

5 Connect DB to EC2 instance

6 Install SSMS on the EC2 instance
7 Create Database, Tables and Seed the Database. See all SQL Commands to do this below
8 Run Sample Queries to confirm all works
9 Create s3 bucket for backup
10 Create an option group by following below steps:

   a) In the Amazon RDS console, choose Option groups, and then choose Create option group.

   b) For Name, enter SQLServerrestore.

   c) For Description, enter SQLServerrestore.

   d) For Engine, choose sqlserver-se.

   e) For Major engine version, choose 14.00.

   f) Choose Create.

   g) Click on the created Option group and Add the SQLSERVER_BACKUP_RESTORE option and the sql-server-backup-restore role we had created earlier on to this option group to access S3 bucket.

       i) On the Option groups page, choose the option group that you created.

       ii) For Options, choose Add option. The Add option page opens.

       iii) For Option name, choose SQLSERVER_BACKUP_RESTORE.

       iv) For IAM role, choose the sql-server-backup-restore role.

   h) Modify your Amazon RDS for SQL Server DB instance and attach this option group.

   In the Amazon RDS console, choose Databases, and then choose our created database.

   i) Choose Modify. The Modify DB instance page opens.

   j) In the Additional configuration section, choose SQLServerrestore for Option group and choose or click apply immediately.



11 Use Below command to back-up, check status and restore


BACKUP

exec msdb.dbo.rds_backup_database 
@source_db_name='OnlineMerchStore', 
@s3_arn_to_backup_to='arn:aws:s3:::karo-bucket-name/july_bak;



STATUS

exec msdb.dbo.rds_task_status @db_name='OnlineMerchStore-clone';



RESTORE

exec msdb.dbo.rds_restore_database
@restore_db_name = 'OnlineMerchStore-clone',
@s3_arn_to_restore_from = 'arn:aws:s3:::karo-bucket-name/july_backup.bak';







DATABASE, TABLES and SEED DATA



-- Create Database

CREATE DATABASE OnlineMerchStore;

-- Use the created database

USE OnlineMerchStore;

-- Create Categories Table

CREATE TABLE Categories (

    CategoryID INT PRIMARY KEY,
    CategoryName VARCHAR(50)
    
);

-- Create Products Table

CREATE TABLE Products (

    ProductID INT PRIMARY KEY,
    Name VARCHAR(100),
    Description TEXT,
    Price DECIMAL(10, 2),
    CategoryID INT,
    StockQuantity INT,
    FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
    
);

-- Create Customers Table

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    Address VARCHAR(255),
    City VARCHAR(100),
    State VARCHAR(100),
    Country VARCHAR(100),
    Phone VARCHAR(20)
);

-- Create Orders Table

CREATE TABLE Orders (

    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    TotalAmount DECIMAL(10, 2),
    Status VARCHAR(50),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- Create OrderDetails Table

CREATE TABLE OrderDetails (

    OrderDetailID INT PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    Subtotal DECIMAL(10, 2),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

-- Create Payments Table

CREATE TABLE Payments (

    PaymentID INT PRIMARY KEY,
    OrderID INT,
    Amount DECIMAL(10, 2),
    PaymentDate DATE,
    PaymentMethod VARCHAR(50),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);





------ SEED DATA for Our Database ------

-- Use the created database

USE OnlineMerchStore;

-- Insert dummy data into Categories table

INSERT INTO Categories (CategoryID, CategoryName)
VALUES

    (1, 'Electronics'),
    
    (2, 'Clothing'),
    
    (3, 'Home & Kitchen');

-- Insert dummy data into Products table

INSERT INTO Products (ProductID, Name, Description, Price, CategoryID, StockQuantity)
VALUES

    (1, 'Laptop', 'High-performance laptop', 1200.00, 1, 50),
    
    (2, 'T-Shirt', 'Cotton T-Shirt', 25.00, 2, 100),
    
    (3, 'Coffee Maker', 'Automatic coffee maker', 45.00, 3, 30);

-- Insert dummy data into Customers table

INSERT INTO Customers (CustomerID, FirstName, LastName, Email, Address, City, State, Country, Phone)
VALUES

    (1, 'John', 'Doe', 'john@example.com', '123 Main St', 'Anytown', 'CA', 'USA', '123-456-7890'),
    
    (2, 'Jane', 'Smith', 'jane@example.com', '456 Elm St', 'Othertown', 'NY', 'USA', '987-654-3210');

-- Insert dummy data into Orders table

INSERT INTO Orders (OrderID, CustomerID, OrderDate, TotalAmount, Status)
VALUES

    (1, 1, '2023-01-10', 1200.00, 'Completed'),
    
    (2, 2, '2023-01-15', 70.00, 'Pending');

-- Insert dummy data into OrderDetails table

INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, Subtotal)
VALUES

    (1, 1, 1, 1, 1200.00),
    
    (2, 2, 2, 2, 50.00),
    
    (3, 2, 3, 1, 20.00);

-- Insert dummy data into Payments table

INSERT INTO Payments (PaymentID, OrderID, Amount, PaymentDate, PaymentMethod)
VALUES

    (1, 1, 1200.00, '2023-01-10', 'Credit Card'),
    
    (2, 2, 70.00, '2023-01-15', 'PayPal');

# Snicker & Suki Pet Company – Database Management System

**Course:** IS 3063 – Database Management  
**Project:** Term Project  
**Student:** Braulio Hurtado  
**Email:** braulio.hurtado@my.utsa.edu  

---

## Project Summary

This project involved designing and implementing a relational database management system (DBMS) for *Snicker and Suki*, a pet supply and service company. The company was experiencing issues with order fulfillment, inventory mismatches, and service scheduling due to poor data management. I helped the design and logic for the schema, SQL functionality, and business rules while providing a sense of leadership and direction for the team. This document outlines our approach, schema design, relationships, queries, and implementation plan.

---

## Business Context & Problem

**Snicker and Suki** operates both online and in-store. Their existing database fails to handle increasing data loads from online orders, service appointments, and in-store purchases. The system caused:
- Missed orders
- Miscounted inventory
- Misplaced appointments

### Project Goal:
Design a new database system to:
- Efficiently organize orders, products, and services
- Ensure accurate inventory tracking
- Support growth and future integration with shipping/analytics

---

## Entity Descriptions & Attributes

### `ORDER_DETAIL` (Associative Entity)
| Attribute        | Description                                   | Constraints      | Data Type     |
|------------------|-----------------------------------------------|------------------|---------------|
| OD_OrderID       | Order ID                                      | PK, FK           | INT(8)        |
| OD_ProID         | Product ID                                    | PK, FK           | INT(5)        |
| OD_ServID        | Service ID                                    | PK               | INT(7)        |
| OD_OrderPrice    | Total price for the line                      | NOT NULL, > 0    | INT(1000)     |
| OD_OrderQuantity | Quantity of product/service                   | NOT NULL, > 1    | INT(500)      |

---

### `SERVICES`
| Attribute         | Description                        | Constraints | Data Type     |
|-------------------|------------------------------------|-------------|---------------|
| SERV_ID           | Service ID                         | PK, FK      | INT(8)        |
| SERV_OrderDetail  | Order detail reference             | PK          | VARCHAR(100)  |
| SERV_Type         | Category/type of service           | None        | VARCHAR(100)  |
| SERV_Description  | Service description                | None        | VARCHAR(500)  |

---

### `PRODUCT`
| Attribute         | Description                        | Constraints       | Data Type     |
|-------------------|------------------------------------|-------------------|---------------|
| PRO_ID            | Product ID                         | PK, FK            | INT(5)        |
| PRO_OrderDetail   | Order detail reference             | PK                | VARCHAR(100)  |
| PRO_Quantity      | Quantity ordered                   | NOT NULL, >=1     | INT(500)      |
| PRO_Price         | Price per unit                     | NOT NULL, > 0     | INT(1000)     |
| PRO_Inventory     | Current inventory count            | NOT NULL, >= 0    | INT(1000)     |
| PRO_Description   | Product description                | None              | VARCHAR(100)  |

---

### `CUSTOMER`
| Attribute          | Description              | Constraints            | Data Type     |
|--------------------|--------------------------|-------------------------|---------------|
| CUST_ID            | Customer ID              | PK                      | INT(6)        |
| CUST_Name          | Customer name            |                         | VARCHAR(40)   |
| CUST_PhoneNumber   | Phone number             | UNIQUE, NOT NULL        | VARCHAR(15)   |
| CUST_Email         | Email address            | UNIQUE, NOT NULL        | VARCHAR(25)   |
| CUST_Address       | Mailing address          | Unique, nullable        | VARCHAR(50)   |

---

### `ORDER`
| Attribute          | Description              | Constraints            | Data Type     |
|--------------------|--------------------------|-------------------------|---------------|
| ORDER_ID           | Order ID                 | PK, FK                  | INT(8)        |
| ORDER_CustomerID   | Customer ID              | FK                      | INT(6)        |
| ORDER_PaymentID    | Payment ID               | FK                      | INT(6)        |
| ORDER_Status       | Order status             | ENUM                    | VARCHAR(20)   |
| ORDER_Date         | Date of order            | NOT NULL                | DATE          |

---

### `PAYMENT`
| Attribute           | Description             | Constraints           | Data Type        |
|---------------------|-------------------------|------------------------|------------------|
| PAY_ID              | Payment ID              | PK, FK                 | INT(8)           |
| PAY_Status          | Payment status          | ENUM                   | VARCHAR(50)      |
| PAY_Method          | Payment method          | NOT NULL, > 0          | VARCHAR(50)      |
| PAY_Amount          | Amount paid             | NOT NULL, > 0          | DECIMAL(10,2)    |
| PAY_Date            | Payment date            | NOT NULL               | DATE             |
| PAY_TransactionID   | Unique transaction ID   | UNIQUE                 | VARCHAR(10)      |

---

## Entity Relationships

- **Customer ↔ Order:** 1:M  
- **Order ↔ Order Detail:** 1:M  
- **Order ↔ Payment:** 1:1  
- **Order Detail ↔ Product/Service:** 1:M each  
- Products and services are both recorded through `OrderDetail`, functioning as an intersection point.

> Business Rule: A customer can either order a product or schedule a service, but not both in a single order line.

---

## ERD Diagram

> ![image](https://github.com/user-attachments/assets/d61b635c-263f-4d3a-a6d4-0e6eb28c5461)


---

## Application Functionality

### 1. Order & Inventory Management

- Track stock levels

- Auto-decrement inventory

- Avoid missed/doubled orders

### 2. Search & Analysis

- Identify top-selling items

- Group by customer location

- Analyze service usage

### 3. Customer Management

- Store and update contact info

- Analyze purchasing trends

### 4. Future Features

- Shipping integration

- Customer delivery notifications

### 5. Interface Goals

- Simple UI for data entry and retrieval

---

 ## SQL Functionality (Examples)

- Total payments made

      SELECT SUM(Pay_amount) AS Total_Payments FROM Payment;

- Add customer, order, update product price

      BEGIN TRANSACTION;

      INSERT INTO Customer (Cust_ID, Name, Email) VALUES (123, 'Braulio H', 'braulio@example.com');
      
      INSERT INTO Order (Order_ID, Cust_ID, OrderDate, TotalAmount) VALUES (456, 123, '2023-11-10', 150.75);
      
      UPDATE Product SET Price = 29.99 WHERE ProductID = 789;
      
      COMMIT;

- Top profit product in Texas

      SELECT Customer.Cust_ID, Cust_Name, Product.Pro_ID,
      
      SUM(OrderItem.Quantity * Product.Price) AS TotalProfit
      
      FROM Customer
      
      JOIN Orders ON Customer.Cust_ID = Orders.Cust_ID
      
      JOIN OrderItem ON Orders.Order_ID = OrderItem.Order_ID
      
      JOIN Product ON OrderItem.ProductID = Product.ProductID
      
      WHERE Customer.Location = 'Texas'
      
      GROUP BY Product.Pro_ID
      
      ORDER BY TotalProfit DESC;

## Table Creation (DDL)

    CREATE TABLE Customer (
        CustomerID INT PRIMARY KEY,
        CustomerName VARCHAR(40) UNIQUE NOT NULL,
        PhoneNumber VARCHAR(15) UNIQUE NOT NULL,
        CustomerEmail VARCHAR(25) UNIQUE NOT NULL,
        CustomerAddress VARCHAR(50)
    );

    CREATE TABLE Payment (
        PaymentID INT PRIMARY KEY,
        PaymentStatus VARCHAR(50) CHECK (PaymentStatus IN ('Valid')),
        PaymentMethod VARCHAR(50) CHECK (LEN(PaymentMethod) > 0),
        PaymentAmount DECIMAL(10,2) NOT NULL CHECK (PaymentAmount > 0),
        PaymentDate DATE NOT NULL,
        TransactionID VARCHAR(10) UNIQUE
    );

    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerID INT REFERENCES Customer(CustomerID),
        PaymentID INT REFERENCES Payment(PaymentID),
        OrderStatus VARCHAR(20),
        OrderDate DATE NOT NULL
    );

---
## How to Run This Project in Microsoft Access

Follow these steps to build the database using the provided SQL:

1. **Open Microsoft Access**  
   Launch Microsoft Access on your computer.

2. **Create a New Database**  
   - Click on **File** → **New**  
   - Choose **Blank Database**  
   - Name your file and click **Create**

3. **Open SQL View**  
   - Go to the **Database Tools** tab  
   - Click on **Query Design**  
   - In the "Show Table" dialog, close it without adding any tables  
   - Then click **SQL View** (in the Query Tools ribbon)

4. **Paste SQL Code**  
   - Copy the SQL schema from this document  
   - Paste it into the SQL View editor

5. **Run the Query**  
   - Click the **Run** button (red exclamation mark), or press `Ctrl + Enter`

6. **Check for Errors**  
   - Look at the message bar at the bottom for any syntax issues  
   - If successful, tables, keys, and relationships will be created automatically

![Screenshot 2025-07-09 114255](https://github.com/user-attachments/assets/3908c40c-cc89-4b17-8ac4-cbf5ff107135)


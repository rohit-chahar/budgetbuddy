
# Budget Buddy

> **A Smart Student Budget & Expense Tracker with Real-Time Analytics and Database Persistence**

##  Project Overview

**Budget Buddy** is a financial management application specifically designed for college and university students. Students managing hostel allowances, PG rents, and daily expenditures frequently face unmonitored cash leaks and month-end financial deficits.

This project solves that friction by providing:

1. **User Authentication**: Secure multi-student registration and login system with user-isolated data.

2. **Budget Threshold Alerting**: Smart alerts triggered dynamically when expenditures reach **80% (Warning)** and **100% (Critical Budget Exceeded)**.

3. **Data Visualization**: Real-time category-wise spending breakdown with interactive charts.

4. **Relational Database Integrity**: Fully backed by MySQL with ACID-compliant transactions, foreign key constraints, and cascading deletes.

5. **Flexible Execution Modes**: Run as a native **Java Swing GUI Desktop Application** or as a lightweight **Web Dashboard on `http://localhost:8080`**.

##  System Architecture

Budget Buddy adopts a decoupled 3-tier architecture:

```
┌────────────────────────────────────────────────────────┐
│                   PRESENTATION TIER                    │
│   • Java Swing GUI (JFrame, JTable, Graphics2D)       │
│   • Localhost Web Dashboard (HTML5, CSS3, Chart.js)    │
└──────────────────────────┬─────────────────────────────┘
                           │ HTTP / Event Dispatch Thread
┌──────────────────────────▼─────────────────────────────┐
│                    LOGIC & API TIER                    │
│   • Session & User Context Handler                     │
│   • Threshold Calculation Engine (80% / 100% Alerts)   │
│   • Category Aggregation Engine                        │
└──────────────────────────┬─────────────────────────────┘
                           │ JDBC (Type-4 Driver)
┌──────────────────────────▼─────────────────────────────┐
│                   PERSISTENCE TIER                     │
│   • MySQL Database: `budget_buddy_db`                 │
│   • Tables: `users`, `budget_settings`, `expenses`    │
└────────────────────────────────────────────────────────┘

```

##  Key Features

* ** Student Authentication System**: Dedicated Sign Up and Login portals. Every student's records (budget, expenses) are strictly isolated via `user_id` foreign keys.

* ** Dynamic Monthly Budgeting**: Students configure their target allowance; remaining balance updates in real-time.

* ** Dual Threshold Safety Net**:

  * **80% Warning**: Alert banner / dialog notifying the student to slow down discretionary spending.

  * **100% Deficit Alert**: Critical notification when expenditures outpace the set allowance.

* ** Structured Categorization**: Log transactions under student-centric categories:

  * *Food & Canteen*

  * *Books & Stationery*

  * *Rent & PG*

  * *Travel*

  * *College Fees*

  * *Leisure & Entertainment*

* ** Visual Analytics**: Real-time category breakdown rendered via custom `Graphics2D` vector arcs (Swing) and `Chart.js` (Web edition).

* ** Real-time CRUD Operations**: Instantly record, view, and delete logged entries with immediate synchronization to the database.

##  Database Schema (ER Structure)

The relational schema uses MySQL with referential integrity constraints:

```
CREATE DATABASE IF NOT EXISTS budget_buddy_db;
USE budget_buddy_db;

-- 1. Users Table (Authentication)
CREATE TABLE IF NOT EXISTS users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Budget Settings (1-to-1 Relationship with User)
CREATE TABLE IF NOT EXISTS budget_settings (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT UNIQUE,
    monthly_budget DOUBLE NOT NULL DEFAULT 0.0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT fk_budget_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 3. Expenses Table (1-to-Many Relationship with User)
CREATE TABLE IF NOT EXISTS expenses (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    expense_date VARCHAR(20) NOT NULL,
    item_title VARCHAR(150) NOT NULL,
    category VARCHAR(50) NOT NULL,
    amount DOUBLE NOT NULL,
    CONSTRAINT fk_expense_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

```

##  Repository Directory Structure

```
StudentExpenseTracker/
├── build.xml                     # Apache Ant build script
├── manifest.mf                   # Application manifest
├── expenses_data.csv             # Fallback flat-file storage
├── nbproject/                    # NetBeans project configuration
│   ├── build-impl.xml
│   ├── genfiles.properties
│   ├── project.properties
│   └── project.xml
└── src/
    └── studentexpensetracker/
        ├── DBConnection.java            # JDBC connection manager
        ├── LoginDialog.java             # Swing authentication modal
        ├── StudentExpenseTracker.java   # Swing desktop GUI application
        └── BudgetBuddyWebServer.java    # Embedded HTTP server for localhost:8080

```

##  Getting Started & Setup Guide

### 1. Prerequisites

* **Java Development Kit (JDK)**: JDK 17, 21, or higher

* **IDE**: Apache NetBeans (v12, 18, 20+)

* **Database**: MySQL Server (v8.0+)

* **JDBC Driver**: `mysql-connector-j-8.x.x.jar`

### 2. Configure Database Credentials

Open `src/studentexpensetracker/DBConnection.java` and enter your MySQL root credentials:

```
private static final String URL = "jdbc:mysql://localhost:3306/budget_buddy_db?useSSL=false&allowPublicKeyRetrieval=true";
private static final String USER = "root";
private static final String PASSWORD = "your_mysql_password"; // <-- Update here

```

### 3. Open Project in NetBeans

1. Launch **NetBeans**.

2. Click **File** > **Open Project...**

3. Browse and select the `StudentExpenseTracker` folder.

4. Expand **Libraries** in the Projects pane and confirm `mysql-connector-j` is present. If missing:

   * Right-click **Libraries** > **Add JAR/Folder...** > Select `mysql-connector-j.jar`.

### 4. Running the Application

#### Option A: Run as Desktop Application (Java Swing)

1. In NetBeans, open `src/studentexpensetracker/StudentExpenseTracker.java`.

2. Press **Shift + F6** (or Right-click > **Run File**).

3. The **Authentication Dialog** will appear. Register a new account on the **Sign Up** tab, then sign in to access the tracker.

#### Option B: Run on Web (`http://localhost:8080`)

1. Open `src/studentexpensetracker/BudgetBuddyWebServer.java`.

2. Press **Shift + F6** (or Right-click > **Run File**).

3. Open your browser and navigate to:

   ```
   http://localhost:8080
   
   ```

##  Test Cases Verified

| Test ID | Module | Scenario | Expected Result | Status | 
| ----- | ----- | ----- | ----- | ----- | 
| **TC-01** | Auth | Register user with new email | Account saved to `users` table; successful prompt | **Passed** | 
| **TC-02** | Auth | Login with incorrect password | Access denied with validation dialog | **Passed** | 
| **TC-03** | Budget | Input negative budget amount | Handled gracefully with NumberFormatException message | **Passed** | 
| **TC-04** | Expense | Spending exceeds 80% of budget | Warning notification pops up; warning flag set | **Passed** | 
| **TC-05** | Expense | Spending reaches/exceeds 100% | Critical overspending banner / alert dialog displayed | **Passed** | 
| **TC-06** | Chart | Insert expenses across 3 categories | Pie chart recalibrates angles and percentage labels | **Passed** | 
| **TC-07** | Persistence | Restart application / refresh page | User state and transactions reload correctly via MySQL | **Passed** | 

##  Key Technical Concepts for University Viva

* **Object-Oriented Programming (OOP)**: Encapsulation of domain logic, class separation (`DBConnection`, `LoginDialog`, `StudentExpenseTracker`).

* **Event-Driven Programming**: Utilization of `ActionListener` interfaces and Java lambdas (`e -> addExpense()`) to handle user UI interactions asynchronously.

* **Relational Integrity**: Foreign key constraints with `ON DELETE CASCADE` ensuring student accounts and expenditures never leave orphan records.

* **SQL Injection Prevention**: Rigorous usage of parameterized `java.sql.PreparedStatement` over raw string concatenation.

* **Custom Vector Graphics**: Mathematical projection of angle sectors using `Arc2D.Double` and anti-aliasing via `Graphics2D` rendering hints.

##  Author

* **Student Name**: Tushar Thakur

* **Degree**: Master of Computer Applications (MCA)

* **Institution**: Chandigarh University


##  License

This project is developed for academic evaluation and educational purposes under the MIT License.

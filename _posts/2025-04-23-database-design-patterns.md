---
title: "Database Design Patterns: Building Robust Data Models"
excerpt: "Master essential database design patterns for creating efficient, scalable data architectures."
categories:
  - Technical Tutorials
  - Database Design
tags:
  - database
  - design-patterns
  - sql
  - data-modeling
date: 2025-04-23
---

## Introduction

Good database design is the foundation of any robust application. Poor design leads to performance issues, data anomalies, and maintenance headaches. This guide covers essential patterns for building effective data models.

## Normalization Basics

Normalization reduces data redundancy and improves data integrity by organizing data into related tables.

### First Normal Form (1NF)
Eliminate repeating groups (atomic values only).

```sql
-- Bad: Non-atomic values
CREATE TABLE orders (
  order_id INT,
  items VARCHAR(500)  -- "Item1, Item2, Item3"
);

-- Good: Atomic values
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT,
  order_date DATE
);

CREATE TABLE order_items (
  item_id INT PRIMARY KEY,
  order_id INT,
  product_id INT,
  quantity INT,
  FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

### Second Normal Form (2NF)
Remove partial dependencies (non-key attributes must depend on full primary key).

```sql
-- Bad: Partial dependency
CREATE TABLE student_courses (
  student_id INT,
  course_id INT,
  instructor_name VARCHAR(100),  -- Depends only on course_id
  PRIMARY KEY (student_id, course_id)
);

-- Good: Separate table for course information
CREATE TABLE student_courses (
  enrollment_id INT PRIMARY KEY,
  student_id INT,
  course_id INT,
  FOREIGN KEY (student_id) REFERENCES students(student_id),
  FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

CREATE TABLE courses (
  course_id INT PRIMARY KEY,
  course_name VARCHAR(100),
  instructor_name VARCHAR(100)
);
```

### Third Normal Form (3NF)
Remove transitive dependencies (non-key attributes depend only on primary key).

```sql
-- Bad: Transitive dependency
CREATE TABLE employees (
  employee_id INT PRIMARY KEY,
  employee_name VARCHAR(100),
  department_id INT,
  department_name VARCHAR(100),     -- Depends on department_id, not employee_id
  department_budget DECIMAL(10,2)
);

-- Good: Separate tables
CREATE TABLE employees (
  employee_id INT PRIMARY KEY,
  employee_name VARCHAR(100),
  department_id INT,
  FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

CREATE TABLE departments (
  department_id INT PRIMARY KEY,
  department_name VARCHAR(100),
  department_budget DECIMAL(10,2)
);
```

## Entity Relationship Patterns

### One-to-Many Relationship

```sql
-- User has many orders
CREATE TABLE users (
  user_id INT PRIMARY KEY,
  username VARCHAR(100) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  user_id INT NOT NULL,
  order_date DATE,
  total_amount DECIMAL(10,2),
  FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Query: Get user with all orders
SELECT u.username, o.order_id, o.total_amount
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE u.user_id = 123;
```

### Many-to-Many Relationship

```sql
-- Student attends many courses, course has many students
CREATE TABLE students (
  student_id INT PRIMARY KEY,
  student_name VARCHAR(100)
);

CREATE TABLE courses (
  course_id INT PRIMARY KEY,
  course_name VARCHAR(100),
  credits INT
);

CREATE TABLE enrollments (
  enrollment_id INT PRIMARY KEY,
  student_id INT NOT NULL,
  course_id INT NOT NULL,
  grade CHAR(1),
  UNIQUE(student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES students(student_id),
  FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

-- Query: Get student's courses
SELECT s.student_name, c.course_name, e.grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
WHERE s.student_id = 456;
```

### Self-Referencing Relationship

```sql
-- Employee reports to a manager
CREATE TABLE employees (
  employee_id INT PRIMARY KEY,
  employee_name VARCHAR(100),
  manager_id INT,
  FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

-- Query: Get employee with manager info
SELECT e.employee_name, m.employee_name as manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
WHERE e.employee_id = 10;
```

## Advanced Patterns

### Soft Deletes

Mark records as deleted instead of removing them (useful for audit trails and data recovery).

```sql
CREATE TABLE users (
  user_id INT PRIMARY KEY,
  username VARCHAR(100),
  email VARCHAR(255),
  deleted_at TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Mark as deleted
UPDATE users SET deleted_at = CURRENT_TIMESTAMP WHERE user_id = 123;

-- Query active users only
SELECT * FROM users WHERE deleted_at IS NULL;

-- Query deleted users
SELECT * FROM users WHERE deleted_at IS NOT NULL;
```

### Audit Trail

Track all changes to important records.

```sql
CREATE TABLE users (
  user_id INT PRIMARY KEY,
  username VARCHAR(100),
  email VARCHAR(255),
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE user_audit_log (
  audit_id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT,
  action VARCHAR(50),  -- INSERT, UPDATE, DELETE
  old_values JSON,
  new_values JSON,
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  changed_by INT,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (changed_by) REFERENCES users(user_id)
);

-- Trigger to log changes
DELIMITER $$

CREATE TRIGGER user_update_trigger
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
  INSERT INTO user_audit_log (user_id, action, old_values, new_values, changed_by)
  VALUES (
    NEW.user_id,
    'UPDATE',
    JSON_OBJECT('username', OLD.username, 'email', OLD.email),
    JSON_OBJECT('username', NEW.username, 'email', NEW.email),
    CURRENT_USER_ID()
  );
END$$

DELIMITER ;
```

### Polymorphic Associations

A record can belong to different types of entities.

```sql
-- Comments can be on posts, videos, or users
CREATE TABLE comments (
  comment_id INT PRIMARY KEY,
  content TEXT,
  created_by INT,
  -- Polymorphic reference
  commentable_type VARCHAR(50),  -- 'post', 'video', 'user'
  commentable_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (created_by) REFERENCES users(user_id)
);

-- Query comments on a post
SELECT * FROM comments 
WHERE commentable_type = 'post' AND commentable_id = 123;
```

## Indexing Strategy

### Primary Key Index (Automatic)

```sql
CREATE TABLE products (
  product_id INT PRIMARY KEY,  -- Automatically indexed
  product_name VARCHAR(255),
  price DECIMAL(10,2)
);
```

### Single Column Index

```sql
-- Index frequently searched columns
CREATE INDEX idx_products_name ON products(product_name);

-- This query will use the index
SELECT * FROM products WHERE product_name = 'Widget';
```

### Composite Index

```sql
-- For queries filtering on multiple columns
CREATE INDEX idx_orders_user_date 
ON orders(user_id, order_date);

-- Query using both columns
SELECT * FROM orders 
WHERE user_id = 123 AND order_date > '2025-01-01';
```

### Unique Index

```sql
-- Ensure uniqueness while improving lookups
CREATE UNIQUE INDEX idx_users_email 
ON users(email);
```

### Full-Text Index

```sql
CREATE FULLTEXT INDEX idx_articles_content 
ON articles(title, content);

-- Natural language search
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('database design' IN NATURAL LANGUAGE MODE);
```

## Data Type Selection

### Choosing Appropriate Types

```sql
CREATE TABLE optimal_types (
  -- Text
  status CHAR(1),                    -- Fixed-length known set ('A', 'I', 'P')
  description VARCHAR(500),          -- Variable-length text
  long_text TEXT,                    -- Large blocks of text
  
  -- Numeric
  quantity INT,                      -- Whole numbers
  price DECIMAL(10,2),               -- Money values (not float!)
  percentage FLOAT,                  -- Approximate decimals
  
  -- Date/Time
  birth_date DATE,                   -- Just date
  created_at TIMESTAMP,              -- Date and time with timezone
  duration TIME,                     -- Time duration
  
  -- Other
  is_active BOOLEAN,                 -- True/False
  metadata JSON,                     -- Flexible key-value data
  
  -- IDs
  id BIGINT PRIMARY KEY AUTO_INCREMENT  -- Large auto-increment IDs
);
```

## Query Optimization

### Using EXPLAIN

```sql
-- Analyze query execution plan
EXPLAIN SELECT * FROM orders 
WHERE user_id = 123 AND order_date > '2025-01-01';

-- Output shows if indexes are used, rows examined, etc.
```

### Common Optimization Techniques

```sql
-- 1. Use column list instead of SELECT *
SELECT user_id, username, email FROM users;  -- Good
SELECT * FROM users;                          -- Avoid

-- 2. Filter early (WHERE before JOIN)
SELECT * FROM orders o
WHERE o.order_date > '2025-01-01'             -- Filter first
JOIN users u ON o.user_id = u.user_id;       -- Then join

-- 3. Avoid functions on indexed columns
SELECT * FROM orders WHERE YEAR(order_date) = 2025;  -- Bad
SELECT * FROM orders WHERE order_date >= '2025-01-01' 
                      AND order_date < '2026-01-01';  -- Good

-- 4. Use LIMIT for pagination
SELECT * FROM products ORDER BY product_id LIMIT 10 OFFSET 0;
```

## Conclusion

Effective database design requires careful planning and understanding of these patterns:

✅ Normalize data to reduce redundancy  
✅ Model relationships appropriately  
✅ Choose correct data types  
✅ Create strategic indexes  
✅ Optimize queries for performance  
✅ Implement audit trails for critical data  

**Next steps:** Audit your current database schema against these patterns and optimize incrementally.

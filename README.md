📦 Product API Review & System Design

Candidate: Mahesh Kumbhar

📌 Summary

This repository contains analysis, improvements, and design decisions for a backend case study. It includes:

Code review of an existing API
Identification of issues and their impact
Improved solution approach
Database design
API design for low-stock alerts
🧩 Part 1: Code Review & Debugging
🚨 Issues Identified
No input validation (missing/null values not handled)
SKU uniqueness not enforced
Multiple database commits (non-atomic operations)
No error handling (failures not managed)
Incorrect assumption: product belongs to only one warehouse
Price not validated (negative or invalid values allowed)
Inventory created without checking existing records
⚠️ Impact in Production
Duplicate SKUs → Data inconsistency
Partial data saved → Product created but inventory missing
Application crashes → Due to missing input validation
Incorrect stock tracking
Security risks → Invalid or malicious input
✅ Improved Approach (Logic)

Since the original code lacks production safety, the improved logic ensures data consistency and reliability:

✔️ Steps:
Validate required fields (name, sku, price, etc.)
Check if SKU already exists
Start a database transaction
Create product
Create inventory entry
Commit both together (atomic operation)
Rollback in case of any failure
💡 Pseudo Code
Get request data

If name or sku is missing:
    return error

Check if SKU already exists:
    If yes → return error

Start transaction

Create product
Create inventory for that product

If everything is successful:
    Commit transaction
Else:
    Rollback transaction

Return success response
🗄️ Part 2: Database Design
📊 Tables Structure
Companies
id (Primary Key)
name
Warehouses
id (Primary Key)
company_id (Foreign Key)
name
Products
id (Primary Key)
name
sku (Unique)
price
Inventory
id (Primary Key)
product_id (Foreign Key)
warehouse_id (Foreign Key)
quantity
Suppliers
id (Primary Key)
name
contact_email
Product_Suppliers
product_id (Foreign Key)
supplier_id (Foreign Key)
Inventory_Logs
id
product_id
warehouse_id
change
timestamp
❓ Missing Requirements (Clarifications Needed)
What defines “recent sales activity”?
Can a product have multiple suppliers?
How are bundle products handled?
Is the threshold fixed or dynamic?
Can inventory go negative?
🧠 Design Decisions
SKU is unique to prevent duplication
Separate Inventory table for multi-warehouse support
Inventory logs for tracking stock changes
Use of foreign keys for data integrity
Indexing on SKU and product_id for performance optimization
🚀 Part 3: API Implementation
📡 Endpoint
GET /api/companies/{company_id}/alerts/low-stock
⚙️ Approach
Extract company_id from request
Fetch all warehouses of the company
Retrieve inventory for those warehouses
Check each product’s quantity against threshold
Add low-stock products to alert list
Attach supplier information
Return alerts with total count
💡 Pseudo Code
Get company_id

Fetch warehouses of company

For each warehouse:
    Get inventory

For each inventory item:
    If quantity < threshold:
        Add to alerts list

Return:
    alerts list
    total alerts count
⚠️ Edge Cases
No warehouses found
No inventory data available
Product has no supplier
Stock is zero or negative
Threshold value missing
🔍 Assumptions
Default threshold = 20
Each product has one primary supplier
“Recent sales activity” not defined → basic filtering assumed
📎 Reference Document

Original submission (PDF):


✅ Final Note

This solution focuses on:

Data consistency (atomic operations)
Scalable database design
Clean and maintainable backend logic

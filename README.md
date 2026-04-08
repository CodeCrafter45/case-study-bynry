# 📦 Product API Review & System Design  
**Candidate:** Mahesh Kumbhar  

---

## 📌 Summary

This repository contains analysis, improvements, and design decisions for a backend case study.

### It includes:
- Code review of an existing API  
- Identification of issues and their impact  
- Improved solution approach  
- Database design  
- API design for low-stock alerts  

---

## 🧩 Part 1: Code Review & Debugging

### 🚨 Issues Identified

- No input validation (missing/null values not handled)  
- SKU uniqueness not enforced  
- Multiple database commits (non-atomic operations)  
- No error handling (failures not managed)  
- Incorrect assumption: product belongs to only one warehouse  
- Price not validated (negative or invalid values allowed)  
- Inventory created without checking existing records  

---

### ⚠️ Impact in Production

- Duplicate SKUs → Data inconsistency  
- Partial data saved → Product created but inventory missing  
- Application crashes → Due to missing input validation  
- Incorrect stock tracking  
- Security risks → Invalid or malicious input  

---

### ✅ Improved Approach (Logic)

The improved solution ensures **data consistency and reliability**.

#### Steps:
1. Validate required fields  
2. Check if SKU already exists  
3. Start a database transaction  
4. Create product  
5. Create inventory entry  
6. Commit both together (atomic operation)  
7. Rollback in case of failure  

---

### 💡 Pseudo Code

```python
data = request.json

if not data.get("name") or not data.get("sku"):
    return error

if sku_exists(data["sku"]):
    return error

start_transaction()

try:
    product = create_product(data)
    create_inventory(product.id, data)

    commit_transaction()
    return success

except:
    rollback_transaction()
    return error
```

---

## 🗄️ Part 2: Database Design

### 📊 Tables Structure

#### Companies
- id (Primary Key)  
- name  

#### Warehouses
- id (Primary Key)  
- company_id (Foreign Key)  
- name  

#### Products
- id (Primary Key)  
- name  
- sku (Unique)  
- price  

#### Inventory
- id (Primary Key)  
- product_id (Foreign Key)  
- warehouse_id (Foreign Key)  
- quantity  

#### Suppliers
- id (Primary Key)  
- name  
- contact_email  

#### Product_Suppliers
- product_id (Foreign Key)  
- supplier_id (Foreign Key)  

#### Inventory_Logs
- id  
- product_id  
- warehouse_id  
- change  
- timestamp  

---

### ❓ Missing Requirements

- What defines “recent sales activity”?  
- Can a product have multiple suppliers?  
- How are bundle products handled?  
- Is threshold fixed or dynamic?  
- Can inventory go negative?  

---

### 🧠 Design Decisions

- SKU is unique to prevent duplication  
- Separate inventory table for multi-warehouse support  
- Inventory logs for tracking stock changes  
- Foreign keys maintain relationships  
- Indexing improves performance  

---

## 🚀 Part 3: API Implementation

### 📡 Endpoint

```
GET /api/companies/{company_id}/alerts/low-stock
```

---

### ⚙️ Approach

1. Extract company_id  
2. Fetch warehouses  
3. Get inventory data  
4. Compare quantity with threshold  
5. Add low-stock items to alerts  
6. Attach supplier info  
7. Return response  

---

### 💡 Pseudo Code

```python
company_id = get_company_id()

warehouses = get_warehouses(company_id)

alerts = []

for warehouse in warehouses:
    inventory = get_inventory(warehouse)

    for item in inventory:
        if item.quantity < threshold:
            alerts.append(item)

return {
    "alerts": alerts,
    "count": len(alerts)
}
```

---

### ⚠️ Edge Cases

- No warehouses found  
- No inventory data  
- Product has no supplier  
- Stock is zero or negative  
- Threshold missing  

---

### 🔍 Assumptions

- Default threshold = 20  
- One primary supplier per product  
- Basic filtering for sales activity  

---

## ✅ Final Note

This solution focuses on:
- Data consistency (atomic transactions)  
- Scalable design  
- Clean backend logic  

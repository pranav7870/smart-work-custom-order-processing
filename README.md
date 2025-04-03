# Custom Order Processing Module

## Overview
The **Custom Order Processing** module allows external systems to update Magento order statuses via a REST API. It also logs order status changes and triggers email notifications for shipped orders.

## Features
- Custom REST API to update order status.
- Order status change observer to log transitions in a custom table.
- Automatic email notifications when an order is marked as shipped.
- Follows Magento 2 coding standards (PSR-4, Dependency Injection, No direct ObjectManager usage).

---

## Installation & Setup
### **1. Enable the Module**
```sh
bin/magento module:enable Vendor_CustomOrderProcessing
bin/magento setup:upgrade
bin/magento cache:flush
```

### **2. Verify Module Status**
```sh
bin/magento module:status | grep Vendor_CustomOrderProcessing
```

### **3. Run Setup Upgrade**
```sh
bin/magento setup:upgrade
```

### **4. Deploy Static Content (For Production Mode)**
```sh
bin/magento setup:static-content:deploy -f
```

### **5. Reindex**
```sh
bin/magento indexer:reindex
```

---

## API Usage
### **Update Order Status API**
**Endpoint:**
```plaintext
POST /rest/V1/custom-order/update-status
```
**Request Body:**
```json
{
    "orderIncrementId": "100000001",
    "status": "processing"
}
```
**Response:**
```json
{
    "message": "Order status updated successfully."
}
```
**Authentication:** Use a **Bearer Token** in Postman or CLI.

---

## Database Schema
A custom table `custom_order_status` is created with the following structure:
| Column      | Type        | Description |
|------------|------------|-------------|
| log_id      | INT (PK)   | Unique log identifier |
| order_id    | INT (FK)   | Related order ID |
| old_status  | VARCHAR(32) | Previous order status |
| new_status  | VARCHAR(32) | Updated order status |
| created_at  | TIMESTAMP  | Status change timestamp |

---

## Architectural Decisions
### **1. Use of Repository Pattern**
- Instead of direct SQL queries, we use `OrderRepositoryInterface` for retrieving and updating order data.

### **2. Dependency Injection (DI) Over ObjectManager**
- All dependencies (Order Repository, Logger, Database Connection) are injected via DI, making the module testable and maintainable.

### **3. Event-Driven Order Status Logging**
- Uses an **observer** on `sales_order_save_after` to log status changes automatically.

### **4. API Security & Validation**
- Uses Magento’s built-in **Bearer Token authentication** for securing API requests.
- Validates **order existence** and **status transition rules** before updating.

---

## Troubleshooting
**1. API returns `"orderIncrementId" is required` error:**
- Ensure the request body contains `orderIncrementId` and `status` fields.

**2. Module not working after installation:**
- Run `bin/magento setup:upgrade && bin/magento cache:flush`
- Check module status using `bin/magento module:status`

**3. No email notification on shipped orders:**
- Verify email configuration in Magento admin (`Stores > Configuration > Sales Emails`).
- Check the log file (`var/log/system.log`) for errors.

---

## Support
For issues or enhancements, create a pull request or raise an issue in the repository.


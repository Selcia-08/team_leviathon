# 🔐 Authentication System - User Guide

## Overview
The Delivery Partner system now has complete authentication and authorization with three user types:
- **Admin**: Full system access
- **Driver**: Container management and routing
- **Producer**: Inventory and package management

## 🎯 Quick Start

### Admin Login
- **Email**: `me@gmail.com`
- **User Type**: Admin
- **Password**: `me@gmail.com` (email as password)
- **Access**: All pages and features

### Driver Login (Examples)
- **Email**: `driver1@delivery.com`, `driver2@delivery.com`, etc.
- **User Type**: Driver
- **Password**: `driver` (same for all drivers)
- **Access**: Container dashboard, routing, transactions

### Producer Login (Examples)
- **Email**: `producer1@delivery.com`, `producer2@delivery.com`, etc.
- **User Type**: Producer
- **Password**: `producer` (same for all producers)
- **Access**: Inventory, packages, producer transactions

## 📋 Features

### Public Pages (No Login Required)
- ✅ Home page (`/`)

### Authentication Pages
- 🔑 Login: `/auth/login`
- 📝 Register Driver: `/auth/register/driver`
- 📝 Register Producer: `/auth/register/producer`
- 🚪 Logout: `/auth/logout`

### Admin-Only Pages
- All producers, packages, containers
- Full blockchain access
- All configuration pages

### Driver Pages
- Container list
- My Dashboard (`/container/{ID}/routing`)
- Task management
- Transaction completion
- Blockchain viewer

### Producer Pages
- Producer list
- My Inventory (`/producer/{ID}/inventory`)
- My Transactions (`/producer/{ID}/transactions`)
- Package creation
- Blockchain viewer

## 🔐 Security Features

### Session Management
- 24-hour session duration
- Secure cookie-based sessions
- Auto-logout on session expiry

### Error Pages
- **401 Unauthorized**: Not logged in
- **403 Forbidden**: Insufficient permissions
- Beautiful error pages with navigation

### Route Protection
All sensitive routes are protected with middleware:
- `requireAuth`: Must be logged in
- `requireAdmin`: Admin only
- `requireDriver`: Driver only
- `requireProducer`: Producer only
- `requireDriverOrAdmin`: Driver or Admin
- `requireProducerOrAdmin`: Producer or Admin

## 📝 Registration Process

### Register as Driver
1. Visit `/auth/register/driver`
2. Fill in:
   - Driver name
   - Email (will be used for login)
   - Phone number
   - Container code
   - Container specifications (sections, cost)
   - Current location (latitude/longitude)
3. System creates:
   - Container record
   - User account linked to container
4. Login with the registered email

### Register as Producer
1. Visit `/auth/register/producer`
2. Fill in:
   - Producer/Company name
   - Email (will be used for login)
   - Phone number
   - Inventory location (latitude/longitude)
3. System creates:
   - Producer record
   - User account linked to producer
4. Login with the registered email

## 🎨 Navigation Based on User Type

### Admin Navigation
- Home
- Producers
- Packages
- Containers
- Add Container
- Blockchain
- **User Badge**: Shows "me (ADMIN)"

### Driver Navigation
- Home
- Containers
- My Dashboard (direct link to their container)
- Blockchain
- **User Badge**: Shows driver name and "(DRIVER)"

### Producer Navigation
- Home
- Producers
- My Inventory (direct link to their inventory)
- My Transactions
- Blockchain
- **User Badge**: Shows producer name and "(PRODUCER)"

## 🚀 Usage Examples

### Example 1: Driver Completing a Task
1. Login as `driver1@delivery.com`
2. Navigate to "My Dashboard" from navbar
3. See container routing with prioritized tasks
4. Click GO on top priority task
5. Complete transaction form
6. View transaction added to blockchain

### Example 2: Producer Managing Inventory
1. Login as `producer1@delivery.com`
2. Navigate to "My Inventory" from navbar
3. View current inventory items
4. Add new inventory with quantities
5. View transactions in "My Transactions"

### Example 3: Admin Oversight
1. Login as `me@gmail.com`
2. Access all pages from navbar
3. View all producers, containers, packages
4. Monitor blockchain for all transactions
5. Access admin-only analytics

## 🔧 Technical Details

### Database Schema
```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    user_type ENUM('ADMIN', 'DRIVER', 'PRODUCER') NOT NULL,
    reference_id INT DEFAULT NULL,  -- container_id or producer_id
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP NULL
);
```

### Session Data Structure
```javascript
req.session.user = {
    user_id: 1,
    name: "Driver Name",
    email: "driver1@delivery.com",
    user_type: "DRIVER",
    reference_id: 1  // Links to container_id or producer_id
}
```

### Available in All Templates
```javascript
res.locals.user  // Current user object or null
```

## 🎯 Testing the System

### Test Scenario 1: Driver Login
```
1. Navigate to http://localhost:5001/auth/login
2. Enter email: driver1@delivery.com
3. Select User Type: Driver
4. Enter password: driver
5. Click Login
6. Should redirect to /container/1/routing
```

### Test Scenario 2: Producer Login
```
1. Navigate to http://localhost:5001/auth/login
2. Enter email: producer1@delivery.com
3. Select User Type: Producer
4. Enter password: producer
5. Click Login
6. Should redirect to /producer/1/inventory
```

### Test Scenario 3: Unauthorized Access
```
1. Logout (if logged in)
2. Try to access /allContainers
3. Should see 401 Unauthorized error
4. Click "Go to Login" button
```

### Test Scenario 4: Forbidden Access
```
1. Login as driver1@delivery.com
2. Try to access /allPackages (admin only)
3. Should see 403 Forbidden error
4. Click "Go Home" or "Logout"
```

## 🛡️ Password Policy

### Current Password Rules
- **Admin**: Password is the email address (`me@gmail.com`)
- **Driver**: Password is `driver` (same for all drivers)
- **Producer**: Password is `producer` (same for all producers)

**Note**: This simplified password system is for development/testing only.

In production, you should:
- Add proper password hashing (bcrypt)
- Implement password strength requirements
- Add password reset functionality
- Enable two-factor authentication

## 📊 Current User Accounts

### Admin
- Email: `me@gmail.com`

### Drivers (10 accounts)
- `driver1@delivery.com` through `driver11@delivery.com`

### Producers (10 accounts)
- `producer1@delivery.com` through `producer10@delivery.com`

## 🔄 Logout
Click "Logout" in the navbar or visit `/auth/logout` to end your session.

## 🎨 UI Features
- Gradient backgrounds for different user types
- User badge showing name and role
- Dynamic navigation based on permissions
- Error pages with easy navigation back
- Responsive design for mobile devices

---

**System is ready!** Visit http://localhost:5001 to start using the authenticated delivery partner system! 🚀

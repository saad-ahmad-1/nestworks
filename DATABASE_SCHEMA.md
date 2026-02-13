# DATABASE SCHEMA DESIGN
## Core Principles & Analytics-Ready Architecture

---

## CORE DATABASE PRINCIPLES

1. **userId is a first-class field** - Every business record MUST have a userId
2. **User ownership** - Every business record belongs to a specific user
3. **No shared data** - No shared or global transactional data for normal users
4. **Analytics-ready** - Schema supports future analytics and aggregation queries

---

## ROLE-BASED ACCESS

Users can have roles:
- `user` (buyer) - Can place orders, make payments
- `seller` - Can create products, manage inventory, receive revenue
- `admin` - Full system access

**Note**: Same email can have multiple roles (e.g., user@example.com as both buyer and seller)

---

## ENTITIES & COLLECTIONS

### 1. users

**Purpose**: Core user entity supporting role-based access

**Fields**:
- `_id` (ObjectId) - Primary key, used as userId
- `name` (String, required) - User's full name
- `email` (String, required, unique per role) - User email
- `passwordHash` (String, required) - Hashed password
- `role` (Enum: 'user' | 'seller' | 'admin', default: 'user') - User role
- `status` (Enum: 'active' | 'inactive' | 'banned' | 'suspended', default: 'active')
- `phone` (String, optional)
- `country` (String, optional)
- `lastLoginAt` (Date, optional)
- `lifetimeValue` (Number, default: 0) - Total lifetime value
- `metadata` (Map<String, String>, optional)
- `createdAt` (Date, auto)
- `updatedAt` (Date, auto)

**Indexes**:
- `{ email: 1, role: 1 }` (unique, compound) - Same email, different roles
- `{ createdAt: 1 }` - User growth analytics
- `{ role: 1, status: 1 }` - Role-based filtering
- `{ role: 1, createdAt: -1 }` - Role-based user lists

**Analytics Queries**:
- Total users per role
- User growth over time
- Active vs inactive users

---

### 2. products

**Purpose**: Products belong to sellers (userId = seller)

**Fields**:
- `_id` (ObjectId) - Primary key
- `userId` (ObjectId, ref: User, required) - **Seller ID (first-class field)**
- `title` (String, required) - Product title
- `description` (String, required) - Product description
- `price` (Number, required, min: 0) - Product price
- `status` (Enum: 'active' | 'inactive', default: 'active')
- `category` (String, required) - Product category
- `shopName` (String, required) - Shop name
- `sku` (String, unique) - Stock keeping unit
- `images` (Array<String>) - Product images
- `views` (Number, default: 0) - View count
- `metadata` (Map<String, String>, optional)
- `createdAt` (Date, auto)
- `updatedAt` (Date, auto)

**Indexes**:
- `{ userId: 1, status: 1 }` - Seller's active products
- `{ userId: 1, createdAt: -1 }` - Seller's product list
- `{ category: 1, status: 1 }` - Category browsing
- `{ userId: 1, category: 1 }` - Seller's products by category
- `{ createdAt: -1 }` - Recent products
- `{ sku: 1 }` - SKU lookup

**Validation**:
- userId must reference a user with role='seller'

**Analytics Queries**:
- Products per seller
- Products by category
- Active vs inactive products

---

### 3. inventory

**Purpose**: Tracks product inventory per seller

**Fields**:
- `_id` (ObjectId) - Primary key
- `userId` (ObjectId, ref: User, required) - **Seller ID (first-class field)**
- `productId` (ObjectId, ref: Product, required) - Product ID
- `quantityAvailable` (Number, required, min: 0, default: 0)
- `reservedQuantity` (Number, default: 0, min: 0) - Reserved for orders
- `lowStockThreshold` (Number, default: 10, min: 0)
- `metadata` (Map<String, String>, optional)
- `createdAt` (Date, auto)
- `updatedAt` (Date, auto) - **Critical for inventory tracking**

**Indexes**:
- `{ userId: 1, productId: 1 }` (unique, compound) - One record per seller-product
- `{ userId: 1, updatedAt: -1 }` - Seller's inventory updates
- `{ productId: 1 }` - Product availability queries
- `{ quantityAvailable: 1 }` - Low stock alerts

**Validation**:
- userId must reference a user with role='seller'
- productId must belong to the same seller (product.userId === inventory.userId)

**Analytics Queries**:
- Inventory count per seller
- Low stock products per seller
- Inventory updates over time

---

### 4. orders

**Purpose**: Orders belong to buyers (userId = buyer/user)

**Fields**:
- `_id` (ObjectId) - Primary key
- `orderNumber` (String, required, unique) - Order number
- `userId` (ObjectId, ref: User, required) - **Buyer ID (first-class field)**
- `totalAmount` (Number, required, min: 0) - Total order amount
- `orderStatus` (Enum: 'pending' | 'shipped' | 'delivered' | 'cancelled', default: 'pending')
- `currency` (String, default: 'USD')
- `shippingAddress` (Object) - Shipping address details
- `metadata` (Object) - Additional metadata
- `createdAt` (Date, auto) - **Critical for analytics**
- `updatedAt` (Date, auto)

**Indexes**:
- `{ orderNumber: 1 }` - Order lookup
- `{ userId: 1, createdAt: -1 }` - Buyer's order history
- `{ userId: 1, orderStatus: 1 }` - Buyer's orders by status
- `{ orderStatus: 1, createdAt: -1 }` - Status-based queries
- `{ createdAt: -1 }` - Time-based analytics
- `{ userId: 1, orderStatus: 1, createdAt: -1 }` - Composite for buyer analytics

**Validation**:
- userId must reference a user with role='user' (buyer) or 'admin'

**Analytics Queries**:
- Total orders per user
- Orders by status per user
- Order trends over time

---

### 5. order_items

**Purpose**: Separate collection for order items (normalized design)

**Fields**:
- `_id` (ObjectId) - Primary key
- `orderId` (ObjectId, ref: Order, required) - Order reference
- `productId` (ObjectId, ref: Product, required) - Product reference
- `sellerId` (ObjectId, ref: User, required) - **Seller ID (first-class field)**
- `quantity` (Number, required, min: 1)
- `price` (Number, required, min: 0) - Unit price
- `discount` (Number, default: 0, min: 0)
- `lineTotal` (Number, required, min: 0) - Total for this line item
- `productName` (String, required) - Denormalized for performance
- `productSku` (String, required) - Denormalized for performance
- `productCategory` (String, optional) - Denormalized for performance
- `createdAt` (Date, auto)
- `updatedAt` (Date, auto)

**Indexes**:
- `{ orderId: 1, createdAt: -1 }` - Order items list
- `{ sellerId: 1, createdAt: -1 }` - **Seller revenue analytics**
- `{ productId: 1 }` - Product sales analytics
- `{ sellerId: 1, productId: 1 }` - Seller-product sales
- `{ createdAt: -1 }` - Time-based analytics

**Validation**:
- sellerId must reference a user with role='seller'

**Analytics Queries**:
- Products sold per seller
- Revenue per seller (sum of lineTotal)
- Product sales analytics

---

### 6. payments

**Purpose**: Payments belong to buyers (userId = buyer/user)

**Fields**:
- `_id` (ObjectId) - Primary key
- `userId` (ObjectId, ref: User, required) - **Buyer ID (first-class field)**
- `orderId` (ObjectId, ref: Order, required) - Order reference
- `amount` (Number, required, min: 0) - Payment amount
- `paymentMethod` (Enum: 'card' | 'cod' | 'paypal' | 'wallet' | 'bank', required)
- `paymentStatus` (Enum: 'pending' | 'completed' | 'failed' | 'refunded' | 'partial', default: 'pending')
- `transactionId` (String, optional) - External transaction ID
- `paymentDate` (Date, default: now)
- `refundAmount` (Number, default: 0, min: 0)
- `refundDate` (Date, optional)
- `metadata` (Map<String, String>, optional)
- `createdAt` (Date, auto)
- `updatedAt` (Date, auto)

**Indexes**:
- `{ userId: 1, createdAt: -1 }` - Buyer's payment history
- `{ orderId: 1 }` - Order-payment relationship
- `{ paymentStatus: 1, createdAt: -1 }` - Payment status queries
- `{ paymentMethod: 1 }` - Payment method analytics
- `{ userId: 1, paymentStatus: 1 }` - Buyer payment status
- `{ createdAt: -1 }` - Time-based analytics

**Validation**:
- userId must reference a user with role='user' (buyer) or 'admin'
- orderId must belong to the same userId (order.userId === payment.userId)

**Analytics Queries**:
- Total amount spent per user
- Payment method distribution
- Payment success rate

---

### 7. revenue_logs

**Purpose**: Tracks revenue per seller for analytics

**Fields**:
- `_id` (ObjectId) - Primary key
- `userId` (ObjectId, ref: User, required) - **Seller ID (first-class field)**
- `orderId` (ObjectId, ref: Order, required) - Order reference
- `productId` (ObjectId, ref: Product, required) - Product reference
- `revenueAmount` (Number, required, min: 0) - Revenue for this item
- `quantity` (Number, required, min: 1)
- `unitPrice` (Number, required, min: 0)
- `orderNumber` (String, required) - Denormalized for performance
- `productName` (String, required) - Denormalized for performance
- `metadata` (Map<String, String>, optional)
- `createdAt` (Date, auto) - **Critical for time-based revenue analytics**
- `updatedAt` (Date, auto)

**Indexes**:
- `{ userId: 1, createdAt: -1 }` - **Seller revenue over time**
- `{ userId: 1, productId: 1 }` - Seller-product revenue
- `{ orderId: 1 }` - Order revenue breakdown
- `{ productId: 1 }` - Product revenue analytics
- `{ createdAt: -1 }` - Time-based analytics
- `{ userId: 1, createdAt: 1 }` - Seller monthly/yearly revenue

**Validation**:
- userId must reference a user with role='seller'

**Analytics Queries**:
- Total revenue per seller
- Revenue by product per seller
- Revenue trends over time (daily, monthly, yearly)

---

## DATA INTEGRITY & SAFETY

### Foreign Key Relationships

1. **products.userId** → users._id (must be seller)
2. **inventory.userId** → users._id (must be seller)
3. **inventory.productId** → products._id (product must belong to same seller)
4. **orders.userId** → users._id (must be user/buyer or admin)
5. **order_items.orderId** → orders._id
6. **order_items.productId** → products._id
7. **order_items.sellerId** → users._id (must be seller)
8. **payments.userId** → users._id (must be user/buyer or admin)
9. **payments.orderId** → orders._id (order must belong to same userId)
10. **revenue_logs.userId** → users._id (must be seller)
11. **revenue_logs.orderId** → orders._id
12. **revenue_logs.productId** → products._id

### Prevent Orphan Records

- All foreign key relationships are enforced via Mongoose pre-save hooks
- Deletes/updates are handled safely (cascade deletes can be added if needed)

### No Duplicate Data

- Denormalized fields (productName, productSku) are only used for performance
- No derived data stored unless required for analytics performance

---

## ANALYTICS-READY DESIGN

### Supported Analytics Queries

1. **Total orders per user**
   ```javascript
   Order.countDocuments({ userId: userId })
   ```

2. **Total amount spent per user**
   ```javascript
   Order.aggregate([
     { $match: { userId: userId } },
     { $group: { _id: null, total: { $sum: "$totalAmount" } } }
   ])
   ```

3. **Total revenue per seller**
   ```javascript
   RevenueLog.aggregate([
     { $match: { userId: sellerId } },
     { $group: { _id: null, total: { $sum: "$revenueAmount" } } }
   ])
   ```

4. **Products sold per seller**
   ```javascript
   OrderItem.aggregate([
     { $match: { sellerId: sellerId } },
     { $group: { _id: null, total: { $sum: "$quantity" } } }
   ])
   ```

5. **Inventory count per seller**
   ```javascript
   Inventory.aggregate([
     { $match: { userId: sellerId } },
     { $group: { _id: null, total: { $sum: "$quantityAvailable" } } }
   ])
   ```

6. **Order status tracking**
   ```javascript
   Order.aggregate([
     { $match: { userId: userId } },
     { $group: { _id: "$orderStatus", count: { $sum: 1 } } }
   ])
   ```

---

## INDEXING STRATEGY

### Critical Indexes (for performance)

1. **userId indexes** - On all collections (first-class field)
2. **orderId indexes** - For order-related queries
3. **productId indexes** - For product-related queries
4. **createdAt indexes** - For time-based analytics
5. **status/enum indexes** - For filtering queries
6. **Compound indexes** - For complex queries (userId + status + createdAt)

### Index Maintenance

- All indexes are created automatically via Mongoose schema definitions
- Background index creation to avoid blocking operations
- Unique indexes prevent data integrity issues

---

## FUTURE CHATBOT COMPATIBILITY

The schema is designed so that:

1. **All user analytics can be fetched using userId**
   - No complex joins required
   - Direct queries on userId field

2. **Aggregations can be performed without complex joins**
   - Denormalized fields where needed (productName, orderNumber)
   - Separate collections for normalized data (order_items, revenue_logs)

3. **No cross-user data leakage is possible**
   - Every query filters by userId
   - Role-based access enforced at schema level
   - Validation hooks prevent invalid relationships

---

## MIGRATION NOTES

### Role Change: 'buyer' → 'user'

The User model now uses `role: 'user'` instead of `role: 'buyer'` to match the specification.

**Migration Required**:
- Update all controllers checking for `role === 'buyer'` to check for `role === 'user'`
- Update frontend code checking for buyer role
- Consider data migration script if existing data uses 'buyer'

### Field Changes

1. **Product**: `createdBy` → `userId`
2. **Order**: `user` → `userId`, `status` → `orderStatus`
3. **OrderItem**: New separate collection (was embedded in Order)
4. **Payment**: New collection
5. **Inventory**: New collection
6. **RevenueLog**: New collection

---

## SCHEMA SUMMARY

| Collection | userId Field | Purpose | Role |
|------------|--------------|---------|------|
| users | _id | User accounts | user/seller/admin |
| products | userId | Products | seller |
| inventory | userId | Inventory tracking | seller |
| orders | userId | Orders | user (buyer) |
| order_items | sellerId | Order items | seller |
| payments | userId | Payments | user (buyer) |
| revenue_logs | userId | Revenue tracking | seller |

---

## NEXT STEPS

1. Update all controllers to use new field names (userId, orderStatus)
2. Update role checks from 'buyer' to 'user'
3. Create migration scripts for existing data
4. Update frontend to use new field names
5. Test analytics queries with new schema


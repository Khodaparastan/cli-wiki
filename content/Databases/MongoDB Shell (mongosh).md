
## Table of Contents
- [Basic Operations](#basic-operations)
- [Database Operations](#database-operations)
- [Collection Operations](#collection-operations)
- [Query Operations](#query-operations)
- [Aggregation Pipeline](#aggregation-pipeline)
- [Index Management](#index-management)
- [User Management](#user-management)
- [Monitoring and Analysis](#monitoring-and-analysis)

## Basic Operations

### Connection
```javascript
// Basic connection
mongosh "mongodb://localhost:27017"

// Connection with authentication
mongosh "mongodb://user:password@localhost:27017/dbname"

// Connection with options
mongosh "mongodb://localhost:27017/?replicaSet=myRepl&authSource=admin"

// Connect to Atlas
mongosh "mongodb+srv://cluster0.example.mongodb.net/mydb"
```

### Database Management
```javascript
// Show databases
show dbs

// Switch database
use mydb

// Show current database
db

// Show collections
show collections

// Database stats
db.stats()
```

## Database Operations

### CRUD Operations
```javascript
// Create document
db.users.insertOne({
    name: "John",
    age: 30,
    email: "john@example.com"
})

// Create multiple documents
db.users.insertMany([
    { name: "Alice", age: 25 },
    { name: "Bob", age: 35 }
])

// Read documents
db.users.find({ age: { $gt: 25 }})

// Update document
db.users.updateOne(
    { name: "John" },
    { $set: { age: 31 }}
)

// Update multiple documents
db.users.updateMany(
    { age: { $lt: 30 }},
    { $inc: { age: 1 }}
)

// Delete document
db.users.deleteOne({ name: "John" })

// Delete multiple documents
db.users.deleteMany({ age: { $lt: 25 }})
```

## Collection Operations

### Collection Management
```javascript
// Create collection with options
db.createCollection("logs", {
    capped: true,
    size: 5242880,
    max: 5000
})

// Collection stats
db.users.stats()

// Rename collection
db.users.renameCollection("people")

// Drop collection
db.users.drop()

// Create indexes
db.users.createIndex({ email: 1 }, { unique: true })
```

### Bulk Operations
```javascript
// Initialize bulk operations
let bulk = db.users.initializeUnorderedBulkOp()

// Add operations
bulk.insert({ name: "John", age: 30 })
bulk.find({ age: { $lt: 25 }}).update({ $inc: { age: 1 }})
bulk.find({ name: "Alice" }).remove()

// Execute bulk operations
bulk.execute()
```

## Query Operations

### Advanced Queries
```javascript
// Complex query with multiple conditions
db.users.find({
    $and: [
        { age: { $gte: 25, $lte: 50 }},
        { "address.city": "New York" },
        { tags: { $in: ["premium", "active"] }}
    ]
})

// Projection
db.users.find(
    { age: { $gt: 25 }},
    { name: 1, email: 1, _id: 0 }
)

// Sort, limit, and skip
db.users.find()
    .sort({ age: -1 })
    .limit(10)
    .skip(20)

// Distinct values
db.users.distinct("tags")
```

### Text Search
```javascript
// Create text index
db.articles.createIndex({ content: "text" })

// Text search
db.articles.find({
    $text: {
        $search: "mongodb database",
        $caseSensitive: false
    }
})

// Search with score
db.articles.find(
    { $text: { $search: "mongodb" }},
    { score: { $meta: "textScore" }}
).sort({ score: { $meta: "textScore" }})
```

## Aggregation Pipeline

### Basic Aggregation
```javascript
// Simple aggregation
db.orders.aggregate([
    { $match: { status: "completed" }},
    { $group: {
        _id: "$customer",
        total: { $sum: "$amount" },
        count: { $sum: 1 }
    }},
    { $sort: { total: -1 }}
])

// Complex aggregation
db.sales.aggregate([
    { $match: { 
        date: { 
            $gte: ISODate("2024-01-01"),
            $lt: ISODate("2025-01-01")
        }
    }},
    { $unwind: "$items" },
    { $group: {
        _id: {
            month: { $month: "$date" },
            category: "$items.category"
        },
        totalSales: { $sum: { $multiply: ["$items.price", "$items.quantity"] }},
        avgPrice: { $avg: "$items.price" }
    }},
    { $sort: { "_id.month": 1, "totalSales": -1 }},
    { $project: {
        month: "$_id.month",
        category: "$_id.category",
        totalSales: { $round: ["$totalSales", 2] },
        avgPrice: { $round: ["$avgPrice", 2] }
    }}
])
```

## Index Management

### Index Operations
```javascript
// Create single index
db.users.createIndex(
    { email: 1 },
    { unique: true, background: true }
)

// Create compound index
db.orders.createIndex(
    { customer_id: 1, date: -1 }
)

// Create geospatial index
db.places.createIndex(
    { location: "2dsphere" }
)

// List indexes
db.users.getIndexes()

// Drop index
db.users.dropIndex("email_1")

// Index statistics
db.users.aggregate([
    { $indexStats: {} }
])
```

## User Management

### User Administration
```javascript
// Create user
db.createUser({
    user: "appUser",
    pwd: "password123",
    roles: [
        { role: "readWrite", db: "myapp" },
        { role: "read", db: "reporting" }
    ]
})

// Grant roles
db.grantRolesToUser("appUser", [
    { role: "readWrite", db: "newdb" }
])

// Update user
db.updateUser("appUser", {
    pwd: "newPassword",
    roles: [
        { role: "readWrite", db: "myapp" }
    ]
})

// Remove user
db.dropUser("appUser")
```

## Monitoring and Analysis

### Performance Analysis
```javascript
// Current operations
db.currentOp()

// Kill operation
db.killOp(opId)

// Collection statistics
db.users.stats()

// Server status
db.serverStatus()

// Profile queries
db.setProfilingLevel(1, { slowms: 100 })
db.system.profile.find().pretty()

// Explain plan
db.users.find({ age: { $gt: 25 }}).explain("executionStats")
```

### Replica Set Operations
```javascript
// Replica set status
rs.status()

// Configuration
rs.conf()

// Step down primary
rs.stepDown()

// Add member
rs.add("mongodb2.example.com:27017")

// Remove member
rs.remove("mongodb3.example.com:27017")
```

Remember:
- Always use indexes for frequent queries
- Monitor query performance
- Regular backup strategy
- Proper error handling
- Security best practices
- Documentation of schemas
- Regular maintenance

For detailed information, consult the MongoDB documentation and mongosh manual.
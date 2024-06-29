---
image:
    path: assets/images/og_image.png
    width: 300
    height: 169
---

# Speedrunning MongoDB

Maybe you're someone who likes keeping basics of MongoDB in your head. Maybe you have someone who asks MongoDB queries for trivia night? Hey, I don't judge! In this blog, we are speedrunning MongoDB. Hopefully, this is what you need!

## Create, Read, Update, Delete

Good Ol' CRUD.

### Inserting Documents in a MongoDB Collection
**Syntax:** `db.collection.insertOne(document)` or `db.collection.insertMany([documents])`

**Example:**
```js
db.users.insertOne({ name: "Alice", age: 25, status: "A" })
```

### Finding Documents in a MongoDB Collection
**Syntax:** `db.collection.find(query)`

**Example:**
```js
db.users.find({ age: { $gt: 20 } })
```

### Finding Documents Using Comparison Operators
**Operators:** `$eq` (equal to), `$gt` (greater than), `$gte` (greater than or equal to), `$lt` (less than), `$lte` (less than or equal to), `$ne` (not equal to), `$in` (in), `$nin` (not in)

**Example:**
```js
db.users.find({ age: { $gte: 25 } })
```

### Querying Array Elements in MongoDB
**Syntax:** Querying specific elements within arrays

**Example:**
```js
db.users.find({ tags: "mongodb" })
```

### Finding Documents Using Logical Operators
**Operators:** `$and`, `$or`, `$not`, `$nor`

**Example:**
```js
db.users.find({ $or: [{ age: { $lt: 20 } }, { age: { $gt: 30 } }] })
```

### Replacing a Document in MongoDB
**Syntax:** `db.collection.replaceOne(filter, replacement)`

**Example:**
```js
db.users.replaceOne({ name: "Alice" }, { name: "Alice", age: 26, status: "B" })
```

### Updating MongoDB Documents Using `updateOne()`
**Syntax:** `db.collection.updateOne(filter, update, options)`

**Example:**
```js
db.users.updateOne({ name: "Alice" }, { $set: { age: 27 } })
```

### Updating MongoDB Documents Using `findAndModify()`
**Syntax:** `db.collection.findAndModify(query)`

**Example:**
```js
db.users.findAndModify({
  query: { name: "Alice" },
  update: { $set: { age: 28 } }
})
```

### Updating MongoDB Documents Using `updateMany()`
**Syntax:** `db.collection.updateMany(filter, update)`

**Example:**
```js
db.users.updateMany({ status: "A" }, { $set: { status: "B" } })
```

### Deleting Documents in MongoDB
**Syntax:** `db.collection.deleteOne(filter)` or `db.collection.deleteMany(filter)`

**Example:**
```js
db.users.deleteOne({ name: "Alice" })
```

### Sorting and Limiting Query Results in MongoDB
**Syntax:** `db.collection.find(query).sort(sort).limit(number)`

**Example:**
```js
db.users.find().sort({ age: -1 }).limit(5)
```

### Returning Specific Data from a Query in MongoDB
**Syntax:** `db.collection.find(query, projection)`

**Example:**
```js
db.users.find({ name: "Alice" }, { name: 1, age: 1, _id: 0 })
```

### Counting Documents in a MongoDB Collection
**Syntax:** `db.collection.countDocuments(query)`

**Example:**
```js
db.users.countDocuments({ age: { $gte: 25 } })
```

## Aggregation

Process data and return computed results.

### Using `$match` and `$group` Stages in a MongoDB Aggregation Pipeline
The `$match` stage filters documents, and the `$group` stage groups documents by some field.

**Example:**
```js
db.orders.aggregate([
  { $match: { status: "A" } },
  { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

### Using `$sort` and `$limit` Stages in a MongoDB Aggregation Pipeline
The `$sort` stage orders documents, and the `$limit` stage limits the number of documents.

**Example:**
```js
db.orders.aggregate([
  { $sort: { amount: -1 } },
  { $limit: 5 }
])
```

### Using `$project`, `$count`, and `$set` Stages in a MongoDB Aggregation Pipeline
The `$project` stage shapes the documents, the `$count` stage counts documents, and the `$set` stage adds new fields or modifies existing fields.

**Example:**
```js
db.orders.aggregate([
  { $project: { cust_id: 1, amount: 1 } },
  { $count: "total_orders" },
  { $set: { status: "processed" } }
])
```

### Using the `$out` Stage in a MongoDB Aggregation Pipeline
The `$out` stage writes the results to a collection.

**Example:**
```js
db.orders.aggregate([
  { $match: { status: "A" } },
  { $out: "processed_orders" }
])
```

## MongoDB Indexes

Indexes improve query performance by storing certain fields in a B-Tree, reducing search time.

### Creating a Single Field Index in MongoDB
**Syntax:** `db.collection.createIndex({ field: 1 })`

**Example:**
```js
db.users.createIndex({ name: 1 })
```

### Creating a Multikey Index in MongoDB
Multikey indexes are used for indexing array fields.

**Example:**
```js
db.users.createIndex({ tags: 1 })
```

### Working with Compound Indexes in MongoDB
Compound indexes index multiple fields.

**Example:**
```js
db.users.createIndex({ name: 1, age: -1 })
```

### Deleting MongoDB Indexes
**Syntax:** `db.collection.dropIndex(index)`

**Example:**
```js
db.users.dropIndex({ name: 1 })
```

### Geospatial Indexing in MongoDB

For legacy coordinate pairs, use a 2D index:

**Syntax:** `db.collection.createIndex({ location: "2d" })`

**Example:**
```js
db.places.createIndex({ location: "2d" })
```

For spherical geometry, use a 2DSphere index:

**Syntax:** `db.collection.createIndex({ location: "2dsphere" })`

**Example:**
```js
db.places.createIndex({ location: "2dsphere" })
```

## Tokenizers

Tokenizers break text into smaller tokens for indexing and searching. They are used in full-text search configurations.

### edgeNGram
Tokenizes text at the edges.
```json
"text": "hello"
// Tokens: ["h", "he", "hel", "hell", "hello"]
```

### nGram
Tokenizes text into n-grams.
```json
"text": "hello"
// Tokens: ["h", "he", "hel", "ell", "llo", "o"]
```

### regexCaptureGroup
Tokenizes text based on regex capture groups.
```json
"text": "abc123"
// Regex: (\w+)
// Tokens: ["abc", "123"]
```

### standard
The default tokenizer, splits on whitespace and punctuation.
```json
"text": "Hello, world!"
// Tokens: ["Hello", "world"]
```

### uaxUrlEmail
Tokenizes URLs and email addresses.
```json
"text": "email@example.com"
// Tokens: ["email@example.com"]
```

### whitespace
Splits text only on whitespace.
```json
"text": "hello world"
// Tokens: ["hello", "world"]
```

That's it for speedrunning MongoDB! Thankyou for reading my blog.

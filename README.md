# MongoDB Aggregation Pipelines

This repository contains code examples and notes based on the YouTube playlist [MongoDB Aggregation Pipelines](https://youtube.com/playlist?list=PLRAV69dS1uWQ6CZCehxKy0rjkqhQ2Z88t).

## Table of Contents
1. [Introduction to Aggregation](#1-introduction-to-aggregation)
2. [Pipeline Stages](#2-pipeline-stages)
3. [Match and Project](#3-match-and-project)
4. [Group and Unwind](#4-group-and-unwind)
5. [Lookup and Facet](#5-lookup-and-facet)
6. [Bucket and Graph Lookup](#6-bucket-and-graph-lookup)
7. [Performance Tuning](#7-performance-tuning)
8. [Real-world Examples](#8-real-world-examples)
9. [Resources](#resources)


## Detailed Explanation

### 1. Introduction to Aggregation

Aggregation in MongoDB is a powerful tool for data processing and analysis, allowing transformation and combination of data to gain insights.

#### Example:
Input:
```json
[
  { "status": "A", "customerId": 1, "amount": 100 },
  { "status": "A", "customerId": 2, "amount": 200 }
]
```

Pipeline:
```javascript
db.sales.aggregate([
  { $match: { status: "A" } },
  { $group: { _id: "$customerId", totalAmount: { $sum: "$amount" } } }
]);
```

Output:
```json
[
  { "_id": 1, "totalAmount": 100 },
  { "_id": 2, "totalAmount": 200 }
]
```

### 2. Pipeline Stages

Aggregation pipelines consist of multiple stages, each transforming documents in a series of steps.

- **$match**: Filters documents.
- **$project**: Reshapes documents.
- **$group**: Groups documents by a specified key.
- **$sort**: Orders documents.

#### Example:
Input:
```json
[
  { "status": "shipped", "orderDate": "2024-01-01" },
  { "status": "pending", "orderDate": "2024-02-01" }
]
```

Pipeline:
```javascript
db.orders.aggregate([
  { $match: { status: "shipped" } },
  { $sort: { orderDate: -1 } }
]);
```

Output:
```json
[
  { "status": "shipped", "orderDate": "2024-01-01" }
]
```

### 3. Match and Project

- **$match**: Filters documents based on specific conditions.
- **$project**: Specifies which fields to include or exclude.

#### Example:
Input:
```json
[
  { "name": "Alice", "age": 30, "address": "123 Main St" },
  { "name": "Bob", "age": 20, "address": "456 Maple Ave" }
]
```

Pipeline:
```javascript
db.customers.aggregate([
  { $match: { age: { $gt: 25 } } },
  { $project: { name: 1, age: 1, address: 1 } }
]);
```

Output:
```json
[
  { "name": "Alice", "age": 30, "address": "123 Main St" }
]
```

### 4. Group and Unwind

- **$group**: Aggregates documents into groups, allowing calculations like sum, average, count, etc.
- **$unwind**: Deconstructs an array field from the input documents to output a document for each element.

#### Example:
Input:
```json
[
  { "productId": 1, "quantity": 10, "items": ["item1", "item2"] },
  { "productId": 2, "quantity": 5, "items": ["item3"] }
]
```

Pipeline:
```javascript
db.orders.aggregate([
  { $group: { _id: "$productId", totalSold: { $sum: "$quantity" } } },
  { $unwind: "$items" }
]);
```

Output:
```json
[
  { "_id": 1, "totalSold": 10 },
  { "_id": 2, "totalSold": 5 }
]
```

### 5. Lookup and Facet

- **$lookup**: Performs a left outer join to another collection.
- **$facet**: Processes multiple aggregation pipelines within a single stage.

#### Example:
Input:
```json
[
  { "orderId": 1, "productId": 1 },
  { "orderId": 2, "productId": 2 }
]
```

```json
[
  { "_id": 1, "productName": "Product A" },
  { "_id": 2, "productName": "Product B" }
]
```

Pipeline:
```javascript
db.orders.aggregate([
  { $lookup: { from: "products", localField: "productId", foreignField: "_id", as: "productDetails" } },
  { $facet: {
      "categorizedByPrice": [
        { $match: { price: { $gt: 100 } } },
        { $bucket: { groupBy: "$price", boundaries: [0, 200, 400] } }
      ],
      "categorizedByQuantity": [
        { $match: { quantity: { $gt: 50 } } },
        { $bucket: { groupBy: "$quantity", boundaries: [0, 50, 100] } }
      ]
    }
  }
]);
```

Output:
```json
[
  {
    "categorizedByPrice": [
      { "_id": 200, "count": 1 },
      { "_id": 400, "count": 0 }
    ],
    "categorizedByQuantity": [
      { "_id": 50, "count": 0 },
      { "_id": 100, "count": 1 }
    ]
  }
]
```

### 6. Bucket and Graph Lookup

- **$bucket**: Groups documents into buckets based on specified boundaries.
- **$graphLookup**: Performs a recursive search on a collection.

#### Example:
Input:
```json
[
  { "price": 150 },
  { "price": 75 }
]
```

```json
[
  { "name": "Alice", "friends": ["Bob"] },
  { "name": "Bob", "friends": ["Charlie"] }
]
```

Pipeline:
```javascript
db.sales.aggregate([
  { $bucket: { groupBy: "$price", boundaries: [0, 50, 100, 200], default: "Other" } }
]);

db.contacts.aggregate([
  { $graphLookup: { from: "contacts", startWith: "$friends", connectFromField: "friends", connectToField: "name", as: "socialNetwork" } }
]);
```

Output:
```json
[
  { "_id": 100, "count": 1 },
  { "_id": 200, "count": 1 }
]

[
  {
    "name": "Alice",
    "friends": ["Bob"],
    "socialNetwork": [
      { "name": "Bob", "friends": ["Charlie"] },
      { "name": "Charlie", "friends": [] }
    ]
  }
]
```

### 7. Performance Tuning

Optimizing aggregation pipelines involves strategies like indexing, efficient use of stages, and memory management to enhance performance.

#### Tips:
- **Indexing**: Index fields used in `$match`, `$sort`, and `$lookup` stages to speed up operations.
- **Pipeline Efficiency**: Use `$match` and `$project` stages early in the pipeline to reduce the number of documents processed, minimizing resource usage.

### 8. Real-world Examples

This section applies aggregation in real-world scenarios, such as generating reports or analyzing user behavior.

#### Example:
Input:
```json
[
  { "date": "2023-03-15", "productId": 1, "amount": 100 },
  { "date": "2023-07-20", "productId": 2, "amount": 150 }
]
```

Pipeline:
```javascript
db.sales.aggregate([
  { $match: { date: { $gte: new ISODate("2023-01-01"), $lt: new ISODate("2024-01-01") } } },
  { $group: { _id: { month: { $month: "$date" }, product: "$productId" }, totalSales: { $sum: "$amount" } } },
  { $sort: { "_id.month": 1, totalSales: -1 } }
]);
```

Output:
```json
[
  { "_id": { "month": 3, "product": 1 }, "totalSales": 100 },
  { "_id": { "month": 7, "product": 2 }, "totalSales": 150 }
]
```

## Resources
- [MongoDB Documentation](https://docs.mongodb.com/manual/aggregation/)
- [YouTube Playlist](https://youtube.com/playlist?list=PLRAV69dS1uWQ6CZCehxKy0rjkqhQ2Z88t)
```
This detailed README with examples and outputs helps illustrate each MongoDB aggregation concept effectively, providing a comprehensive guide to using aggregation pipelines.

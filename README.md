### MongoDB Aggregation Pipelines Explained

MongoDB aggregation framework is a powerful tool for performing data processing and analysis directly within the database. Aggregation operations process data records and return computed results. This is especially useful for transforming data and computing aggregated results from multiple documents in a collection. Let's break down the concepts of aggregation pipelines using examples from the provided FakeJSON-Data.

### Resources

- [FakeJSON-Data](https://gist.github.com/theshubhamkumr/9d27d004c3bba89b4152eb57f99361bd)
- [YouTube Playlist](https://youtube.com/playlist?list=PLRAV69dS1uWQ6CZCehxKy0rjkqhQ2Z88t)
- [MongoDB Documentation](https://docs.mongodb.com/manual/aggregation/)

### Table of Contents

1. [Introduction to Aggregation Pipelines](#introduction-to-aggregation-pipelines)
2. [Aggregation Pipeline Stages](#aggregation-pipeline-stages)
3. [Example Aggregation Pipeline](#example-aggregation-pipeline)
   1. [$match Stage](#1-match-stage)
   2. [$group Stage](#2-group-stage)
   3. [$project Stage](#3-project-stage)
   4. [$sort Stage](#4-sort-stage)
   5. [$limit Stage](#5-limit-stage)
   6. [$skip Stage](#6-skip-stage)
4. [Additional Aggregation Pipeline Stages](#additional-aggregation-pipeline-stages)
   1. [$unwind Stage](#7-unwind-stage)
   2. [$lookup Stage](#8-lookup-stage)
   3. [$addFields Stage](#9-addfields-stage)
   4. [$out Stage](#10-out-stage)
   5. [$merge Stage](#11-merge-stage)
5. [Contributions](#contributions)

### Aggregation Pipeline Stages

An aggregation pipeline consists of stages. Each stage transforms the documents as they pass through the pipeline. The stages can filter, sort, group, reshape, and modify documents. Here are some of the primary stages used in aggregation pipelines:

1. **$match**: Filters documents to pass only those that match the specified condition(s).
2. **$group**: Groups input documents by a specified identifier expression and applies the accumulator expressions to each group.
3. **$project**: Reshapes each document in the stream, such as by including, excluding, or adding fields.
4. **$sort**: Sorts all input documents and returns them in the specified order.
5. **$limit**: Restricts the number of documents passed to the next stage in the pipeline.
6. **$skip**: Skips the first N documents, and passes the remaining documents to the next stage in the pipeline.
7. **$unwind**: Deconstructs an array field from the input documents to output a document for each element.
8. **$lookup**: Performs a left outer join to another collection in the same database to filter in documents from the “joined” collection for processing.
9. **$addFields**: Adds new fields to documents. These fields can be computed or static values.
10. **$out**: Writes the resulting documents of the aggregation pipeline to a specified collection.
11. **$merge**: Allows the results of the aggregation pipeline to be output to a specified collection, either updating existing documents or inserting new ones.

### Example Aggregation Pipeline

Let's consider a few examples using the FakeJSON-Data to demonstrate these stages.

#### Sample Data

Here is a subset of the FakeJSON-Data we will use:

```json
[
  {
    "name": "John Doe",
    "age": 29,
    "city": "New York",
    "salary": 70000
  },
  {
    "name": "Jane Smith",
    "age": 32,
    "city": "Los Angeles",
    "salary": 80000
  },
  {
    "name": "Jim Brown",
    "age": 45,
    "city": "Chicago",
    "salary": 120000
  },
  {
    "name": "Jake Blues",
    "age": 35,
    "city": "New York",
    "salary": 95000
  }
]
```

#### 1. $match Stage

The `$match` stage filters the documents.

```json
db.employees.aggregate([
  { $match: { city: "New York" } }
])
```

**Explanation**: This pipeline filters documents to only those where the city is "New York".

**Output**:

```json
[
  { "name": "John Doe", "age": 29, "city": "New York", "salary": 70000 },
  { "name": "Jake Blues", "age": 35, "city": "New York", "salary": 95000 }
]
```

#### 2. $group Stage

The `$group` stage groups documents by a specified field and can perform operations on each group.

```json
db.employees.aggregate([
  { $group: { _id: "$city", averageSalary: { $avg: "$salary" } } }
])
```

**Explanation**: This pipeline groups documents by the city and calculates the average salary for each city.

**Output**:

```json
[
  { "_id": "New York", "averageSalary": 82500 },
  { "_id": "Los Angeles", "averageSalary": 80000 },
  { "_id": "Chicago", "averageSalary": 120000 }
]
```

#### 3. $project Stage

The `$project` stage reshapes documents by including, excluding, or adding new fields.

```json
db.employees.aggregate([
  { $project: { name: 1, city: 1, salary: 1, _id: 0 } }
])
```

**Explanation**: This pipeline projects documents to include only the `name`, `city`, and `salary` fields, excluding the `_id` field.

**Output**:

```json
[
  { "name": "John Doe", "city": "New York", "salary": 70000 },
  { "name": "Jane Smith", "city": "Los Angeles", "salary": 80000 },
  { "name": "Jim Brown", "city": "Chicago", "salary": 120000 },
  { "name": "Jake Blues", "city": "New York", "salary": 95000 }
]
```

#### 4. $sort Stage

The `$sort` stage orders documents by specified fields.

```json
db.employees.aggregate([
  { $sort: { age: 1 } }
])
```

**Explanation**: This pipeline sorts documents by age in ascending order.

**Output**:

```json
[
  { "name": "John Doe", "age": 29, "city": "New York", "salary": 70000 },
  { "name": "Jane Smith", "age": 32, "city": "Los Angeles", "salary": 80000 },
  { "name": "Jake Blues", "age": 35, "city": "New York", "salary": 95000 },
  { "name": "Jim Brown", "age": 45, "city": "Chicago", "salary": 120000 }
]
```

#### 5. $limit Stage

The `$limit` stage restricts the number of documents to pass to the next stage.

```json
db.employees.aggregate([
  { $limit: 2 }
])
```

**Explanation**: This pipeline limits the output to the first 2 documents.

**Output**:

```json
[
  { "name": "John Doe", "age": 29, "city": "New York", "salary": 70000 },
  { "name": "Jane Smith", "age": 32, "city": "Los Angeles", "salary": 80000 }
]
```

#### 6. $skip Stage

The `$skip` stage skips the first N documents.

```json
db.employees.aggregate([
  { $skip: 2 }
])
```

**Explanation**: This pipeline skips the first 2 documents and returns the rest.

**Output**:

```json
[
  { "name": "Jim Brown", "age": 45, "city": "Chicago", "salary": 120000 },
  { "name": "Jake Blues", "age": 35, "city": "New York", "salary": 95000 }
]
```

### Combining Multiple Stages

Aggregation pipelines often combine multiple stages to perform complex data processing in a single query. Here is an example combining `$match`, `$group`, and `$sort` stages:

```json
db.employees.aggregate([
  { $match: { city: "New York" } },
  { $group: { _id: "$city", totalSalary: { $sum: "$salary" }, averageAge: { $avg: "$age" } } },
  { $sort: { totalSalary: -1 } }
])
```

**Explanation**:

1. `$match` filters for employees in "New York".
2. `$group` groups these employees by city and calculates the total salary and average age.
3. `$sort` orders the results by total salary in descending order.

**Output**:

```json
[{ "_id": "New York", "totalSalary": 165000, "averageAge": 32 }]
```

Sure, let's delve into more stages of MongoDB aggregation pipelines, exploring their functions and providing practical examples based on the provided FakeJSON-Data. These stages include `$unwind`, `$lookup`, `$addFields`, `$out`, and `$merge`.

### Additional Aggregation Pipeline Stages

#### 7. $unwind Stage

The `$unwind` stage deconstructs an array field from the input documents to output a document for each element.

**Example**: Suppose we have the following modified dataset with an array field `hobbies`:

```json
[
  { "name": "John Doe", "hobbies": ["reading", "swimming"] },
  { "name": "Jane Smith", "hobbies": ["dancing", "cycling", "cooking"] }
]
```

**Pipeline**:

```json
db.employees.aggregate([
  { $unwind: "$hobbies" }
])
```

**Explanation**: This pipeline breaks down the `hobbies` array so that each hobby becomes a separate document.

**Output**:

```json
[
  { "name": "John Doe", "hobbies": "reading" },
  { "name": "John Doe", "hobbies": "swimming" },
  { "name": "Jane Smith", "hobbies": "dancing" },
  { "name": "Jane Smith", "hobbies": "cycling" },
  { "name": "Jane Smith", "hobbies": "cooking" }
]
```

#### 8. $lookup Stage

The `$lookup` stage performs a left outer join to another collection in the same database to filter in documents from the “joined” collection for processing.

**Example**: Suppose we have another collection `departments`:

```json
[
  { "department_id": 1, "department_name": "HR" },
  { "department_id": 2, "department_name": "Engineering" }
]
```

And the `employees` collection has a `department_id` field:

```json
[
  { "name": "John Doe", "department_id": 1 },
  { "name": "Jane Smith", "department_id": 2 }
]
```

**Pipeline**:

```json
db.employees.aggregate([
  {
    $lookup: {
      from: "departments",
      localField: "department_id",
      foreignField: "department_id",
      as: "department_info"
    }
  }
])
```

**Explanation**: This pipeline joins `employees` with `departments` based on `department_id`.

**Output**:

```json
[
  {
    "name": "John Doe",
    "department_id": 1,
    "department_info": [{ "department_id": 1, "department_name": "HR" }]
  },
  {
    "name": "Jane Smith",
    "department_id": 2,
    "department_info": [
      { "department_id": 2, "department_name": "Engineering" }
    ]
  }
]
```

#### 9. $addFields Stage

The `$addFields` stage adds new fields to documents. These fields can be computed or static values.

**Pipeline**:

```json
db.employees.aggregate([
  {
    $addFields: {
      full_name: { $concat: ["$name", " ", "$last_name"] }
    }
  }
])
```

**Explanation**: Assuming each document has a `name` and `last_name` field, this pipeline adds a `full_name` field by concatenating `name` and `last_name`.

**Output**:

```json
[
  { "name": "John", "last_name": "Doe", "full_name": "John Doe" },
  { "name": "Jane", "last_name": "Smith", "full_name": "Jane Smith" }
]
```

#### 10. $out Stage

The `$out` stage writes the resulting documents of the aggregation pipeline to a specified collection.

**Pipeline**:

```json
db.employees.aggregate([
  {
    $match: { city: "New York" }
  },
  {
    $out: "new_york_employees"
  }
])
```

**Explanation**: This pipeline filters employees in "New York" and writes the results to a new collection named `new_york_employees`.

**Output**: The results are written to the `new_york_employees` collection, not directly returned.

#### 11. $merge Stage

The `$merge` stage allows the results of the aggregation pipeline to be output to a specified collection, either updating existing documents or inserting new ones.

**Pipeline**:

```json
db.employees.aggregate([
  {
    $match: { city: "New York" }
  },
  {
    $merge: { into: "city_employees", on: "_id", whenMatched: "merge", whenNotMatched: "insert" }
  }
])
```

**Explanation**: This pipeline filters employees in "New York" and merges the results into the `city_employees` collection. If documents match on `_id`, they are merged; otherwise, they are inserted.

**Output**: The results are merged into the `city_employees` collection.


### Contributions

- **FakeJSON-Data**: Used as the primary dataset for examples.
- **YouTube Playlist**: Provided practical examples and tutorials.
- **MongoDB Documentation**: Served as the reference for explaining concepts and syntax.

Feel free to contribute to this guide by adding more examples, explanations, or corrections. Your contributions are highly appreciated!

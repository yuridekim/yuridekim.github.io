---
title : 'Query Optimization in Databases: Translating, Planning, and Cost Estimation in Databases'
date : 2024-05-09 04:10:00 +0900
categories : [Database Systems, Query Optimization]
tags : [database] #소문자만 가능
pinned : 0
---

Query optimization is a fundamental part of database management that translates high-level SQL queries into efficient execution plans. The goal? Minimize costs — whether that's CPU time, memory usage, or network transfer time in distributed processing environments.

# Translating Queries into Logical Expressions
The optimization process starts with parsing the incoming query, translating it into a logical expression that represents the operations required. This parsed form allows the system to generate several potential plans for executing the query, exploring different paths to achieve the same result.

# Identifying the Optimal Plan
With multiple plans on hand, the next step is to identify the one with the lowest cost. Cost here is a multi-dimensional metric, often including:

- CPU time: How long will the processor spend executing the query?
- Elapsed time: The total time taken from start to end, considering potential waits and dependencies.
- Memory usage: How much memory will be required during execution?
- Network transfer time (for distributed queries): Time spent transferring data across nodes.

## Cost Estimation in Database Queries
The core principle in query optimization is finding the “cheapest” plan. But estimating cost is far from straightforward, as it depends on factors like data distribution, table sizes, and even hardware specifics.

## Compilation and Runtime Execution
Database systems split query processing into two phases:

- Compile time: The optimizer selects the best plan from the candidates.
- Runtime: The chosen execution plan is executed to produce the final results.
Here's a look at how the process typically unfolds:

- Query Parsing: The database parses the query to check for syntax issues.
- Logical Plan Creation: A base logical plan represents the query in a structured way.
- Optimization: The optimizer uses data statistics to estimate costs, choosing the plan with the least cost.
Execution: At runtime, the system executes the plan, producing the output.
- Syntax and Semantic Analysis
After syntax analysis, semantic analysis checks that the query meets requirements like user access permissions and type compatibility between predicates. For instance, if a query involves comparing an age field to 20, semantic analysis ensures the data type and predicate context are compatible.

- Logical Enumeration and Normalization
To avoid excessive backtracking and complexity, logical enumeration moves in one direction only. This step includes normalization — breaking down expressions into standard forms and simplifying them wherever possible.

For example:

An outer join query like:

```sql
Copy code
SELECT A.id, A.name, B.salary
FROM Employees A
LEFT OUTER JOIN Salaries B ON A.id = B.employee_id
WHERE B.salary IS NOT NULL;
```
could be converted to an <b>inner join</b>:

```sql
Copy code
SELECT A.id, A.name, B.salary
FROM Employees A
INNER JOIN Salaries B ON A.id = B.employee_id;
```
This adjustment reduces processing time by removing the need to handle NULL values from the right table in the outer join.

## Heuristic Rewriting: Enhancing Efficiency with Proven Patterns
Certain optimizations, like selection pushdown, improve efficiency by moving selective operations earlier in the process. However, this is not always beneficial — an expensive predicate that doesn’t significantly reduce the dataset may not be worth pushing down.

## Physical Plan Enumeration and Operator Costing
For each operator in the execution plan, databases evaluate several algorithms to determine the best option:

- Selection: This could involve a table scan, unique index scan, or index scan.
- Join: Options include nested index joins or hash joins.
- Aggregation: Hashing and indexing are common methods here.
Selecting between a table scan and index scan often depends on selectivity. For example, if selectivity is high (e.g., 90% of records match), a table scan may outperform an index scan due to sequential access advantages.

## Cardinality and Cost Estimation
Accurate cardinality estimation — predicting the number of rows an operation will yield — is essential for precise cost estimation. Systems like SAP HANA leverage CPU cost models to guide cardinality and cost predictions, ensuring efficient query execution by incorporating both CPU time and memory demands.
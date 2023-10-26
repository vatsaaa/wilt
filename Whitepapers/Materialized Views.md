---
share:
---
### A View...
- is a derived relation defined in terms of stored base relations (generally tables)
- defines a SQL transformation from a set of base tables to a derived table; this transformation is typically recomputed / re-compiled every time the view is referenced to in a query
- when created, does not compute any results nor does it change how data is stored or indexed
- is a saved query on tables of a DB
- is referenced to, in queries, as if it were a table

**Example:**
```sql
CREATE VIEW user_purchase_summary AS SELECT
  u.id as user_id,
  COUNT(*) as total_purchases,
  SUM(purchases.amount) as lifetime_value
FROM users u
JOIN purchases p ON p.user_id = u.id;
```

_Every time a query referencing view/s is executed, it first computes the results of the view, and then computes the rest of the query using those results._

### A Materialized View...
- takes a regular view and materializes it by upfront computing and storing its results in a “virtual” table
- is like a cache, i.e. a copy of the data that can be accessed quickly
- A regular view is “materialized” by storing tuples of the view in the database (TODO: ???)
- can have index structures and hence database access to materialized views can be much faster than recomputing the view

**Example:**
```sql
CREATE MATERIALIZED VIEW user_purchase_summary AS SELECT
  u.id as user_id,
  COUNT(*) as total_purchases,
  SUM(CASE when p.status = 'cancelled' THEN 1 ELSE 0 END) as cancelled_purchases
FROM users u
JOIN purchases p ON p.user_id = u.id;
```

*A regular view is a saved query, and, a materialized view is a saved query along with its results stored as a table.*
#### Implications of materializing a view
1. When referenced in a query, a materialized view is not recomputed as the results are pre-stored and hence querying materialized views tends to be faster
2. Because it’s stored as if it were a table, indexes can be built on the columns of a materialized view
3. Once a view is materialized, it is only accurate until the underlying base relations are modified. The process of updating a materialized view in response to changes in underlying  is called _view maintenance._

A “view” is an anchored perspective on changing inputs, results are constantly changing as the underlying data changes. Materialization just implies that the transformation is done proactively. So, "materialized views" should update automatically. 

However, in practice, some databases need materialized views to be manually refreshed and others have implemented automatic updates, albeit with limitations.

**Note:** MySQL does not support materialized view as of now. Oracle, Snowflake, MongoDB, Redshift, PostgreSQL all others do.

### Materialized views are used...
- when SQL query is known ahead of time and needs to be repeatedly recalculated
- primarily for caching the results of extremely heavy and complex queries that cannot be run frequently as regular views
- as ability to define (using SQL) any complex transformation of data in DB, and let the DB maintain the results in a “virtual” table.
- when low end-to-end latency is required between when data originates to when it is reflected in a query
- when low-latency query response times with high concurrency or high volume of queries is expected
### References
- [Materialized views](https://materialize.com/guides/materialized-views/)
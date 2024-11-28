
Conceptual and Theoretical Questions

1. Why is normalization important, and where does it conflict with cost-effective reporting?

Normalization eliminates redundancy and ensures data integrity by splitting data into related tables. However, highly normalized databases require multiple joins to retrieve data, increasing query time and ETL complexity.

To balance this, denormalization is selectively applied to combine frequently accessed data into fewer tables for faster reporting.


2. What are the common types of relationships in an ER diagram, and how do they impact database design?

One-to-One (1:1): Simple relationships; use them sparingly as they may lead to unused tables.

One-to-Many (1:N): Common; ensure foreign keys are indexed to speed up joins.

Many-to-Many (M:N): Requires a junction table; can complicate ETL but ensures scalability and integrity.


3. What is a surrogate key, and why is it better than a composite key in large systems?

A surrogate key is a system-generated unique identifier (e.g., CustomerID) that simplifies joins and improves performance.

A composite key uses multiple columns, which can slow down queries and increase storage.


4. What role does cardinality play in cost-effective database design?

Cardinality impacts the number of rows in joins.

High cardinality: Large datasets can cause slow queries. Pre-aggregate or partition such tables.

Low cardinality: Useful for grouping but avoid too many low-cardinality indexes, as they can bloat the database.



5. What are the drawbacks of over-denormalization?

Increased storage usage.

Higher risk of data inconsistency.

Complex ETL processes to manage redundant data updates.



---

Implementation and Practical Questions

6. How do you decide which tables to denormalize for reporting?

Analyze query patterns and frequency.

If a report frequently requires data from multiple tables, merge them into a reporting table.

Denormalize dimensions like time, geography, or customer demographics, as they are static or infrequently updated.


7. How do you optimize ETL pipelines when using normalized databases?

Pre-compute aggregates during ETL for repeated queries.

Use incremental loads instead of full table loads.

Employ materialized views or summary tables for commonly queried data.


8. How does a star schema reduce ETL costs compared to a normalized schema?

Star schemas flatten dimensions into single tables, reducing joins.

Fact tables store measures, while dimension tables provide descriptive data, making data retrieval straightforward and fast for analytical tools.


9. What indexing strategies can lower ETL costs and improve reporting performance?

Use clustered indexes for primary keys.

Apply non-clustered indexes to columns frequently used in WHERE, GROUP BY, and ORDER BY clauses.

Avoid indexing low-cardinality columns like boolean fields.


10. How do you manage Slowly Changing Dimensions (SCDs) in a cost-effective manner?

Use Type 2 SCD for historical tracking, but ensure changes are applied only to relevant columns.

Archive older data to reduce table size, making queries faster and ETL costs lower.



---

Scenario-Based Questions

11. What would you do if an ER design led to slow query performance?

Steps:

Identify bottlenecks using query execution plans.

Denormalize heavily joined tables.

Create summary tables for commonly aggregated data.

Partition large tables based on date or region.



12. How do you design for real-time reporting with low ETL costs?

Use a lambda architecture:

Real-time data (speed layer): Stream data using tools like Apache Kafka or Spark.

Batch data (batch layer): Process historical data for complex transformations.


Ensure tables are indexed and denormalized for real-time querying.


13. If a client requests dynamic reporting capabilities, how do you design the ER diagram?

Focus on dimension tables that are easy to filter and aggregate.

Ensure fact tables are wide but optimized for common metrics.

Use tools like Power BI or Tableau that can query star schemas efficiently.



---

Advanced Questions

14. How do you ensure scalability in an ER diagram for high-volume transactional systems?

Partition large tables horizontally (sharding) or vertically.

Use NoSQL databases like Cassandra for high write throughput.

Store frequently accessed summary data in in-memory databases like Redis.


15. What are the trade-offs between star and snowflake schemas in terms of ETL costs?

Star Schema:

Faster queries due to fewer joins.

Higher storage cost due to redundancy.


Snowflake Schema:

Reduced storage due to normalization.

Slower queries requiring multiple joins.



16. How do you handle hierarchical data (e.g., product categories) in an ER diagram?

Use a self-referential relationship (e.g., ParentID referencing ProductID).

For reporting, denormalize the hierarchy into a single column using a delimiter or flatten the hierarchy.



---

Behavioral Questions

17. How would you handle disagreements with stakeholders on denormalization decisions?

Explain the trade-offs: denormalization benefits reporting but increases storage costs and maintenance complexity.

Provide data-backed examples showing performance improvements from denormalization.


18. How do you ensure data quality in a cost-effective ER design?

Enforce constraints (PK, FK, unique, and check).

Use ETL pipelines with validation steps to catch errors early.

Implement data profiling tools to monitor anomalies.


19. How do you ensure the ER diagram aligns with both business and technical requirements?

Conduct joint workshops with stakeholders and technical teams.

Translate business processes into entities and attributes.

Continuously validate and iterate the design during development.


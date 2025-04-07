# Summary

## Query-Specific Changes

| Recommendation                             | Expected Impact   | Notes                                                          |
|--------------------------------------------|-------------------|----------------------------------------------------------------|
| Materialized/Indexed Views                 | Extra Large       | Precomputes heavy joins/aggregations for repeated use          |
| Eliminate Non-SARGable Operations in Joins | Large             | Avoids full table scans by enabling index seeks                |
| Optimize Subqueries with CTEs              | Medium            | Clarifies logic and may allow better query plan optimization     |
| Review Join Types and Their Order          | Medium            | Reduces intermediate row counts by filtering data earlier       |
| Use WITH (NOLOCK)                          | Small to Medium   | Reduces blocking; use cautiously due to potential dirty reads     |
| Minimize On-the-Fly Data Conversions/String Ops | Small         | Reduces per-row CPU overhead by precomputing conversions           |

## General Changes

| Recommendation                                      | Expected Impact   | Notes                                                          |
|-----------------------------------------------------|-------------------|----------------------------------------------------------------|
| Optimize Data Distribution and Partitioning         | Large             | Minimizes data movement; enables partition elimination          |
| Adopt Incremental (Differential) Loading            | Large             | Loads only changes instead of full reloads; reduces ETL cost      |
| Leverage Result Set Caching                         | Large             | Serves repeated queries from cache to cut execution time         |
| Maintain Clustered Columnstore Indexes/Statistics   | Large             | Ensures efficient scans via healthy indexes and accurate stats     |
| Rewrite Expressions for SARGability                 | Medium            | Enables index/segment elimination by avoiding per-row functions     |
| Implement Advanced Monitoring/Diagnostics           | Small             | Essential for tuning; helps detect issues and validate improvements |

---

## Query-Specific Changes

### 1. Materialized/Indexed Views  
- **Why:**  
  Materialized views store precomputed results of complex joins and aggregations, so repeated queries do not re-execute expensive operations.  
- **Before:**  
  ```sql
  CREATE VIEW [gold].[vw_Dim_FICAHeader_Live] AS
  SELECT
      -- Complex SELECT with multiple joins and calculations
  FROM silver.SAP_DFKKKO AS FICAH
      -- Other joins and subqueries...
  ```
- **After:**  
  ```sql
  CREATE MATERIALIZED VIEW [gold].[mv_Dim_FICAHeader_Live]
  WITH (CLUSTERED COLUMNSTORE INDEX) AS
  SELECT
      -- Same complex SELECT with multiple joins and calculations
  FROM silver.SAP_DFKKKO AS FICAH
      -- Other joins and subqueries...
  ```
- **Explanation:**  
  By storing the result set, queries reading this view will execute much faster. This is especially useful when real-time data isn’t critical.  
- **T-Shirt Impact:** **Extra Large**

---

### 2. Eliminate Non-SARGable Operations in Joins  
- **Why:**  
  A non-SARGable operation prevents the optimizer from using an index. “SARGable” (Search ARGument-able) means that a condition can efficiently use an index. Using a function like `RIGHT(FICAH.XBLNR,12)` forces a full table scan.
- **Before:**  
  ```sql
  LEFT OUTER JOIN silver.SAP_ERDK AS ERDK 
      ON ERDK.MANDT = FICAH.MANDT 
     AND ERDK.OPBEL = RIGHT(FICAH.XBLNR, 12)
  ```
- **After:**  
  First, create a computed, persisted column and index it:
  ```sql
  ALTER TABLE silver.SAP_DFKKKO
  ADD XBLNR_Right12 AS RIGHT(XBLNR, 12) PERSISTED;
  
  CREATE INDEX IX_SAP_DFKKKO_XBLNR_Right12 
      ON silver.SAP_DFKKKO(XBLNR_Right12);
  ```
  Then update the join:
  ```sql
  LEFT OUTER JOIN silver.SAP_ERDK AS ERDK 
      ON ERDK.MANDT = FICAH.MANDT 
     AND ERDK.OPBEL = FICAH.XBLNR_Right12
  ```
- **Explanation:**  
  Precomputing the value makes the join SARGable, allowing the optimizer to use the index and significantly reducing scanning time.
- **T-Shirt Impact:** **Large**

---

### 3. Optimize Subqueries and Aggregations Using CTEs  
- **Why:**  
  Complex subqueries with DISTINCT, GROUP BY, and HAVING clauses can be expensive if written inline. CTEs (Common Table Expressions) can simplify and potentially optimize the processing of these subqueries.
- **Before (FICAOP subquery inline):**
  ```sql
  LEFT OUTER JOIN (
      SELECT DISTINCT OPBEL, MANDT,
             MAX(CASE WHEN AUGRS = '8' THEN '8' ELSE NULL END) AS Restriction8Flag,
             MAX(CASE WHEN AUGRS = 'N' THEN 'N' ELSE NULL END) AS RestrictionNFlag
      FROM Silver.SAP_DFKKOP
      GROUP BY OPBEL, MANDT
      HAVING MAX(CASE WHEN AUGRS = '8' THEN '8' END) IS NOT NULL
          OR MAX(CASE WHEN AUGRS = 'N' THEN 'N' ELSE NULL END) IS NOT NULL
  ) AS FICAOP 
    ON FICAH.MANDT = FICAOP.MANDT AND FICAH.OPBEL = FICAOP.OPBEL
  ```
- **After (Using a CTE):**
  ```sql
  ;WITH cte_FICAOP AS (
      SELECT OPBEL, MANDT,
             MAX(CASE WHEN AUGRS = '8' THEN '8' ELSE NULL END) AS Restriction8Flag,
             MAX(CASE WHEN AUGRS = 'N' THEN 'N' ELSE NULL END) AS RestrictionNFlag
      FROM Silver.SAP_DFKKOP
      GROUP BY OPBEL, MANDT
      HAVING MAX(CASE WHEN AUGRS = '8' THEN '8' END) IS NOT NULL
          OR MAX(CASE WHEN AUGRS = 'N' THEN 'N' END) IS NOT NULL
  )
  ...
  LEFT OUTER JOIN cte_FICAOP AS FICAOP 
    ON FICAH.MANDT = FICAOP.MANDT AND FICAH.OPBEL = FICAOP.OPBEL
  ```
- **Explanation:**  
  Using a CTE can isolate heavy subqueries, making them easier to manage and possibly allowing the optimizer to reuse the computed result.
- **T-Shirt Impact:** **Medium**

---

### 4. Review Join Types and Their Order  
- **Why:**  
  Using the most appropriate join type (e.g., converting a LEFT JOIN to an INNER JOIN when possible) can help filter data earlier and reduce intermediate result sizes.
- **Before:**
  ```sql
  LEFT OUTER JOIN silver.SAP_TFK003T AS DTYP 
      ON DTYP.MANDT = FICAH.MANDT 
     AND DTYP.SPRAS = 'E' 
     AND DTYP.APPLK = FICAH.APPLK 
     AND DTYP.BLART = FICAH.BLART
  ```
- **After (if business rules allow an INNER JOIN):**
  ```sql
  INNER JOIN silver.SAP_TFK003T AS DTYP 
      ON DTYP.MANDT = FICAH.MANDT 
     AND DTYP.SPRAS = 'E' 
     AND DTYP.APPLK = FICAH.APPLK 
     AND DTYP.BLART = FICAH.BLART
  ```
- **Explanation:**  
  Inner joins let the optimizer reorder operations, potentially reducing the number of rows processed in subsequent joins.
- **T-Shirt Impact:** **Medium**

---

### 5. Use WITH (NOLOCK) Hints Where Appropriate  
- **Why:**  
  Adding the `WITH (NOLOCK)` hint allows the query to read uncommitted data without waiting for shared locks, reducing blocking. However, it can lead to “dirty reads.”
- **Before:**
  ```sql
  FROM silver.SAP_DFKKKO AS FICAH
  LEFT OUTER JOIN silver.SAP_TFK003T AS DTYP 
      ON DTYP.MANDT = FICAH.MANDT 
     AND DTYP.SPRAS = 'E' 
     AND DTYP.APPLK = FICAH.APPLK 
     AND DTYP.BLART = FICAH.BLART
  ```
- **After:**
  ```sql
  FROM silver.SAP_DFKKKO AS FICAH WITH (NOLOCK)
  LEFT OUTER JOIN silver.SAP_TFK003T AS DTYP WITH (NOLOCK)
      ON DTYP.MANDT = FICAH.MANDT 
     AND DTYP.SPRAS = 'E' 
     AND DTYP.APPLK = FICAH.APPLK 
     AND DTYP.BLART = FICAH.BLART
  ```
- **Explanation:**  
  Use this hint only when data consistency is not critical during high-concurrency loads.
- **T-Shirt Impact:** **Small to Medium**

---

### 6. Minimize On-the-Fly Data Conversions and String Operations  
- **Why:**  
  Runtime conversions (like `CONVERT(date, FICAH.BUDAT,112)`) add CPU overhead on every row processed. Precomputing these results improves performance.
- **Before:**
  ```sql
  CONVERT(date, FICAH.BUDAT, 112) AS "PostingDate",
  ```
- **After:**  
  Create a computed, persisted column:
  ```sql
  ALTER TABLE silver.SAP_DFKKKO
  ADD Budat_Date AS CONVERT(date, BUDAT, 112) PERSISTED;
  ```
  Then update the query:
  ```sql
  Budat_Date AS "PostingDate",
  ```
- **Explanation:**  
  Precomputing conversions avoids per-row calculations, saving CPU cycles.
- **T-Shirt Impact:** **Small**

---

## General Changes
(for this query and others like it)

### 1. Optimize Data Distribution and Partitioning  
- **Why:**  
  Proper data distribution minimizes data movement (the costly reshuffling of data across nodes) and partitioning allows the query engine to read only the necessary data segments (partition elimination).  
- **Action:**  
  - **Distribution:** Hash-distribute large tables on frequently joined keys; use REPLICATE for small dimensions.  
  - **Partitioning:** Partition large fact tables (e.g., by date) so that queries filtering on the partition key scan only relevant partitions.
- **Terms Explained:**  
  - **Hash Distribution:** Rows are spread across nodes based on a hash of a column.  
  - **Replicate Distribution:** Small tables are copied to every node.  
  - **Partition Elimination:** The query engine skips partitions that do not meet the filter criteria.
- **T-Shirt Impact:** **Large**

---

### 2. Adopt Incremental (Differential) Loading  
- **Why:**  
  Instead of truncating and reloading entire tables, loading only new or changed data reduces ETL processing time and resource usage, while preserving indexes and statistics.  
- **Action:**  
  Implement Change Data Capture (CDC) or use a high-water mark to detect changes and apply only the differential updates.
- **Terms Explained:**  
  - **Incremental Load:** Only new or modified data is loaded instead of a full reload.
- **T-Shirt Impact:** **Large**

---

### 3. Leverage Result Set Caching  
- **Why:**  
  When the same query is executed repeatedly against static or slowly changing data, caching the result set prevents repeated execution of heavy queries.  
- **Action:**  
  Enable result set caching at the Synapse database level.
- **Terms Explained:**  
  - **Result Set Caching:** Stores the output of a query so that subsequent identical queries return cached results instantly.
- **T-Shirt Impact:** **Large**

---

### 4. Maintain Clustered Columnstore Indexes and Statistics  
- **Why:**  
  Healthy indexes and up-to-date statistics help the query optimizer make accurate decisions, ensuring efficient execution plans.  
- **Action:**  
  Regularly rebuild or reorganize columnstore indexes and update statistics after major data loads.
- **Terms Explained:**  
  - **Clustered Columnstore Index (CCI):** A storage format optimized for large-scale analytical queries that compresses data.  
  - **Statistics:** Data distribution metadata used by the optimizer.
- **T-Shirt Impact:** **Large**

---

### 5. Rewrite Expressions for SARGability  
- **Why:**  
  Ensuring that predicates are SARGable (i.e., can use indexes) by avoiding functions on columns in WHERE or JOIN conditions enables the query to perform efficient searches.
- **Action:**  
  Rewrite expressions so that filtering conditions compare raw columns rather than computed values. For example, instead of `CONVERT(date, Column)` in the WHERE clause, use a range filter.
- **Terms Explained:**  
  - **SARGable:** A predicate that allows the optimizer to use an index to quickly locate rows.
- **T-Shirt Impact:** **Medium**

---

### 6. Implement Advanced Monitoring and Diagnostics  
- **Why:**  
  Continuous monitoring with Synapse DMVs and execution plan analysis helps detect bottlenecks (e.g., data shuffles or spills) and validates that optimizations are effective.
- **Action:**  
  Use DMVs such as `sys.dm_pdw_exec_requests` and `sys.dm_pdw_request_steps` to track query performance and data movement.
- **Terms Explained:**  
  - **DMVs (Dynamic Management Views):** System views providing runtime statistics about the database engine.
- **T-Shirt Impact:** **Small** (but essential as a best practice)

---

## Conclusion

For **query-specific changes**, focus first on materialized views (Extra Large impact) and eliminating non-SARGable operations (Large impact) to reduce full table scans. Then use CTEs, review join types, add NOLOCK hints where safe, and precompute data conversions to further streamline the query.

For **general changes**, optimize your data distribution/partitioning, adopt incremental loads, and enable result caching for broad performance gains. Maintain healthy columnstore indexes and update statistics so the optimizer has accurate information. Finally, rewrite expressions for SARGability and set up advanced monitoring to keep performance on track.
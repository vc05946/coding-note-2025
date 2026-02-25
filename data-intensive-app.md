This note covers the followings:

- Ch 2 (Requirements)
- Ch 4 (Storage Engines - especially B-Trees vs LSM)
- Ch 6 (Replication)
- Ch 7 (Sharding)
- Ch 12 (Stream Processing/Kafka)

## Chapter 4: Storage and Retrieval 

### Part 1: The Log-Structured Approach (LSM-Trees)
*Used by: Cassandra, RocksDB, LevelDB, ScyllaDB, Google Bigtable.*

The chapter starts with a shell script that appends data to a file. This is the simplest database. It introduces the concept of a **Log**—an append-only sequence of records.

**Key Concepts to Master:**
1.  **Hash Indexes:** Keeping a map in memory (`key -> byte offset on disk`). Fast, but requires all keys to fit in RAM.
2.  **SSTables (Sorted String Tables):** The breakthrough moment. Instead of writing data in random order, we sort it by key.
    *   *Why sort it?* It allows you to use a sparse index (you don't need every key in RAM) and enables efficient range scans.
3.  **LSM-Trees (Log-Structured Merge-Trees):** This is the engine that manages SSTables.
    *   **Memtable:** Writes go to RAM first (Red-Black tree or similar).
    *   **Flushing:** When RAM is full, write it to disk as an immutable SSTable.
    *   **Compaction:** Background processes merge old files and throw away deleted data (tombstones).

**⚠️ Mental Check:** Do you understand why LSM-trees are generally **faster for writes**? (Hint: They turn random writes into sequential writes).

---

### Part 2: The Update-in-Place Approach (B-Trees)
*Used by: PostgreSQL, MySQL, Oracle, SQL Server.*

This is the industry standard. Unlike LSMs, which write new files, B-Trees modify files in place.

**Key Concepts to Master:**
1.  **Pages (Blocks):** The database breaks the disk into fixed-size blocks (usually 4KB or 8KB).
2.  **The Tree Structure:** A hierarchy of pointers. You start at the root and traverse down to a leaf page to find your data.
3.  **The WAL (Write-Ahead Log):** B-Trees have a vulnerability—if the power cuts while overwriting a page, the database is corrupted. The WAL is the fail-safe appended to disk *before* the tree is touched.

**⚠️ Mental Check:** Do you understand why B-Trees are generally **faster for reads**? (Hint: You don't have to check multiple different files/levels to find a key; you just trace a single path down the tree).

---

### Part 3: The Showdown (LSM vs. B-Tree)
*This section (around page 129 in the new edition) is the heart of the chapter.*

Here is the cheat sheet you should keep in mind while reading this section:

| Feature | **LSM-Trees** (e.g., Cassandra) | **B-Trees** (e.g., Postgres) |
| :--- | :--- | :--- |
| **Write Performance** | **High.** Writes are sequential (append-only). | **Lower.** Must write to WAL + overwrite page. Write Amplification is higher. |
| **Read Performance** | **Lower.** Must check Memtable + multiple SSTables. | **High.** Predictable lookup time. |
| **Space Efficiency** | **High.** Compaction compresses data well. | **Lower.** Fragmentation leaves unused space in pages. |
| **Consistency** | **Harder.** Requires complex locks or atomicity checks. | **Better.** Easy to lock a specific page/row. |

**New in 2nd Edition:** Look for the discussion on **Write Amplification** on SSDs. This is a crucial concept for modern hardware.

---

### Part 4: Other Indexing Structures
The chapter briefly covers what happens when you need to search by something *other* than the primary key.

1.  **Secondary Indexes:** Pointers to the primary key.
2.  **Multi-column Indexes:** Concatenated indexes (Lastname, Firstname).
3.  **Vector Embeddings (New in 2nd Edition):** This is vital for AI applications. The book explains how **HNSW** (Hierarchical Navigable Small World) indexes work for nearest-neighbor search. *Pay attention to this section if you work with LLMs.*

---

### Part 5: Analytical Storage (OLAP)
*Used by: Snowflake, Redshift, BigQuery, ClickHouse.*

The chapter shifts gears here. The previous trees are for **OLTP** (getting specific rows). This section is about **OLAP** (scanning billions of rows).

## OLTP vs OLAP – The Core Difference

| Aspect                | OLTP (Transactional)                          | OLAP (Analytical)                                   |
|-----------------------|-----------------------------------------------|-----------------------------------------------------|
| Purpose               | Day-to-day operations (CRUD)                  | Analytics, reporting, BI, dashboards                |
| Typical query         | `SELECT * FROM orders WHERE id = 12345`       | `SELECT country, AVG(amount) FROM sales GROUP BY country, month` |
| Data scanned per query| Few – hundreds of rows                        | Millions – billions of rows                         |
| Storage orientation   | **Row-oriented**                              | **Column-oriented**                                 |
| Compression           | Moderate                                      | Very high (5–30× better)                            |
| Write pattern         | Many small, frequent updates/inserts          | Bulk loads, append-only or infrequent big updates   |
| Classic databases     | PostgreSQL, MySQL, Oracle, SQL Server (app DB)| Snowflake, BigQuery, Redshift, ClickHouse, Databricks |

**Key reason columnar wins for analytics**:  
When you run `SUM(sales)` filtered on 3 columns, the engine reads **only those 3 columns** from disk → massive speed & I/O savings.

## Modern Cloud OLAP / Analytical Systems (2026 snapshot)

| System              | Columnar?              | Compute ↔ Storage          | Architecture style                  | Standout Features                                  | Pricing model (main driver)          | Open Source?   | Best For                                          |
|---------------------|------------------------|----------------------------|-------------------------------------|----------------------------------------------------|--------------------------------------|----------------|---------------------------------------------------|
| **Snowflake**       | Yes                    | **Full separation**        | Virtual Warehouses + S3-like storage| Auto-scaling, Time Travel, Zero-copy cloning, Snowpark | Credits (compute seconds)           | No             | Ease of use, multi-cloud, governed sharing        |
| **Amazon Redshift** | Yes                    | Partial (Serverless = yes) | MPP clusters / Serverless           | Deep AWS integration, sort & dist keys tuning      | Node-hours or Serverless RPU        | No             | Heavy AWS shops, mature ecosystem                 |
| **Google BigQuery** | Yes                    | **Full separation**        | Serverless + Dremel engine          | Pay-per-byte-scanned, near-zero management         | Bytes scanned + slots (on-demand/flat-rate) | No     | Ad-hoc exploration, Google Cloud users            |
| **ClickHouse**      | Yes (very aggressive)  | Yes (in Cloud version)     | Distributed MergeTree + vectorized  | Blazing query speed, real-time ingestion           | Cloud = compute / OSS = free         | **Yes** (core) | Highest raw performance, observability, events    |

## One-liner Positioning (2026)

- **Snowflake**  
  → “The friendly, almost magical data warehouse — scales itself, clones instantly, works everywhere.”

- **Redshift**  
  → “The battle-tested AWS columnar MPP warehouse — powerful when you tune it right.”

- **BigQuery**  
  → “Google’s serverless analytics beast — just write SQL and pay mostly for what you scan.”

- **ClickHouse**  
  → “The performance monster — sub-second queries on trillions of rows, open-source roots, now excellent managed too.”

**Key Concepts:**
1.  **Column-Oriented Storage:** Instead of storing `[Name, Age, Salary]`, you store all `Names` together, all `Ages` together, etc.
2.  **Compression:** Because the data in a column is similar (e.g., many people have the age "29"), it compresses incredibly well using **Bitmap Encoding** or **Run-Length Encoding**.
3.  **Vectorization:** Using modern CPU instructions (SIMD) to process a whole chunk of a column at once.

---

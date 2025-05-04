# Why is the current implementation slow? (binary search tree)

## Comparison (using Mac's Instrument profiler)

### Streaming search on local file system

![CleanShot 2025-03-20 at 10.49.59@2x.png](attachment:89bae45b-7276-4df7-a777-02b0ee2e2efa:CleanShot_2025-03-20_at_10.49.592x.png)

### Read all into memory then search

![CleanShot 2025-03-20 at 10.51.04@2x.png](attachment:f8d004ca-3598-491d-9bae-69a696faf7ef:CleanShot_2025-03-20_at_10.51.042x.png)

## Insight

- Current streaming search invokes many system calls, resulting in high disk I/O latency

# The gap between initial plan and result

## Initial assumption

Binary search would perform better than B-tree index because it eliminates redundant key comparisons.

## What I found after implementation

Rather than gaining performance from fewer key comparisons, I/O latency has a larger impact on overall search query performance. As described below, most databases employ B-tree indexing not just for its logarithmic time complexity but also for I/O latency optimisation.

# How does other DB system address to this issue?

## What kinds of indexing does it have? and how they are stored?

## Kinds of indices

Various types of indexes that serve as auxiliary access structures to speed up the retrieval of records from data files based on specific search conditions. These indexes are essentially additional files on disk that provide secondary access paths, offering alternative ways to access records without altering their physical placement in the primary data file. The chapter primarily focuses on indexes based on ordered files (single-level indexes) and tree data structures (multilevel indexes, B+-trees).

In the context of database storage, data is organised into **blocks** or **pages** on disk. These terms are often used interchangeably. A block or page is a contiguous unit of storage of a fixed size (e.g., 4KB, 8KB, 16KB) that is the basic unit of data transfer between the disk and main memory. When the database needs to access a record, it reads the entire block containing that record into a memory buffer. Indexes work by providing a more efficient way to locate the specific blocks that contain the desired data, thereby reducing the need to scan many blocks unnecessarily.

**Pointers** are crucial components of index structures. They essentially hold the address of a specific disk block or a record within a block. By following these pointers in the index, the DBMS can directly access the location of the data it needs.

**1. Primary Index:**

- **Definition and Purpose:** A primary index is an ordered file specifically designed for an ordered data file whose records are physically sorted on a key field known as the ordering key or primary key. Its primary purpose is to provide an efficient means to search for and access records in the data file based on this ordering key. The primary index acts as an access structure.
- **Indexing Field Characteristics:** The indexing field for a primary index *must* be the ordering key field of the data file, and it *must* be a key, meaning it has a unique value for every record.
- **Structure of the Index File:** The primary index is an ordered file containing fixed-length records with two fields. The first field has the same data type as the primary key of the data file. The second field is a **pointer to a disk block** (a block address). There is *one index entry* for *each block* in the data file. Each index entry stores the value of the primary key field of the *first record* in a block (known as the block anchor) and a pointer to that block. These entries are ordered by the primary key value.
- **Dense or Sparse:** A primary index is a **nondense (sparse)** index because it contains an entry for each disk block of the data file, referencing the key of its anchor record, rather than for every single record.
- **Block Anchors:** Primary indexes utilise **block anchors**, which are typically the first (or sometimes the last) record in each block of the data file. The index entry points to the block containing this anchor record.
- **Data Retrieval:** To retrieve a record with a specific primary key value, a binary search is performed on the primary index file to find the index entry where the key value is less than or equal to the search key and the next key value is greater. The pointer in this index entry then directs the DBMS to the specific disk block in the data file that contains the desired record (and potentially other records with primary key values within the range implied by the adjacent index entries). One additional block access to the data file is then needed.
- **Impact on Updates:** Insertion and deletion of records in an ordered file with a primary index are complex. Inserting a record might require shifting other records in the data file to maintain the order and updating the index entries if block anchors change. Using an unordered overflow file or a linked list of overflow records for each block can mitigate this problem. Deletion can be handled using deletion markers.

**2. Clustering Index:**

- **Definition and Purpose:** A clustering index is used when the file records are physically ordered on a nonkey field, known as the clustering field. This physical ordering results in a clustered file. The purpose of a clustering index is to speed up the retrieval of *all* records that have the same value for the clustering field.
- **Indexing Field Characteristics:** The indexing field for a clustering index is the ordering field of the data file, but unlike a primary index, this field is a *nonkey* field, meaning multiple records can have the same value for it.
- **Structure of the Index File:** Similar to a primary index, a clustering index is an ordered file with two fields. The first field has the same data type as the clustering field of the data file. The second field is a **disk block pointer**. However, unlike a primary index, there is *one entry* in the clustering index for *each distinct value* of the clustering field in the data file. This entry contains the distinct clustering field value and a pointer to the *first block* in the data file that contains a record with that value.
- **Dense or Sparse:** A clustering index is generally considered a **nondense (sparse)** index because it has an entry for each distinct value of the clustering field, not for every record.
- **Block Anchors:** Clustering indexes implicitly use the first record with each distinct clustering field value as an anchor to the subsequent records with the same value, which are physically clustered together.
- **Data Retrieval:** To retrieve all records with a specific clustering field value, a binary search on the clustering index locates the entry with that value. The pointer then leads to the first block containing such records. Since the file is ordered on this field, subsequent consecutive blocks will likely also contain records with the same value, until the value changes. Sometimes, to handle insertions efficiently, a whole block (or a cluster of contiguous blocks) is reserved for each value of the clustering field, with a block pointer in the index pointing to the start of this block cluster.
- **Impact on Updates:** Similar to primary indexes, insertion and deletion can be problematic due to the physical ordering of the data records. Reserving block clusters for each clustering field value helps alleviate insertion issues by providing space within the cluster.

**3. Secondary Index (on a Key Field):**

- **Definition and Purpose:** A secondary index provides an alternative way to access a data file for which some primary access method already exists (the data file could be ordered, unordered, or hashed). A secondary index on a key field is created on a field that is a candidate key and thus has a unique value in every record.
- **Indexing Field Characteristics:** The indexing field for a secondary index (on a key) is a nonordering field of the file that is also a key (has unique values).
- **Structure of the Index File:** Like primary and clustering indexes, a secondary index is an ordered file with two fields. The first field has the same data type as the nonordering key field of the data file (the indexing field). The second field is either a **block pointer** or, more commonly for secondary indexes (especially on key fields where each value points to a unique record), a **record pointer**. A record pointer uniquely identifies a record and provides its address on disk, often as a block number and an offset within the block. An index entry is created for *each record* in the data file. The entries are ordered by the value of the secondary key field.
- **Dense or Sparse:** A secondary index on a key field is a **dense** index because it contains an entry for every record in the data file. Since each key value is unique, each index entry corresponds to a unique record.
- **Block Anchors:** Secondary indexes do not rely on block anchors in the same way as primary or clustering indexes because the data file is not ordered based on the secondary key.
- **Data Retrieval:** To retrieve a record based on a secondary key value, a binary search is performed on the secondary index to find the entry matching the key. The pointer (block or record pointer) in this entry directly leads to the block (and potentially the record within the block if it's a block pointer) or directly to the record itself (if it's a record pointer). If the pointer is a block pointer, one additional disk access is needed to fetch the data block and then locate the record within it.
- **Impact on Updates:** When a record is inserted or deleted, the secondary index must be updated to reflect these changes. If a record is moved to a different disk location, and the secondary index uses physical record pointers, these pointers would need to be updated. Logical indexes can address this by providing an extra level of indirection.

**4. Secondary Index (on a Nonkey Field):**

- **Definition and Purpose:** Similar to a secondary index on a key field, this index provides an alternative access path to a data file based on a nonordering field that may contain duplicate values. Its purpose is to improve the efficiency of queries that select records based on this nonkey field.
- **Indexing Field Characteristics:** The indexing field for a secondary index (on a nonkey) is a nonordering field of the file that may have duplicate values across different records.
- **Structure of the Index File:** The index file is ordered by the values of the nonkey indexing field. The second field in the index entry can be a **block pointer** or a **record pointer**. Since multiple records can have the same value for the nonkey field, there are a few common implementation options:
  - **Option 1 (Dense with Block Pointers):** Create an index entry for each record with the nonkey value and a pointer to the block containing that record. For a given nonkey value, multiple index entries might point to the same block if several records with that value are in the same block.
  - **Option 2 (Nondense with Block Pointers):** Create one index entry for each distinct nonkey value and a pointer to the first block containing that value. This is similar to a clustering index, but the data file is not physically ordered on this nonkey field. This is less common for secondary indexes on nonkey fields because it only leads to the first block and requires further searching in subsequent blocks that might contain the same value.
  - **Option 3 (Nondense with Indirection):** This is a more common approach. Create a single index entry for each distinct nonkey value. The pointer in this entry points to a disk block that contains a *set of record pointers* (or sometimes block pointers) to all the data file records that have that nonkey value. If the number of such records is large, a linked list or cluster of blocks might be used to store all the record pointers.
- **Dense or Sparse:** A secondary index on a nonkey field can be **dense** (if there's an index entry for every record) or **nondense** (if there's an entry for each distinct value, potentially with an extra level of indirection). Option 1 above results in a dense index, while Option 3 leads to a nondense first level index but requires accessing additional blocks for the record pointers.
- **Block Anchors:** Similar to secondary indexes on key fields, these indexes do not inherently rely on block anchors of the data file since the file isn't ordered on the indexing field.
- **Data Retrieval:**
  - **Option 1 (Dense with Block Pointers):** A binary search finds all index entries with the desired nonkey value. The block pointers are then used to retrieve the corresponding data blocks, and the records are located within those blocks. This can result in accessing the same data block multiple times if records with the same nonkey value are scattered across the file.
  - **Option 3 (Nondense with Indirection):** A binary search on the first-level index finds the entry for the desired nonkey value. The pointer leads to a block (or blocks) containing record pointers. These record pointers are then used to directly retrieve the records from the data file. This method introduces an extra level of block access (to the block of record pointers) but can be more efficient than retrieving the same data block multiple times.
- **Impact on Updates:** Insertions and deletions require updating the secondary index. With Option 3, if a new record with a specific nonkey value is inserted, a pointer to this new record needs to be added to the block of record pointers associated with that value. If the block is full, a new block might need to be allocated and linked. Similarly, deletions require removing the corresponding record pointer.

In summary, all these indexing techniques aim to reduce the number of disk block accesses needed to retrieve specific data. They achieve this by creating ordered structures that allow for efficient searching (typically using binary search for single-level ordered indexes) and by using pointers to directly locate the disk blocks or records of interest. The choice of index type depends on whether the indexing field is a key or nonkey, whether the data file is physically ordered on that field, and the types of queries that are most frequently executed on the database. Multilevel indexes (like B-trees and B+-trees, discussed later in Chapter 17) are then built on top of these single-level index concepts to further enhance search performance by reducing the number of index levels that need to be traversed.

## Summary table

![CleanShot 2025-03-16 at 15.06.55.png](attachment:0b40d5d9-023e-4dc1-bb77-e5008f342dbb:CleanShot_2025-03-16_at_15.06.55.png)

# Sparse index VS Dense index

## Example of sparse index (primary key)

It only contains pointer to blocks (not record)

- **Advantages**:
  - Significantly smaller than dense indexes, reducing storage overhead.
  - Efficient for large datasets because fewer index entries are maintained.
- **Disadvantages**:
  - Lookup is slower compared to dense indexes due to the need for additional searching within blocks.
  - Only applicable when data is sorted by indexing key.

![CleanShot 2025-03-16 at 15.31.52.png](attachment:d22b6f13-7058-4c58-ad61-95717d79ea92:CleanShot_2025-03-16_at_15.31.52.png)

## Example of Dense index (secondary key)

It contains all entries corresponding to every records, thus

- **Advantages**:
  - **Fast lookups**: Every record can be directly accessed via the index.
  - Ideal for exact-match queries and frequently accessed data.
- **Disadvantages**:
  - Larger size due to storing pointers for every row.
  - Can incur overhead in storage and maintenance.

![CleanShot 2025-03-16 at 15.38.08.png](attachment:fc3fca32-0544-4b20-b24a-1534bd23c453:CleanShot_2025-03-16_at_15.38.08.png)

Reference: <https://www.pearson.com/en-us/subject-catalog/p/fundamentals-of-database-systems/P200000003546/9780137502523>

## How does it handle String type?

### Suffix compression

> Indexing of Strings: There are a couple of issues that are of particular concern
when indexing strings. Strings can be variable length (e.g., VARCHAR data type in
SQL; see Chapter 6) and strings may be too long limiting the fan-out. If a B+-tree
index is to be built with a string as a search key, there may be an uneven number of
keys per index node and the fan-out may vary. Some nodes may be forced to split
when they become full regardless of the number of keys in them. The technique of
**prefix compression** alleviates the situation. Instead of storing the entire string in
the intermediate nodes, it stores only the prefix of the search key adequate to distin-
guish the keys that are being separated and directed to the subtree. For example, if
Lastname was a search key and we were looking for “Navathe”, the nonleaf node
may contain “Nac” for Nachamkin and “Nay” for Nayuddin as the two keys on
either side of the subtree pointer that we need to follow.
>

Reference: <https://www.pearson.com/en-us/subject-catalog/p/fundamentals-of-database-systems/P200000003546/9780137502523> chp 17.6.3

### Postgres approach

### 1. TOAST (The Oversized Attribute Storage Technique)

- Large values (e.g., long strings) are stored out-of-line in a separate table.
- The index itself only stores a reference or a shortened prefix (to reduce index bloat).

### 2. Prefix compression

- In the B-tree itself, PostgreSQL may store only a prefix of the string in internal pages to save space, since only comparisons are needed.
- Full value is only stored in leaf pages.

**3. Detoasting on Compare**

- If a string is TOASTed (stored externally), it is detoasted (retrieved) only when necessary for comparison.

<https://www.postgresql.org/docs/current/storage-toast.html>

### Hashing string value?

Hashing string value doesn’t seem to work since the hashed value doesn’t preserve the property of order of original string value, thus we can’t compare with given query value.

# 📂 Block in File System

- File systems write and read data in fixed-size **blocks**, typically **4 KB**.
- When writing data to disk, even if the actual data is smaller than the block size, the file system allocates and uses the **entire block**.
- This introduces potential **internal fragmentation**, especially with many small writes.
- Tools like `ls` and `du` in Unix-like systems reveal the difference between **logical file size** and **physical disk usage** due to this block-based allocation.

---

# ⚡ Cache Gain from Fixed-Sized I/O

- Operating systems maintain a **page cache** (commonly 4 KB pages) to cache disk blocks in memory.
- Reading or writing data in **4 KB aligned, fixed-size chunks**:
  - Minimizes syscall overhead.
  - Leverages page cache efficiently.
  - Enables prefetching and avoids read-modify-write cycles.
- Accessing data using **aligned offsets and sizes** (e.g., reading one B-tree node = one page) ensures optimal interaction with OS-level caching and disk I/O.

---

# 🗃️ The Strategy That RDBMSs Use for Persisting B-tree Indexes

- Relational databases (e.g., PostgreSQL, MySQL) use **B-trees** or **B+ trees** to store indexes on disk.
- Each **B-tree node** is encoded as a **single page (usually 4 KB)**.
- Pages are aligned and accessed using offset-based I/O — often with `mmap` or large block reads.
- This strategy supports:
  - **Efficient page-level caching**
  - **Sequential prefetching**
  - **Minimal I/O operations during index traversal**

---

# 🔍 Insight into My Research

### ✅ Use Similar Strategy to RDBMS

- Design a file format that stores a **B-tree index in a fixed-size, page-aligned layout**.
- Persist each B-tree node as a 4 KB block.
- Align reads and writes to these node boundaries for maximal benefit from OS **page cache** and **hardware caching**.

### ✅ Use Static B-tree Instead of Binary Search Tree

- While **binary search trees** can be faster in memory (due to pointer-based branching), they are **not cache-friendly** and require many small, scattered accesses.
- A **static B-tree** or **B+ tree** with more entries per node:
  - Groups related keys together in each node → better **spatial locality**
  - Reduces depth of the tree → fewer I/O operations per search
  - Suits **streaming access patterns** well, such as reading over HTTP or loading partial index segments into memory (batch request per a page)

# Btree vs Binary search tree

> The main advantage of this approach is that it reduces the tree height by
>
>
> $$
> \log_k n = \frac{\log n}{\log k} = \frac{\log k^h}{\log k} = \log_2 k \text{ times},
> $$
>
> while fetching each node still takes roughly the same time --- as long as it fits into a single [memory block](https://en.wikipedia.org/wiki/Memory_block).
>
> B-trees were primarily developed for the purpose of managing on-disk databases, where the latency of randomly fetching a single byte is comparable with the time it takes to read the next 1MB of data sequentially. For our use case, we will be using the block size of $B = 16$ elements --- or 64 bytes, the size of the cache line --- which makes the tree height and the total number of cache line fetches per query
>
> $$
> \log_2 17 \approx 4
> $$
>
> times smaller compared to the binary search.
>

<https://en.algorithmica.org/hpc/data-structures/s-tree/>

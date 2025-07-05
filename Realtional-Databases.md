### Notes on Relational Databases and Database Indexes

#### **Relational Databases**
1. **Data Storage**:
   - Data is stored in **tables** with rows and columns.
   - Example: A table might represent "Employees" with columns like ID, Name, and Department.

2. **Key Highlights**:
   - **Relations**: The core feature of relational databases is the ability to establish relationships (e.g., one-to-many, many-to-many) between tables.
   - **ACID Properties** (referred to as "AC10" in the notes, likely a typo):
     - **A**tomicity: Ensures all operations in a transaction are completed; if not, none are.
     - **C**onsistency: Ensures data remains in a valid state before and after transactions.
     - **I**solation: Ensures concurrent transactions do not interfere with each other.
     - **D**urability: Ensures committed transactions remain permanent even after system failures.

3. **When to Use**:
   - Choose relational databases when you need to manage structured data with clear relationships (e.g., customer orders, inventory systems).

---

#### **Database Indexes**
1. **Purpose**:
   - Indexes improve query performance by allowing faster data retrieval (similar to an index in a book) and wiriting slower.
   - Example: Creating an index on a "CustomerID" column speeds up searches for specific customers.

   ---
   ### **How Database Indexes Make Queries Faster**

Indexes in databases work like the index of a book—they help locate data quickly without scanning the entire table. Here’s how they improve query performance:
1. **Without an Index (Full Table Scan)**

    The database scans every row in the table to find matching records.

    Slow for large tables (O(N) time complexity).

    Example:
    sql

    SELECT * FROM users WHERE username = 'alice'; -- Without an index, checks every row

2. **With an Index (Faster Lookup)**

    Indexes create a sorted data structure (often a B-Tree or Hash) for specific columns.

    Queries use the index to jump directly to the relevant data (O(log N) time for B-Trees).

    Example:
    sql

    CREATE INDEX idx_username ON users(username); -- Creates an index
    SELECT * FROM users WHERE username = 'alice'; -- Now uses the index for fast lookup

3. **How Indexes Work Internally**

    B-Tree Index (Most Common)

        Balanced tree structure allowing fast searches, inserts, and deletes.

        Each node contains keys and pointers to child nodes or data.

    Hash Index

        Uses a hash function for exact-match lookups (O(1) time).

        Not suitable for range queries.

4. **When to Use Indexes**

    Good for:

        Columns frequently used in WHERE, JOIN, or ORDER BY.

        High-selectivity columns (e.g., unique usernames, IDs).

    Avoid for:

        Columns with few unique values (e.g., gender).

        Tables with frequent write operations (indexes slow down inserts/updates).

5. **Downsides of Indexes**

    Storage Overhead: Indexes consume extra disk space.

    Write Penalty: Inserts/updates must also update the index.

### **Database Locking: Pessimistic Locking**  

#### **Core Idea**  
- **"Acquire the lock before proceeding"** (`ACQ_Lock()`).  
- Assumes conflicts are likely, so locks are taken **early** to prevent concurrent modifications.  

---

### **Two Types of Locks**  
1. **Shared Lock (S-Lock) (Read Locks)**  
   - Multiple transactions can **read** data simultaneously.  
   - Prevents **exclusive locks** (no writes allowed while shared locks exist).  
   - If the current transaction wants to modify then the lock  will be **upgraded** to exclusive locks 
   - Example:  
    **Transaction1**
     ```sql
     SELECT * FROM accounts WHERE id = 1,2,6 LOCK IN SHARE MODE; -- id row with id 1,2,6 will be locked
     ```
     
     **Transaction 2**
     ```sql
     SELECT * FROM accounts WHERE id = 3,4,6 LOCK IN SHARE MODE; -- id row with id 3,4,6 will be locked
     ```
     Now while **Transaction1** is active **Transaction 2** can read the data but not be able to write into it because it is locked by **transaction 1** so **transaction2** has to wait for transaction 1 to finish modifying so **transaction2** can update 
     
     - This will provide a very high read through put


2. **Exclusive Lock (X-Lock)(Write locks)**  
   - Only **one transaction** can hold it for **writing**.  
   - Blocks **all other locks** (shared & exclusive).  
   - Does not allow any other transaction to even read thw row its updating
   - Example:  
     ```sql
     SELECT * FROM accounts WHERE id = 1 FOR UPDATE; -- MySQL
     ```
3. **Skip Locked**
    - Removes the locked rows from the result set
    - Example:
        ```sql
        Select * from t where id =2
        for update skip locked 
        ```
4.   **No Wait**

        - Locking read doing wait for the lock to be acquited it fails immediately if the row is locked 
        - Example
            ```sql
            Select * from t where id=2
            for update no wait
            ```
---


### **Why Do We Need Locks?**  
- Protect **data consistency & integrity** (avoid race conditions).  
- Prevent:  
  - **Lost Updates** (two transactions overwrite each other).  
  - **Dirty Reads** (reading uncommitted changes).  
  - **Non-Repeatable Reads** (same row changes during a transaction).  

---

### **Risk: Transaction Deadlock**  
- Occurs when:  
  - Transaction A holds Lock 1 and waits for Lock 2.  
  - Transaction B holds Lock 2 and waits for Lock 1.  
- **Database resolves deadlocks by**:  
  - **Aborting one transaction** (victim) to break the cycle.  
  - The aborted transaction must **retry**. 
  

---

### **When to Use Pessimistic Locking?**  
**Best for**:  
- High-contention scenarios (e.g., banking systems, inventory management).  
- Critical operations where consistency **must** be guaranteed.  

 **Avoid if**:  
- Most operations are **read-heavy** (use optimistic locking instead).  
- Long-running transactions (locks block other transactions).  




# **Airline Check-In System**

 - Multiple Airlines
 - Every airline has 120 seats
 - Every flight has multiple trips
 - user books a seat in one type of a flight
 * Handle multiple people trying to pick seats 

![alt text](image-1.png) ![alt text](image.png)




### **Distributed KV Store**

Creating a key value store over a relational database in order to reducde the latency and improve susteme perforamce

field-> Key, value, expired at
```sql
Delete from store where expred_at >now
```
only one line is needed to delete the row with no mathematical opertaions

insert

```sql
Upsert into store value (k,v1,e)
```

```sql
upsert store set exp_at=-1 where key =k
```
GET 

```sql
select * from store where key =k and expred_at > now
```

**How to hard delete** (BATCH AND PERODIC)
- Run a seperate cleanup process ro hard delete , the soft deleted rows minimize I/0 & rebalancing

**Insert new keys/ values**
1. Key exist -> update
2. key does not exist -> insert

* instead of applying GET & insert/Update
we use Upsert
mysql:- Replace into 'store' values(k,v,el)
postgresql -> upsert statement

**Multiple put requests?**
 - Apply locks exclusive locks For Update or No wait or Skip row or play with database isolation levels 

**Implementing TTL**

- **Approach 1**: Batch deletion with CronJob (Staggered eleteion)
select * from store where key =k and ttl > now

- **Approach 2** : LAzy deletion [In MEM] Using malloc delete
    - When someone access the key which is expred it will hard delete itself
    - Do hard delete when an expired key is fetched
    - Key is never deletd
    - Cron is delete expred key -> Small pause
**Approach 3** : random Sampling & Deletion
 * Not suitable for disk based Dbs
 - Randomly sample 30 keys having expiration set
 - Delete all the sets keys that are expired from the sample
 - If delete key >25% , repet the process

idea: if sampe has <25% of expired keys, popualtion will have < 25% of expired keys 
- The approach is used by redis & the number ( sample size 20 comes from the central limit theorem)

Implemetning Delete
```sql
Update store set ttl =-1 where key=k1 and ttl > now() ---reduces the load on disk
```

Implementing Batch cleanup delete
```sql
Delete from store where ttl <= now() --- expired and soft deleted
``` 
Implementing get
```sql
select * from store 
where key =k1 and ttl > now()
```

![alt text](image-2.png)

- If we have 99:1 read request and one DB is not able to handle it we add read replicas
![alt text](image-3.png)

# Homework

 **We add replicals only when**
    - Reading stale dat is okay
    - read: write = 99:1
if master is now seeing a lot of writes
- We first vertically scale up
- if it is still not able to handle the load we partition
- Intelligent balancing

Distributed transaction doesnt huarantees acid from transaction to trabs action but with in
router info will be in the KV-API

![alt text](image-4.png)



# In PostgreSQL, everything is a transaction

- The simple `SELECT` statement is treated as a separate transaction
- If we want some statements to be parts of the same transaction, the `BEGIN` statement must be used
- Use `COMMIT` to end transaction
- Use `ROLLBACK` to abort transaction without commit/change anything
- Inside transaction, after an error has occured, no more statements/instructions will be accepted. So only thing we can do is `COMMIT` to end this transaction
- It's possible to run DDLs (commands that change data structure) inside a transaction block

# Basic locking

- Multi-Version Concurrency Control (MVCC): A transaction can see only changed data that have already been committed.
- So, in Postgre, reads and writes can coexist. Write transactions won't block read transactions.
- If there're 2 `UPDATE` statement is run at the same time, Postgre guarantees that one `UPDATE` statement is performed after another.
- Postgre will only lock rows that affected by `UPDATE`. So if we have X rows, we can theoretically run X concurrent changes.

# Explicit Locking
- Use `LOCK` statement is used if we want explicitly lock table in a transaction
- Postgre offers these lockmode:
  - `ACCESS SHARE` : affect read transactions. `SELECT` cannot start if a table is about to be dropped. `DROP TABLE` has to wait until a read transaction is completed
  - `ROW SHARE` : lock in the case of `SELECT FOR UPDATE` / `SELECT FOR SHARE`. Conflict with `EXCLUSIVE` & `ACCESS EXCLUSIVE`
  - `ROW EXCLUSIVE` : lock in `INSERT` / `UPDATE` / `DELETE`
  - `SHARE UPDATE EXCLUSIVE`: 
  - `SHARE` : 
  - `SHARE ROW EXCLUSIVE` : 
  - `EXCLUSIVE` : is the most restrictive one. It protects against reads and writes alike
  - `ACCESS EXCLUSIVE` : prevents concurrent transactions from reading and writing.
  

# Making use of `FOR SHARE` and `FOR UPDATE`
- `SELECT ... FOR UPDATE` will lock rows just like `UPDATE` would. No changes can happen concurrently. All locks will be released on `COMMIT`
- If one `SELECT ... FOR UPDATE` is waiting for another `SELECT ... FOR UPDATE` command, we will have to wait until first one completes, or wait forever.
- To avoid this:
  - `SELECT ... FOR UPDATE NOWAIT` : throw error if locked and nowait
  - `SET lock_timeout TO ...` (Eg. 5000) : wait 5 seconds before error
  - `SELECT ... FOR UPDATE SKIP LOCKED` : skip locked rows and return next rows
- If `SELECT FOR UPDATE` is run on table X, the `UPDATE` command on table Y that have `FOREIGN KEY` in table X will be locked
- Another type `SELECT ... FOR ...`:
  - `FOR NO KEY UPDATE`:
  - `FOR SHARE`:
  - `FOR KEY SHARE`: 

# Understanding transaction isolation levels (Very important topic)

- By default, Postgre runs in the `READ COMMITTED` transaction isolation level. Every statement inside a transaction will get a new snapshot of data
- To run reports that need data to be consistent and use snapshot throughout entire transaction, use `REPEATABLE READ` transaction isolation level
- `SERIALIZABLE SNAPSHOT` isolation level: 
- Postgre DO NOT support `READ UNCOMMITED`

# Deadlocks
- deadlock can happen when at least 2 transactions have to wait on each other

# Advisory locks
- Using a number to lock. Synxtax `SELECT pg_advisory_lock(int: x)` -> `COMMIT` -> `SELECT pg_advisory_unlock(x)`
- If a number `x` is locked, another transaction using advisory lock with `x` have to be waited
- Using `pg_advisory_unlock_all()` to unlock all numbers


# Vacuum
- `VACUUM` will visit pages that contain modifications and find all the dead space.
- The free space that's found is then tracked by the `free space map (FSM)`
- Note that `VACUUM` will, in most cases, not shrink the size of a table.

# Configure AUTOVACUUM
- a tool called `autovacuum`, is a part of PostgreSQL server infra. We can configure it in `postgresql.conf`
  - `autovacuum_vacuum_scale_factor`: % of data need to be changed to trigger `autovacuum`
  - `autovacuum_vacuum_threshold`: `autovacuum_vacuum_scale_factor` must be at least x rows
  - `autovacuum_analyze_scale_factor`: % of data need to be changed to justify new optimizer
  - `autovacuum_analyze_threshold`: `autovacuum_analyze_scale_factor` must be at least x rows
  - `autovacuum_vacuum_insert_threshold`: if transaction consists only `INSERT`, and x rows is inserted, then `autovacuum` is triggered
  - `autovacuum_freeze_max_age`: defines maximum number of transactions that `pg_class.relfrozenxid` before `VACUUM` is forced
  - `autovacuum_multixact_freeze_max_age`: defines maximum age that `pg_class.relminmxid` before `VACUUM` is forced

# Understand of VACUUM with `pg_relation_size`
- `pg_relation_size`: returns the size of a table in bytes
- when we run `UPDATE`, the database engine has to copy all the affected rows. Because, Postgre doesnt know wheter this transaction will be successful and it has to deal with concurrent transaction
- So, the size of table will be larger (check by `pg_relation_size`)
- So, after run `VACUUM`, the size of table will remain. It just mark some space to be reused
- The next `UPDATE` will not make the table grow because it will use free space that `VACUUM` marks above

-> `VACUUM` actually clean out rows and return free space to disk IF a row cannot be seen by any transaction anymore

# More `VACUUM` features
- `DISABLE_PAGE_SKIPPING`
- `SKIP_LOCKED`
- `INDEX_CLEANUP`
- `PROCESS_TOAST`
- `TRUNCATE`
- `PARALLEL`

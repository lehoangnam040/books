
- Database and most applications now do not just only do things sequentially, but concurrently with many users
- Database concurrency usually deals with:
  - concurrency between readers
  - concurrency between readers and writers
  - concurrency with atomicity, such as read-modify-write operation
  - transactions
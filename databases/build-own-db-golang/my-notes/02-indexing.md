
- There are 2 types of DB: OLAP and OLTP
  - OLAP usually for analytical purposes with large amount of data
  - OLTP usually for transactional query purposes with smaller amount of data

- most software should response in a reasonable amount of time, meaning we should indexing data to meet with the requirements
- if the dataset fits only in memory, finding data quickly is the problem of data structure. But since database store data on disk, the problems can be more than only data structure.

-> Indexing is another concern of database, how do we find data quickly enough even if dataset is large


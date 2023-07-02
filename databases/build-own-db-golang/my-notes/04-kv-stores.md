# Key-Value (KV) Store

## Hashtables
- simple K-V store
- Resizing is O(n) which causes sudden increase in disk space and IO
- Can resizing incrementally, but more complexity

## B-trees (Balanced trees)
- Balanced binary tree:
  - can be queried and updated in O(log(n))
  - can be range-queried
- B-tree can be n-ary balanced tree:
  - less space overhead: 
  with binary tree, only 2 child of each node can cause more pointers. But with B-trees, multiple data can share a parent -> less space is wasted on pointers
  - faster in memory
  - less disk IO
    - shorter means fewer disk seeks
    - using node size as 4K pages, which optimal with OS disks IO

## LSM-Trees (Log-structured merge-tree)

- How to query
  - contains multiple level of data
  - each level is sorted and split into multiple fields
  - a point query start at top level, if not found, continue to next level
  - range query merge results from all levels
- How to update
  - when updating, key inserted into a file from top level first
  - if file size exceeds threshold, merge it with next level
  
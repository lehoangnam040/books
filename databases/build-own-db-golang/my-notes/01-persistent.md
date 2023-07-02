
# What if we dump data directly to files

- What if process while write to files is crashed, or power lost, what is the state of a file?
  - lost the last write
  - half-written file
  - corrupted states
  
-> data is not guaranteed to persist on disk

- Can we achieve persistence without DB?
  - write whole updated dataset to a new file
  - fsync on new file
  - overwrite whole system by using os system rename from old to new
  
-> only acceptable when dataset is tiny


->> Persistence is a concern of database
# Size of databases on disk

```
select TABLE_SCHEMA, round(sum(DATA_LENGTH)/1024/1024/1024,2) as DATASIZE_GB, round(sum(INDEX_LENGTH)/1024/1024/1024,2) as INDEXSIZE_GB from TABLES group by TABLE_SCHEMA;
```

# Size of tables on disk

```
select TABLE_SCHEMA, TABLE_NAME, round(sum(DATA_LENGTH)/1024/1024/1024,2) as DATASIZE_GB, round(sum(INDEX_LENGTH)/1024/1024/1024,2) as INDEXSIZE_GB from TABLES group by TABLE_NAME ORDER BY DATASIZE_GB;
```

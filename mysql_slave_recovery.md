# MySqL Slave Recovery


## On Master

```
RESET MASTER;
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

Save output of master status.

Dump databases:

```
mysqldump -u root -p --all-databases > /some/place/dump.sql
```

Release lock:

```
UNLOCK TABLES;
```

Copy dump to slave.

## On Slave

```
STOP SLAVE;
```

```
mysql -uroot -p < dump.sql
```

```
RESET SLAVE;
CHANGE MASTER TO MASTER_LOG_FILE='FILE', MASTER_LOG_POS=POS;
```

FILE= and POS= from master status output.

```
START SLAVE;
```

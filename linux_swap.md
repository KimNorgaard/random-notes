# See swap usage and limit

```
grep CommitLimit /proc/meminfo
grep Committet_AS /proc/meminfo
```

# Prioritize OOM for processes

oom_adj_score between -1000 and 1000.

-1000 = diable OOM-killing.

```
echo <int> > /proc/<pid>/oom_adj_score
```

# Prevent application from swapping

```
mlock(2)
```

---
title: "Finding CPU numbers in Linux"
date: 2018-01-16 16:41:37
---
###The physical CPUs
```
|Thu Jun 16 14:54:45| [kingsley@oc7421025535 ~:(42)]
$ grep 'physical id' /proc/cpuinfo | sort -u | wc -l
1
```

###The core numbers
```
|Thu Jun 16 15:39:19| [kingsley@oc7421025535 ~:(42)]
$ grep 'core id' /proc/cpuinfo | sort -u | wc -l
2
```

###The thread numbers
```
|Thu Jun 16 15:40:07| [kingsley@oc7421025535 ~:(42)]
$ grep 'processor' /proc/cpuinfo | sort -u | wc -l
4
```

From above 3 outputs, we can know this machine has one physical CPU, each have 2 cores, each core have 2 threads.



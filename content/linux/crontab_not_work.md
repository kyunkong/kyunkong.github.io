---
title: "crontab not work"
date: 2018-01-16 16:43:17
---

```
$ ps -ef | grep cron | grep -v grep
    root  4194442        1   0   Jul 26      - 10:51 /usr/sbin/cron

$ ps -T 4194442
```

Log:
`/var/adm/cron/log`



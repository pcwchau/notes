# Index
- [Index](#index)
- [Overview](#overview)
- [Cron Expression](#cron-expression)

# Overview

This is a note page that contains some common features in backend development.

# Cron Expression

- Cron Syntax (Unix): https://www.netiq.com/documentation/cloud-manager-2-5/ncm-reference/data/bexyssf.html
- Cron (Spring boot): https://stackoverflow.com/questions/30887822/spring-cron-vs-normal-cron

```cs
   1    |    2    |   3   |         4        |   5   |        6        |  7
Seconds | Minutes | Hours | Day of the Month | Month | Day of the Week | Year

0 * * * * *  // every minute, hh:mm:00
0 0 * * * *  // every hour, hh:00:00
0 0 0 * * *  // every day at 00:00:00
0 0 0 ? * 1  // every sunday at 00:00:00
```
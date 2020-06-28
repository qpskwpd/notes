##### 插入当前时间

```
NOW()函数以`'YYYY-MM-DD HH:MM:SS'返回当前的日期时间，可以直接存到DATETIME字段中。

CURDATE()以’YYYY-MM-DD’的格式返回今天的日期，可以直接存到DATE字段中。

CURTIME()以’HH:MM:SS’的格式返回当前的时间，可以直接存到TIME字段中。
```

时间格式函数

```sql
SELECT DATE_FORMAT(update_time, '%Y') FROM t_blog;
返回该字段时间中年的全部部分，如2020，2019
SELECT DATE_FORMAT(update_time, '%y') FROM t_blog;
返回该字段时间中年的后两位部分，如20，19

%M：January
%m：01
%D：1st，5th
%d：01，5
%H：7，19
%h：7，7

```


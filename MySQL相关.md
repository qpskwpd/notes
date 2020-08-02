##### 插入当前时间

```
NOW()函数以`'YYYY-MM-DD HH:MM:SS'返回当前的日期时间，可以直接存到DATETIME字段中。

CURDATE()以’YYYY-MM-DD’的格式返回今天的日期，可以直接存到DATE字段中。

CURTIME()以’HH:MM:SS’的格式返回当前的时间，可以直接存到TIME字段中。
```

##### 时间格式函数

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

##### 类型

| 类型                | 范围                               | 说明                                                    |                                                            |
| ------------------- | ---------------------------------- | ------------------------------------------------------- | ---------------------------------------------------------- |
| Char(N) [ binary]   | N=1~255 个字节 binary ：分辨大小写 | 固定长度                                                | std_name cahr(32) not null                                 |
| VarChar(N) [binary] | N=1~255 个字节 binary ：分辨大小写 | 可变长度                                                | std_address varchar(256)                                   |
| TinyBlob            | 最大长度255个字节(2^8-1)           | Blob (Binary large objects)储存二进位资料，且有分大小写 | memo text not null                                         |
| TinyText            | 最大长度255个字节(2^8-1)           |                                                         |                                                            |
| Blob                | 最大长度65535个字节(2^16-1)        |                                                         |                                                            |
| Text                | 最大长度65535个字节(2^16-1)        |                                                         |                                                            |
| MediumBlob          | 最大长度 16777215 个字节(2^24-1)   |                                                         |                                                            |
| MediumText          | 最大长度 16777215 个字节(2^24-1    |                                                         |                                                            |
| LongBlob            | 最大长度4294967295个字节 (2^32-1)  |                                                         |                                                            |
| LongText            | 最大长度4294967295个字节 (2^32-1)  |                                                         |                                                            |
| Enum                | 集合最大数目为65535                | 列举(Enumeration)，Enum单选、Set复选                    | sex enum(1,0) habby set(‘玩电玩’,'睡觉’,'看电影’,'听音乐’) |
| Set                 | 集合最大数目为64                   |                                                         |                                                            |

| 类型                    | 范围                                   | 说明                                                  | 例如                 |
| ----------------------- | -------------------------------------- | ----------------------------------------------------- | -------------------- |
| TinyInt[M] [UNSIGNED]   | -128~127 UNSIGNED ： 0~255             |                                                       | num tinyint unsigned |
| SmallInt[M] [UNSIGNED]  | -32768~32767 UNSIGNED ：0~ 65535       |                                                       |                      |
| MediumInt[M] [UNSIGNED] | -8388608~8388607 UNSIGNED ：0~16777215 |                                                       |                      |
| Int[M] [UNSIGNED]       | -2^31~2^31-1 UNSIGNED ： 0~2^32        |                                                       |                      |
| BigInt[M] [UNSIGNED]    | -2^63~2^63-1 UNSIGNED ： 0~2^64        |                                                       |                      |
| Float [(M,D)]           | -3.4E+38~3.4E+38( 约 )                 | 注： M 为长度， D 为小数,Float 4 bytes,Double 8 bytes |                      |
| Double [(M,D)]          | -1.79E+308~1.79E+308( 约 )             |                                                       |                      |
| Decimal [(M,D)]         |                                        |                                                       |                      |

| 类型      | 范围                                | 说明 |
| --------- | ----------------------------------- | ---- |
| Date      | 日期(yyyy-mm-dd)                    |      |
| Time      | 时间(hh:mm:ss)                      |      |
| DateTime  | 日期与时间組合(yyyy-mm-dd hh:mm:ss) |      |
| TimeStamp | yyyymmddhhmmss                      |      |
| Year      | 年份yyyy                            |      |
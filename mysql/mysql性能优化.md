### mysql 性能优化

- `当只要一行数据时使用 limit 1查询时如果已知会得到一条数据，这种情况下加上 limit 1 会增加性能。因为 mysql 数据库引擎会在找到一条结果停止搜索，而不是继续查询下一条是否符合标准直到所有记录查询完毕。`
- `用 not exists 代替 not inNot exists 用到了连接能够发挥已经建立好的索引的作用，not in 不能使用索引。Not in 是最慢的方式要同每条记录比较，在数据量比较大的操作红不建议使用这种方式。`
- `like很慢，一般来说最好使用fulltext而不是like`
- `char比varchar效率高很多，并且varchar字段不能进行索引，所以确定的字符长度字段最好使用char字段`
- `存储过程执行比一条一条地执行其中的各条语句快`
- `where在查询为空数据时不能使用==，应该书写成以下方式`

### mysql 数据类型的选择和优化案例

- 手机号存储<br>
  使用 BIGINT 代替 CHAR 或者 VARCHAR 存放手机号码。
  这是因为 CHAR 或者 VARCHAR，占用空间大，影响查询性能。
  例如：11 位手机号 CHAR 存储，utf8 编码，则占用 33 个字节；
  使用如果使用 INT 的话，INT 最大只能保存 10 为数据，而手机号为 11 位，会出现溢出，所以使用 BIGINT 占用 8 个字节，支持 11 为数据存储。

- IP 地址可以使用 INT 存储 <br>
  MySQL 里提供了一个很好的函数 INET_ATON(),他负责把 IP 地址转化为数字，而另一个函数 INET_NTOA（）负责将数字转化为 IP 地址，示例如下：注意：INT 使用无符号，这是因为 INT 有符号最大为 2147483647 而无符号 最大为 4294967295，如果使用有符号的话，会出现溢出，使用无符号则不会溢出。

* 避免使用 enum 类型，用 TINYINT 替代<br>
  修改值必须使用 alter、排序的效率比较低、禁止使用数字作为 enum 枚举值

* 尽可能把所有列定义为 not null<br>
  索引 null 列会占用更多的空间
  进行比较时需要作额外的处理
- 用timestamp 代替Datetime，（四个字节，八个字节）

### 数据库索引
- 业务需要的相关索引是根据实际的设计所构造sql语句的where条件来确定的，业务不需要的不要建索引

- 对于经常查询的字段，其值不唯一，也应该考虑建立普通索引，查询语句中该字段条件置于第一个位置，对联合索引处理的方法同样。


### Query语句与应用系统优化
- Insert语句中，根据测试，批量一次插入1000条时效率最高，多于1000条时，要拆分，多次进行同样的插入，应该合并批量进行。注意query语句的长度要小于mysqld的参数 max_allowed_packet
- 查询条件中各种逻辑操作符性能顺序是and,or,in,因此在查询条件中应该尽量避免使用在大集合中使用in,实在要用也要把in里面的数量控制在1000以内
- 尽量避免使用子查询，可以把子查询优化为join操作（子查询的结果集无法使用索引，子查询会产生临时表操作，如果子查询数据量大会影响效率，消耗过多的CPU及IO资源）
 - 使用 ISNULL()来判断是否为 NULL 值。
 - 尽量不要使用物理删除（即直接删除，如果要删除的话提前做好备份），而是使用逻辑删除，使用字段delete_flag做逻辑删除，类型为tinyint，0表示未删除，1表示已删除
 - 如果有 order by 的场景，请注意利用索引的有序性。order by 最后的字段是组合,索引的一部分，并且放在索引组合顺序的最后，避免出现 file_sort 的情况，影响查询性能。
 - 数据库和表的字符集尽量统一使用utf8（字符集必须统一，避免由于字符集转换产生的乱码，汉字utf8下占3个字节）
 - 每张表的索引数量限制在5个以内
 - 同财务相关的金额数据，采用decimal类型（不丢失精度，禁止使用 float 和 double）
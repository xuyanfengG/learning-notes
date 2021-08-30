## 一、索引概念

### 1. 查看已创建索引

```sql
show index from [tableName];
```

> id：SELECT识别符。这是SELECT的查询序列号。
>
> select_type：SELECT类型。
>
> > SIMPLE： 简单SELECT(不使用UNION或子查询) PRIMARY： 最外面的SELECT UNION：UNION中的第二个或后面的SELECT语句 DEPENDENT UNION：UNION中的第二个或后面的SELECT语句，取决于外面的查询 UNION RESULT：UNION的结果 SUBQUERY：子查询中的第一个SELECT DEPENDENT SUBQUERY：子查询中的第一个SELECT，取决于外面的查询 DERIVED：导出表的SELECT(FROM子句的子查询) table：表名
>
> type：联接类型。是SQL性能的非常重要的一个指标，结果值从好到坏依次是：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL。 一般来说，得保证查询至少达到range级别。
>
> > system：表仅有一行(=系统表)。这是const联接类型的一个特例。 const：表最多有一个匹配行，它将在查询开始时被读取。因为仅有一行，在这行的列值可被优化器剩余部分认为是常数。const用于用常数值比较PRIMARY KEY或UNIQUE索引的所有部分时。 eq_ref：对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY。eq_ref可以用于使用= 操作符比较的带索引的列。比较值可以为常量或一个使用在该表前面所读取的表的列的表达式。 ref：对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY(换句话说，如果联接不能基于关键字选择单个行的话)，则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。ref可以用于使用=或<=>操作符的带索引的列。 ref_or_null：该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。 index_merge：该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。 unique_subquery：该类型替换了下面形式的IN子查询的ref：value IN (SELECT primary_key FROMsingle_table WHERE some_expr);unique_subquery是一个索引查找函数，可以完全替换子查询，效率更高。 index_subquery：该联接类型类似于unique_subquery。可以替换IN子查询，但只适合下列形式的子查询中的非唯一索引：value IN (SELECT key_column FROM single_table WHERE some_expr) range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时，可以使用range index：该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。 all：对于每个来自于先前的表的行组合，进行完整的表扫描。如果表是第一个没标记const的表，这通常不好，并且通常在它情况下很差。通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或列值被检索出。
>
> possible_keys：possible_keys列指出MySQL能使用哪个索引在该表中找到行。注意，该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。
>
> key：key列显示MySQL实际决定使用的键(索引)。如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。
>
> key_len：key_len列显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。注意通过key_len值我们可以确定MySQL将实际使用一个多部关键字的几个部分。
>
> ref：ref列显示使用哪个列或常数与key一起从表中选择行。
>
> rows：rows列显示MySQL认为它执行查询时必须检查的行数。
>
> Extra：该列包含MySQL解决查询的详细信息。
>
> > Distinct：MySQL发现第1个匹配行后，停止为当前的行组合搜索更多的行。 Not exists：MySQL能够对查询进行LEFT JOIN优化，发现1个匹配LEFT JOIN标准的行后，不再为前面的的行组合在该表内检查更多的行。 range checked for each record (index map: #)：MySQL没有发现好的可以使用的索引，但发现如果来自前面的表的列值已知，可能部分索引可以使用。对前面的表的每个行组合，MySQL检查是否可以使用range或index_merge访问方法来索取行。 Using filesort：MySQL需要额外的一次传递，以找出如何按排序顺序检索行。通过根据联接类型浏览所有行并为所有匹配WHERE子句的行保存排序关键字和行的指针来完成排序。然后关键字被排序，并按排序顺序检索行。 Using index：从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的列信息。当查询只使用作为单一索引一部分的列时，可以使用该策略。 Using temporary：为了解决查询，MySQL需要创建一个临时表来容纳结果。典型情况如查询包含可以按不同情况列出列的GROUP BY和ORDER BY子句时。 Using where：WHERE子句用于限制哪一个行匹配下一个表或发送到客户。除非你专门从表中索取或检查所有行，如果Extra值不为Using where并且表联接类型为ALL或index，查询可能会有一些错误。 Using sort_union(...), Using union(...), Using intersect(...)：这些函数说明如何为index_merge联接类型合并索引扫描。 Using index for group-by：类似于访问表的Using index方式，Using index for group-by表示MySQL发现了一个索引，可以用来查询GROUP BY或DISTINCT查询的所有列，而不要额外搜索硬盘访问实际的表。并且，按最有效的方式使用索引，以便对于每个组，只读取少量索引条目。

### 2. 删除索引

```sql
drop index [indexName] from [tableName];
# 或
alter table [tableName] drop index [indexName];
```

### 3. 查看查询语句使用索引的情况

```sql
explain [sql]
```

### 4. 创建索引

语法：

```sql
ALTER TABLE [tableName] ADD [UNIQUE | FULLTEXT | SPATIAL]  INDEX | KEY  [索引名] (字段名1 [(长度)] [ASC | DESC]) [USING 索引方法]
```

#### 4.1 主键索引

主索引，根据主键建立索引，不允许重复，不允许空值

```sql
ALTER TABLE [tableName] ADD PRIMARY KEY pk_index('columnName');
```

#### 4.2 唯一索引

用来建立唯一索引的列的值必须是唯一的，允许有空值

```sql
ALTER TABLE [tableName] ADD UNIQUE index_name('columnName');
```

#### 4.2 普通索引

用表中的普通列构建索引，没有任何要求

```sql
ALTER TABLE [tableName] ADD INDEX index_name('columnName');
```

#### 4.3 全文索引

用大文本对象的列构建的索引

```sql
ALTER TABLE [tableName] ADD FULLTEXT INDEX ft_index('columnName');
```

#### 4.4 组合索引

用多个列组合构建索引，多个列的值不允许有空值

```sql
ALTER TABLE [tableName] ADD INDEX index_name('columnName1','columnName2','columnName3');
```

组合索引遵循==”最左前缀“==原则，把最常用作为检索和排序的列放到最前面，依次递减。

例如：组合索引a、b、c

- 完全满足最左原则：当sql中有a或a、b或a、b、c
- 部分满足最左原则：当sql中只有a、c
- 不满足最左原则：当sql中没有a

>  MySQL5.7开始，会自动优化，如：会把c，b，a优化为a，b，c使之完全遵循最左原则；会把c，a优化为a，c使之部 分遵循最左原则，即SQL语句中的对应条件的先后顺序无关。
>
>  如果满足（部分满足）最左原则但是不满足索引自身的使用规则，则组合索引走到此处就不再往下走了。

a. 聚集索引与非聚集索引：

每个InnDB表具有一个特殊的索引成为聚簇索引（也叫聚集索引、聚类索引、簇集索引），如果表上定义有主键，则该主键就是聚簇索引；如果没有定义主键，则会取第一个唯一索引而且只含非空列作为主键，即作为聚簇索引；如果没有这样的列，InnDB会自己产生一个这样的ID值，有六个字节，且是隐藏的，作为聚簇索引。

表中的聚簇索引就是一级索引，除此之外，表上的其他非聚簇索引都是二级索引，又叫辅助索引。

b. 回表

当二级索引无法直接查到（sql中select所需要的所有）列的数据时，会通过二级索引查找到聚簇索引后，再根据聚簇索引查询到（二级索引中无提供的）数据，这种通过二级索引查询出一级索引，再通过一级索引查询二级索引无法提供的数据的过程，就叫做回表。

当无需回表时，不遵循最左原则也是会走组合索引。

例如现有表user：

| id   | name | age  | gender |
| ---- | ---- | ---- | ------ |
| 1    | jack | 18   | 男     |

id为主键，name、age、gender组成联合索引，当执行以下sql时，虽然不满足组合索引的最左原则，但也会走索引

```
select * from user where gender = '男'
```

如果此时user表再插入一列motto，使用上述的sql进行查询时就需要回表，并且查询条件不遵循最左原则，就不会走组合索引。

## 二、MyISAM与InnDB

### 1. MyISAM

> MyISAM引擎使用B+Tree（B+树）做为索引结构，数据与索引文件分开，叶节点的data域存放的是数据记录的地址，主索引与辅助索引在结构上没有任何区别，只是主索引要求key是唯一的，而辅助索引的key可以重复。

MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的key存在，则取出data域中的值作为地址，读取相应的数据记录。

MyISAM的索引方式也叫作“非聚簇索引”。

#### 2. InnDB

> InnDB引擎使用B+Tree（B+树）做为索引结构，数据文件本身就是索引文件，叶节点的data域存放了完整的数据记录，这个索引的key就是数据表的主键，故InnDB要求表必须有主键（MyISAM可以没有），而且应尽量使用自增字段作为主键，因为 InnoDB 数据文件本身是一棵B+Tree,非单调的主键会造成在插入新记录时数据文件为了维持 B+Tree 的特性而频繁的分裂调整,十分低效,而使用自增字段作为主键则是一个很好的选择。如果表使用自增主键,那么每次插入新的记录,记录就会顺序添加到当前索引节点的后续位置,当一页写满,就会自动开辟一个新的页。

InnDB辅助索引叶子节点data域存放的是相应记录主键的值，而不是地址。
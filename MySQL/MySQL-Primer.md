# MySQL-Primer

## 数据存储引擎

### MyISAM
#### 索引 B-TREE ,FULL-TREE

### Innodb
#### 实务支持
#### 行级锁
#### 数据（表数据+索引数据）
#### 表空间 
- 共享空间 ：所有数据和索引
- 独享空间 ： 每个表的数据和索引

## MySQL数据库锁机制
MySQL 各存储引擎使用了三种类型（级别）的锁机制：行级锁，页级锁和表级锁
### 行级锁(row-level)
行级锁最大的特点就是锁对象的颗粒度很小，也是目前各大数据库管理软件所实现的锁定颗粒
度最小的。由于锁颗粒度很小，所以发生锁资源争用的概率也最小，能够给予应用程序尽可能大的,并发处理能力而提高一些需要高并发应用系统的整体性能.
虽然能够在并发处理能力上面有较大的优势，但是行级锁也因此带来了不少弊端。由于锁资源
的颗粒度很小，所以每次获取锁和释放锁需要做的事情也更多，带来的消耗自然也就更大了。此外，行级锁也最容易发生死锁。
### 表级锁(table-level)
和行级锁相反，表级别的锁是 MySQL 各存储引擎中最大颗粒度的锁机制。该锁机制最大的特点是实现逻辑非常简单，带来的系统负面影响最小。所以获取锁和释放锁的速度很快。由于表级锁一次会将整个表锁定，所以可以很好的避免困扰我们的死锁问题.当然，锁定颗粒度大所带来最大的负面影响就是出现锁定资源争用的概率也会最高，致使并大度大打折扣
### 页级锁定(page-level)

## MySQL优化
### Index 优化
#### 避免引擎放弃使用索引而进行全表扫描
- 1.应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。
- 2.对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
- 3.应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：
  select id from t where num is null 
  可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：
  select id from t where num=0
- 4.尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：
select id from t where num=10 or num=20
可以这样查询：
select id from t where num=10
union all
select id from t where num=20
- 5.下面的查询也将导致全表扫描：(不能前置百分号)
select id from t where name like ‘%c%’
优化为：
select id from t where name like ‘c%’
若要提高效率，可以考虑全文检索。
- 6.in 和 not in 也要慎用，否则会导致全表扫描，如：
select id from t where num in(1,2,3)
对于连续的数值，能用 between 就不要用 in 了：
select id from t where num between 1 and 3
- 7.如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：
select id from t where num=@num
可以改为强制查询使用索引：
select id from t with(index(索引名)) where num=@num
- 8.应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：
select id from t where num/2=100
select id from t where num=100 * 2
- 9.应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：
select id from t where substring(name,1,3)=’abc’–name以abc开头的id
select id from t where datediff(day,createdate,’2005-11-30′)=0–’2005-11-30′生成的id
select id from t where createdate>=’2005-11-30′ and createdate<’2005-12-1′
- 10.不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
- 11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
- 12.很多时候用 exists 代替 in 是一个好的选择：
select num from a where num in(select num from b)
用下面的语句替换：
select num from a where exists(select 1 from b where num=a.num)
- 13.并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，如一表中有字段 sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。
- 14.索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。
- 15.应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。
- 16.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会 逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了
- 17.尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

#### 临时表
- 1.尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）
- 2.避免频繁创建和删除临时表，以减少系统表资源的消耗
- 3.临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件，最好使 用导出表。
- 4.在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先create table，然后insert。
- 5.如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。

#### 表级优化
- 1.对大表定期执行ANALYZE TABLE(更新表统计信息)、OPTIMIZE TABLE(优化表存储，减少存储碎片，InnoDB应该使用ALTER TABLE TABLE.name ENGINE='InnoDB' )有利于查询优化器对QL进行优化（数据库空闲时执行，尤其是OPTIMIZE TABLE语句）
- 2.一个大表设计的时候应当考虑，将经常被检索的字段放一起，不经常被检索的应当设计一个从表，有需要时再做关联、 不同表中用于关联的列应当保持一致的数据类型

## MySQL 千万级别表优化
优化顺序：
- 1）优化sql和索引
- 2）加缓存，redis等
- 3）读写分离
- 4）垂直拆分，业务上做拆分
- 5）水平切分，针对数据量大的表，这一步最麻烦，最能考验技术水平，要选择一个合理的sharding key,为了有好的查询效率，表结构也要改动，做一定的冗余，应用也要改，sql中尽量带sharding key，将数据定位到限定的表上去查，而不是扫描全部的表




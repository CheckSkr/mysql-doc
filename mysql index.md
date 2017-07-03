

# 1.基础篇

### 每个程序员该知道一些计算机的时间数据
    L1 cache ............................ 0.5 ns
    Branch mispredict(转移、分支预测) ....... 5 ns
    L2 cache  ............................. 7 ns
    Mutex lock/unlock(互斥锁\解锁)........... 25 ns
    Main memory(内存)  ..................... 100 ns
    1k字节压缩(Zippy)  ...................... 3,000 ns = 3 µs
    在1Gbps的网络上发送2k字节 ................. 20,000 ns = 20 µs
    SSD随机读 ............................... 150,000 ns = 150 µs
    从内存顺序读取1MB ........................ 250,000 ns = 250 µs
    同一个数据中心往返一次 ..................... 500,000 ns = 0.5 ms
    从SSD顺序读取1MB ......................... 1,000,000 ns = 1 ms
    磁盘检索 ................................. 10,000,000 ns = 10 ms
    从磁盘里面读出1MB ........................ 20,000,000 ns = 20 ms
 

***没感觉?上面的时间扩大10亿倍的直观感受：***

### Minute:
    L1 cache: 0.5 秒(一次心跳)
    Branch mispredict 5秒
    L2 cache： 7 秒
    Mutex lock/unlock: 25秒(做一杯咖啡的时间)
### Hour:
    Main memory reference: 100秒(刷个牙齿的时间)
    1k字节压缩(Zippy):  50 分钟（一集电视剧[含广告时间]）
### Day:
    在1Gbps的网络上发送2k字节: 5.5 小时(下午上班的时间)
    
### Week
    SSD随机读： 1.7 天 (一个周末)
    从内存顺序读取1MB： 2.9 天 (一个小长假)
    同一个数据中心往返一次： 5.8 天 (近一个国庆假期)
    从SSD顺序读取1MB： 11.6 天(近两周)
   
### Year
    磁盘检索 16.5 周 (大学的一学期)
    从磁盘里面读出1MB： 7.8 个月(差不多生个娃的时间)   
    
    
 
## 基础数据结构
 B+树的演变

>     二叉树 --> 二叉查找树 --> 平衡二叉树 --> B树 --> B+树


**二叉树**

>     1.每个节点最多只能有两个叶子节点

**二叉查找树**

>     1. 每个节点最多只能有两个叶子节点
>     2. 左子树的键值总是小于根的键值，右子树的键值总是大于根的键值


**平衡二叉树**

>     1.符合二叉查找树的定义
>     2.满足任何的左右子树的高度差为1

**B树**
性质：是一种多路搜索树（并不是二叉的）：

    1.定义任意非叶子结点最多只有M个儿子；且M>2；
    2.根结点的儿子数为[2, M]；
    3.除根结点以外的非叶子结点的儿子数为[M/2, M]；
    4.每个结点存放至少M/2-1（取上整）和至多M-1个关键字；（至少2个关键字）
    5.非叶子结点的关键字个数=指向儿子的指针个数-1；
    6.非叶子结点的关键字：K[1], K[2], …, K[M-1]；且K[i] < K[i+1]；
    7.非叶子结点的指针：P[1], P[2], …, P[M]；其中P[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M-1]的子树，其它P[i]指向关键字属于(K[i-1], K[i])的子树；
    8.所有叶子结点位于同一层；
    
   ![image](http://p.blog.csdn.net/images/p_blog_csdn_net/manesking/4.JPG)
    

搜索过程：从根结点开始，对结点内的关键字（有序）序列进行二分查找，如果命中则结束，否则进入查询关键字所属范围的儿子结点；重复，直到所对应的儿子指针为空，或已经是叶子结点；

    1.关键字集合分布在整颗树中；
    2.任何一个关键字出现且只出现在一个结点中；
    3.搜索有可能在非叶子结点结束；
    4.其搜索性能等价于在关键字全集内做一次二分查找；
    5.自动层次控制；
    6.由于限制了除根结点以外的非叶子结点，至少含有M/2个儿子，确保了结点的至少利用率，其最底搜索性能为lgN；
    7.B-树的性能总是等价于二分查找（与M值无关），没有B树平衡的问题；
    8.由于M/2的限制，在插入结点时，如果结点已满，需要将结点分裂为两个各占M/2的结点；删除结点时，需将两个不足M/2的兄弟结点合并；



**B+树**
B+树是B树的变体，也是一种多路搜索树：
>     1.其定义基本与B-树同，除了：
>     2.非叶子结点的子树指针与关键字个数相同；
>     3.非叶子结点的子树指针P[i]，指向关键字值属于[K[i], K[i+1])的子树（B-树是开区间）；
>     4.为所有叶子结点增加一个链指针；
>     5.所有关键字都在叶子结点出现；
![image](http://p.blog.csdn.net/images/p_blog_csdn_net/manesking/5.JPG)




 B+树的特性：
>     1.所有关键字都出现在叶子结点的链表中（稠密索引），且链表中的关键字恰好是有序的；
>     2.不可能在非叶子结点命中；
>     3.非叶子结点相当于是叶子结点的索引（稀疏索引），叶子结点相当于是存储（关键字）数据的数据层；
>     4.更适合文件索引系统；


B+树的插入删除：
>     B+树的维护（算法演示：https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html）
```
var tds = $('#AlgorithmSpecificControls').find('td');
tds.eq(0).find('input').attr('id',"addTxt");
tds.eq(1).find('input').attr('id',"addBtn");


function addNode(count,startValue){
	for(var i =0 ;i<count;i++){
		(function(i){
			setTimeout(function(){
				$('#addTxt').val(startValue+100*i);
				$('#addBtn').trigger('click');
			},1000*i);
		})(i)
		
	}
};

function  addOne(value){
    $('#addTxt').val(value);
	$('#addBtn').trigger('click');
};

```

**问题:为什么MySQL索引(InnoDB)采用B+树而不是B树？**
>     1.B-树和B+树最重要的一个区别就是B+树只有叶节点存放数据，其余节点用来索引，而B-树是每个索引节点都会有Data域。这就决定了B+树更适合用来存储外部数据，也就是所谓的磁盘数据。
>     2.B+树所有的Data域在叶子节点，一般来说都会进行一个优化，就是将所有的叶子节点用指针串起来。这样遍历叶子节点就能获得全部数据，这样就能进行区间访问
>     3.B+树支持range-query(区间查询)非常方便，而B树不支持










# 理论篇

### MySQL索引(InnoDB)

- MySQL索引本质上就是一种数据结构，目标是高效的获取数据。作用类似一本书或字典的索引。主要是以空间换取时间，以牺牲写性能换取高效读的性能。
- 索引是在mysql的存储引擎实现的，不是mysql server层
- 一次查询可能使用到多个索引(merge index,老版的mysql5.0查询基本就是走一个索引)

**关于索引的基础名词**
- 索引基数(cardinality): 不重复的索引值。
- 索引的区分度(selectivity):索引基数/记录总数，(select count(distinct(colX))/count(colX) from table )。
- 回表(access table):如果执行计划里出现table access by rowid说明要回表。回表次数越多，性能越差。
- 索引的分级(三星索引 three-star system)

```
    索引将相关的记录放到一起则获得一星；
    如果索引的顺序和查找中的排序顺序一致，则获得二星；
    如果索引中的列包含了查询中所有的列，则获得三星
```

 **InnoDB索引的分类**
> 主键索引: 简称主键，原文是PRIMARY KEY，由一个或多个列组成，用于唯一性标识数据表中的某一条记录。一个表可以没有主键，但最多只能有一个主键，并且主键值不能包含NULL

> 辅助索引:就是我们常规所指的索引，原文是SECONDARY KEY。辅助索引里还可以再分为唯一索引，非唯一索引。唯一索引其实应该叫做唯一性约束，它的作用是避免一列或多列值存在重复，是一种约束性索引。

> HASH索引：类似java的HashMap，InnoDB不支持物理的HASH索引，但是它有一个自适应索引(adaptive hash index)，就是当InnoBB注意到某些索引的值被使用的非常频繁，那么它会在内存中基于B+Tree
  索引的基础上进一个hash索引，该过程完全是InnoDB处理的，用户无法控制或配置。(因为HASH索引和JAVA HASHMAP，类似有快速查找的功能，所以我们有时候会建立伪HASH索引，参见后面)

> 全文索引： FULLTEXT,参见：https://dev.mysql.com/doc/refman/5.7/en/fulltext-search.html 
  
  在MySQL中，InnoDB数据表的主键设计我们通常遵循几个原则：
```
    1. 采用一个没有业务用途的自增属性列作为主键；
    2. 主键字段值总是不更新，只有新增或者删除两种操作；
    3. 不选择会动态更新的类型，比如当前时间戳等。
    
    这么做的好处有几点：
    新增数据时，由于主键值是顺序增长的，innodb page发生分裂的概率降低了；
    业务数据有变更时，不修改主键值，物理存储位置发生变化的概率降低了，innodb page中产生碎片的概率也降低了。
    
    自增属性列作为主键劣势：
    在访问插入比较频繁的业务表中，自增属性列的上界可能会成为 热点。    
``` 
   在MySQL中，InnoDB数据表的辅助索引设计我们通常遵循几个原则：
``` 
    1.多用复合索引，少用多个独立索引(即使现在MySQL有index merge 特性)；
    2.一些基数（Cardinality）太小（比如说，该列的唯一值总数少于255）的列就不要创建独立索引了；
    3.使用辅助索引时，尽量 使用覆盖索引的特性(索引要减少使用select * )，减少回表次数。
```    

  


**高性能索引策略**
1. 独立的列：索引的列不能是表达式的一部分，也不能是函数的参数。
```
    select act_id from actor where actor_id+1 = 5;
    select ... from where TO_DAYS(current_date)-TO_DAYS(date_col) <= 10
```
 
2. 前缀索引：当一个列的长度很大的时候，它是不合适建索引的，因为这样的索引会很大；对此一般有两个策略，第一个就是前面提到的 伪HASH索引，
   在就是前缀索引: 索引一个字段的开始的部分字段，这样可以大大节约索引空间，和提高索引效率。
   问题：前缀索引长度的选择？
   
```
   mysql> select count(distinct left(city,3))/count(*) as sel3, count(distinct left(city,4))/count(*) as sel4,
       ->        count(distinct left(city,5))/count(*) as sel5, count(distinct left(city,6))/count(*) as sel6,   
       ->        count(distinct left(city,7))/count(*) as sel7, count(distinct left(city,8))/count(*) as sel8
       ->        from city; 
   +--------+--------+--------+--------+--------+--------+
   | sel3   | sel4   | sel5   | sel6   | sel7   | sel8   |
   +--------+--------+--------+--------+--------+--------+
   | 0.7633 | 0.9383 | 0.9750 | 0.9900 | 0.9933 | 0.9933 |
   +--------+--------+--------+--------+--------+--------+
   1 row in set (0.00 sec)
```
3. 多列索引(组合索引)： 在多个列上建立多个索引的时候，对我们的查询有时候并没有太大的提升，在MySQL5.0之前，我们基本认为MySQL只走一个索引，
   在MySQL5.0之后引入了 索引合并(index merge)的策略， 虽然在一定程度上提升了效率，但是在大部分情况 我们使用explain 的时候看到有index merge
   恰恰表明了我们的索引建立的不合理。
   
4. 覆盖索引(三星索引)： 当MySQL走索引检索时查询到的key值的value 就直接包含了锁查询的字段，不需要回表查询。那么就可以称该索引就是覆盖索引。
   覆盖索引可以极大提高查询效率，我们使用查询时，尽量优化到覆盖索引。      
   
``` 
   mysql> explain select store_id ,film_id from inventory \G;
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: inventory
      partitions: NULL
            type: index
   possible_keys: NULL
             key: idx_store_id_film_id
         key_len: 3
             ref: NULL
            rows: 4581
        filtered: 100.00
           Extra: Using index
   1 row in set, 1 warning (0.00 sec)
   
   ERROR: 
   No query specified
   
   上面的查询就用到了 覆盖索引。
   
```   


### Index Merge Optimization(索引合并优化)：

```
MySQL优化器如果发现可以使用多个索引查找后的交集/并集定位数据，那么MySQL优化器就会尝试index merge这类访问方式。index merge主要分为两大类，多个索引交集访问(intersections)，多个索引并集访问，当然这两类还可以组合出更为复杂的方式，例如多个交集后做并集。
```
例如下面的查询可能会使用索引合并：
> SELECT * FROM tbl_name WHERE key1 = 10 OR key2 = 20;

> SELECT * FROM tbl_name WHERE (key1 = 10 OR key2 = 20) AND non_key = 30;

> SELECT * FROM t1, t2  WHERE (t1.key1 IN (1,2) OR t1.key2 LIKE 'value%')  AND t2.key1 = t1.some_col;

> SELECT * FROM t1, t2 WHERE t1.key1 = 1 AND (t2.key1 = t1.some_col OR t2.key2 = t1.some_col2);

#### **Note:**
>   索引合并有以下限制：

```
 1.在复杂的Where查询中如果有比较深(deep)的and/or 查询，那么优化器不会选择索引合并;
        (x AND y) OR z => (x OR z) AND (y OR z)
        (x OR y) AND z => (x AND z) OR (y AND z)
2. full-text indexes优化器不会选择索引合并
3. MySQL在5.6.7之前，使用index merge有一个重要的前提条件：没有range可以使用。这个限制降低了MySQL index merge可以使用的场景。理想状态是同时评估成本
后然后做出选择。因为这个限制，就有了下面这个已知的bad case
         SELECT * FROM t1 WHERE (goodkey1 < 10 OR goodkey2 < 20) AND badkey < 30;
优化器可以选择使用goodkey1和goodkey2做index merge，也可以使用badkey做range。因为上面的原则，无论goodkey1和goodkey2的选择度如何，MySQL都只会考虑range，
而不会使用index merge的访问方式。
（[5.6.7版本针对此有修复](http://jorgenloland.blogspot.jp/2012/10/index-merge-annoyances-fixed-in-mysql-56.html))
```

##### 索引合并算法：
1.    The Index Merge Intersection Access Algorithm
2.    The Index Merge Union Access Algorithm
3.    The Index Merge Sort-Union Access Algorithm
  
**Intersection**

  简单而言，index intersect merge就是多个索引条件扫描得到的结果进行交集运算。显然在多个索引提交之间是 AND 运算时，才会出现 index intersect merge. 下面两种where条件或者它们的组合时会进行 index intersect merge:
```
1) 条件使用到复合索引中的所有字段或者左前缀字段(对单字段索引也适用)
key_part1=const1 AND key_part2=const2 ... AND key_partN=constN

2) InnoDB表主键上的任何范围条件(Any range condition over a primary key of an InnoDB table)
例子：
SELECT * FROM innodb_table WHERE primary_key < 10 AND key_col1=20;
SELECT * FROM tbl_name WHERE (key1_part1=1 AND key1_part2=2) AND key2=2;
```
Intersection特点：
```
1.运行方式：多个索引同时扫描，然后结果取交集
2.索引覆盖扫描，无需回表
3.索引不能覆盖，则对满足条件的再进行回表

```

**Union**
简单而言，index uion merge就是多个索引条件扫描，对得到的结果进行并集运算，显然是多个条件之间进行的是 OR 运算。

下面几种类型的 where 条件，以及他们的组合可能会使用到 index union merge算法：
```
1) 条件使用到复合索引中的所有字段或者左前缀字段(对单字段索引也适用)
2) 主键上的任何范围条件
3) 任何符合 index intersect merge 的where条件；
```
例子：

```
SELECT * FROM t1 WHERE key1=1 OR key2=2 OR key3=3;
SELECT * FROM innodb_table WHERE (key1=1 AND key2=2) OR (key3='foo' AND key4='bar') AND key5=5;
第一个例子，就是三个 单字段索引 进行 OR 运算，所以他们可能会使用 index union merge算法。
第二个例子，复杂一点。(key1=1 AND key2=2) 是符合 index intersect merge; (key3='foo' AND key4='bar') AND key5=5 也是符合index intersect merge，所以 二者之间进行 OR 运算，自然可能会使用 index union merge算法
```

**Sort-Union**





###  问题：是索引合并还是组合索引(多个独立索引还是组合索引)？参见实践部分

### Index Condition Pushdown(ICP,索引条件下推)

> ICP（index condition pushdown）是mysql利用索引（二级索引）元组和筛字段在索引中的where条件从表中提取数据记录的一种优化操作。

> ICP的思想是：存储引擎在访问索引的时候检查筛选字段在索引中的where条件（pushed index condition，推送的索引条件），
   如果索引元组中的数据不满足推送的索引条件，那么就过滤掉该条数据记录。ICP（优化器）尽可能的把index condition的处理从server层下推到storage engine层。
   storage engine使用索引过过滤不相关的数据，仅返回符合index condition条件的数据给server层。也是说数据过滤尽可能在storage engine层进行，而不是返回所有数据给server层，
   然后后再根据where条件进行过滤。使用ICP（mysql 5.6版本以前）和没有使用ICP的数据访问和提取过程如下：

优化器没有使用ICP时，数据访问和提取的过程如下：
    
    1. 当storage engine读取下一行时，首先读取索引元组（index tuple），然后使用索引元组在基表中（base table）定位和读取整行数据。
    2. sever层评估where条件，如果该行数据满足where条件则使用，否则丢弃。
    3. 执行1），直到最后一行数据。
    
![image](http://mdba.cn/wp-content/uploads/2014/01/index-access-2phases.png)


优化器使用ICP时，server层将会把能够通过使用索引进行评估的where条件下推到storage engine层。数据访问和提取过程如下：

    1. storage engine从索引中读取下一条索引元组。
    2. storage engine使用索引元组评估下推的索引条件。如果没有满足wehere条件，storage engine将会处理下一条索引元组（回到上一步）。只有当索引元组满足下推的索引条件的时候，才会继续去基表中读取数据。
    3. 如果满足下推的索引条件，storage engine通过索引元组定位基表的行和读取整行数据并返回给server层。
    4. server层评估没有被下推到storage engine层的where条件，如果该行数据满足where条件则使用，否则丢弃。

![image](http://mdba.cn/wp-content/uploads/2014/01/index-access-with-icp.png)

在mysql5.6以前，还没有采用ICP这种查询优化，where查询条件中的索引条件在某些情况下没有充分利用索引过滤数据。假设一个组合索引（多列索引）K包含（c1,c2,…,cn）n个列，如果在c1上存在范围扫描的where条件，
那么剩余的c2,…,cn这n-1个上索引都无法用来提取和过滤数据（不管不管是唯一查找还是范围查找），索引记录没有被充分利用。即组合索引前面字段上存在范围查询，那么后面的部分的索引将不能被使用，
因为后面部分的索引数据是无序。比如，索引key（a，b）中的元组数据为(0,100)、(1,50)、（1，100） ，where查询条件为 a < 2 and b = 100。由于b上得索引数据并不是连续区间，
因为在读取（1，50）之后不再会读取（1，100），mysql优化器在执行索引区间扫描之后也不再扫描组合索引其后面的部分。


### 覆盖索引的延迟关联(deferred join)
    
  尽量的使用 覆盖索引，但现实缺很难做到。
```
  mysql> explain select * from user_1 where first_name='CzOBj' and last_name like '%G%';
  +----+-------------+--------+------------+------+---------------+---------+---------+-------+------+----------+-----------------------+
  | id | select_type | table  | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra                 |
  +----+-------------+--------+------------+------+---------------+---------+---------+-------+------+----------+-----------------------+
  |  1 | SIMPLE      | user_1 | NULL       | ref  | IDX_F_L       | IDX_F_L | 194     | const |    1 |    11.11 | Using index condition |
  +----+-------------+--------+------------+------+---------------+---------+---------+-------+------+----------+-----------------------+
  1 row in set, 1 warning (0.00 sec)
  
 
  mysql> explain select * from user_1 as t0 join (select id from user_1 where first_name='CzOBj' and last_name like '%G%' )  as t1 on (t1.id = t0.id);
  +----+-------------+--------+------------+--------+-----------------+---------+---------+----------------------+------+----------+--------------------------+
  | id | select_type | table  | partitions | type   | possible_keys   | key     | key_len | ref                  | rows | filtered | Extra                    |
  +----+-------------+--------+------------+--------+-----------------+---------+---------+----------------------+------+----------+--------------------------+
  |  1 | SIMPLE      | user_1 | NULL       | ref    | PRIMARY,IDX_F_L | IDX_F_L | 194     | const                |    1 |    11.11 | Using where; Using index |
  |  1 | SIMPLE      | t0     | NULL       | eq_ref | PRIMARY         | PRIMARY | 8       | my_indexed.user_1.id |    1 |   100.00 | NULL                     |
  +----+-------------+--------+------------+--------+-----------------+---------+---------+----------------------+------+----------+--------------------------+
  2 rows in set, 1 warning (0.00 sec)
  
  

  
```
  






# 实践篇

### 1.索引、提交频率对InnoDB表写入速度的影响

##### 1.1 关于索引对写入速度的影响：
>     a、如果有自增列做主键，相对完全没索引的情况，写入速度约提升 3.11%；
>     b、如果有自增列做主键，并且二级索引，相对完全没索引的情况，写入速度约降低 27.37%；

结论：InnoDB表最好总是有一个自增列做主键

##### 1.2 关于提交频率对写入速度的影响（以表中只有自增列做主键的场景，一次写入数据30万行数据为例）：
>     a、等待全部数据写入完成后，最后再执行commit提交的效率最高；
>     b、每10万行提交一次，相对一次性提交，约慢了1.17%；
>     c、每1万行提交一次，相对一次性提交，约慢了3.01%；
>     d、每1千行提交一次，相对一次性提交，约慢了23.38%；
>     e、每100行提交一次，相对一次性提交，约慢了24.44%；
>     f、每10行提交一次，相对一次性提交，约慢了92.78%；
>     g、每行提交一次，相对一次性提交，约慢了546.78%，也就是慢了5倍；

结论：最好是等待所有事务结束后再批量提交，而不是每执行完一个SQL就提交一次。


---
**Caution**：
```
这个建议并不是绝对成立的，要看具体的场景。如果是一个高并发的在线业务，
就需要尽快提交事务，避免锁范围被扩大。但如果是在非高并发的业务场景，尤其
是做数据批量导入的场景下，就建议采用批量提交的方式
```

***实践测试代码***：
```
-- 创建数据库表
DROP TABLE IF EXISTS `indexed_test`;
CREATE TABLE `indexed_test` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `code_int_1` int(11) unsigned zerofill NOT NULL,
  `code_int_2` int(11) unsigned zerofill NOT NULL,
  `modified` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  `code_str_1` varchar(200) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


-- 创建随机字符串函数
DELIMITER //
CREATE DEFINER=`root`@`localhost` FUNCTION `rand_string`(n INT) RETURNS varchar(255) CHARSET latin1
BEGIN
	DECLARE chars_str varchar(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
	DECLARE return_str varchar(255) DEFAULT '';
DECLARE i INT DEFAULT 0;
WHILE i < n DO
        SET return_str = concat(return_str,substring(chars_str , FLOOR(1 + RAND()*62 ),1));
        SET i = i +1;
    END WHILE;
    RETURN return_str;
END;//


-- 创建批量测试存储过程
DELIMITER %%%
DROP PROCEDURE IF EXISTS insert_indexed_test ;
CREATE PROCEDURE insert_indexed_test (IN rownum INT, IN commitcount INT)
BEGIN
DECLARE i INT DEFAULT 0 ;
SET AUTOCOMMIT = 0 ;
WHILE i < rownum DO
	INSERT INTO indexed_test (
		code_int_1,
		code_int_2,
		code_str_1
	) values (
		FLOOR(RAND() * rownum),
		FLOOR(RAND() * rownum),
		rand_string(180)
	) ;
SET i = i + 1 ;
IF (commitcount > 0) AND (i % commitcount = 0) THEN
	COMMIT ; 
	END IF ;
	END	WHILE ; 
COMMIT ;
	END ;%%%

```



 
### 2.是索引合并还是组合索引(多个独立索引还是组合索引)？

    Q:当我们查询字段涉及到 code1=XX and code2=YY的时候我们是建立连个独立二级索引还是创建一个 combined的组合索引：
    A:对支持索引合并的mysql版本来说大部分情况是组合索引更优，但是答案不是绝对的，着主要和索引的 区分度(selectivity) and 数据的关联性(correlation)有关。
    
   ***实践测试代码***

```
1. 准备测试数据user_1 表共 ：6447156条
2. 各个索引的区分度如下：
   IDX_ZIP: 
   select count(distinct(zip_code))/count(zip_code) from user_1 值为:0.0100
   IDX_AGE:
   select count(distinct(age))/count(age) from user_1 值为：0.1385
   IDX_CODE:
   select count(distinct(code))/count(code) from user_1 值为：1.0000
3.各项测试数据：
   
  1) OR 使用 索引合并 
mysql> select * from user_1 ignore index(IDX_AGE_ZIP)  where   zip_code=797727 or age =704312;
+---------+----------+--------+------------+-----------+---------------------+----------------------+
| id      | zip_code | age    | first_name | last_name | modified            | code                 |
+---------+----------+--------+------------+-----------+---------------------+----------------------+
|  425308 |   123825 | 704312 | wSuxO      | C5nei7    | 2017-05-16 11:05:19 | -2958036916730861468 |
.....................................................................................................
.....................................................................................................
| 7480737 |   797727 | 531096 | YdWYf      | cDuXvY    | 2017-05-16 14:33:59 | -6967670584370978578 |
+---------+----------+--------+------------+-----------+---------------------+----------------------+
107 rows in set (0.01 sec)

mysql> desc select * from user_1 ignore index(IDX_AGE_ZIP)  where   zip_code=797727 or age =704312;
+----+-------------+--------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-------------------------------------------+
| id | select_type | table  | partitions | type        | possible_keys   | key             | key_len | ref  | rows | filtered | Extra                                     |
+----+-------------+--------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-------------------------------------------+
|  1 | SIMPLE      | user_1 | NULL       | index_merge | IDX_ZIP,IDX_AGE | IDX_ZIP,IDX_AGE | 8,4     | NULL |  108 |   100.00 | Using union(IDX_ZIP,IDX_AGE); Using where |
+----+-------------+--------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-------------------------------------------+
1 row in set, 1 warning (0.00 sec)

  2) OR 使用 组合索引
  mysql> select * from user_1 use index(IDX_AGE_ZIP)  where   zip_code=797727 or age =704312;
+---------+----------+--------+------------+-----------+---------------------+----------------------+
| id      | zip_code | age    | first_name | last_name | modified            | code                 |
+---------+----------+--------+------------+-----------+---------------------+----------------------+
|  425308 |   123825 | 704312 | wSuxO      | C5nei7    | 2017-05-16 11:05:19 | -2958036916730861468 |
.....................................................................................................
.....................................................................................................
| 7480737 |   797727 | 531096 | YdWYf      | cDuXvY    | 2017-05-16 14:33:59 | -6967670584370978578 |
+---------+----------+--------+------------+-----------+---------------------+----------------------+
107 rows in set (3.30 sec)


mysql>  desc select * from user_1 use index(IDX_AGE_ZIP)  where   zip_code=797727 or age =704312;
+----+-------------+--------+------------+------+---------------+------+---------+------+---------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | user_1 | NULL       | ALL  | IDX_AGE_ZIP   | NULL | NULL    | NULL | 6477156 |     0.00 | Using where |
+----+-------------+--------+------------+------+---------------+------+---------+------+---------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

  3) AND 使用 索引合并 

mysql> select * from user_1 ignore index(IDX_AGE_ZIP)  where   zip_code=797727 and age =704312;
+---------+----------+--------+------------+-----------+---------------------+---------------------+
| id      | zip_code | age    | first_name | last_name | modified            | code                |
+---------+----------+--------+------------+-----------+---------------------+---------------------+
| 7480661 |   797727 | 704312 | dQYGR      | xpovix    | 2017-05-16 14:33:59 | 3943646070993448765 |
+---------+----------+--------+------------+-----------+---------------------+---------------------+
1 row in set (0.00 sec)

mysql> desc select * from user_1 ignore index(IDX_AGE_ZIP)  where   zip_code=797727 and age =704312;
+----+-------------+--------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-----------------------------------------------+
| id | select_type | table  | partitions | type        | possible_keys   | key             | key_len | ref  | rows | filtered | Extra                                         |
+----+-------------+--------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-----------------------------------------------+
|  1 | SIMPLE      | user_1 | NULL       | index_merge | IDX_ZIP,IDX_AGE | IDX_AGE,IDX_ZIP | 4,8     | NULL |    1 |   100.00 | Using intersect(IDX_AGE,IDX_ZIP); Using where |
+----+-------------+--------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-----------------------------------------------+
1 row in set, 1 warning (0.00 sec)



4) AND 使用 组合索引 

mysql> select * from user_1 use index(IDX_AGE_ZIP)  where   zip_code=797727 and age =704312;
+---------+----------+--------+------------+-----------+---------------------+---------------------+
| id      | zip_code | age    | first_name | last_name | modified            | code                |
+---------+----------+--------+------------+-----------+---------------------+---------------------+
| 7480661 |   797727 | 704312 | dQYGR      | xpovix    | 2017-05-16 14:33:59 | 3943646070993448765 |
+---------+----------+--------+------------+-----------+---------------------+---------------------+
1 row in set (0.00 sec)


mysql> desc select * from user_1 use index(IDX_AGE_ZIP)  where   zip_code=797727 and age =704312;
+----+-------------+--------+------------+------+---------------+-------------+---------+-------------+------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key         | key_len | ref         | rows | filtered | Extra |
+----+-------------+--------+------------+------+---------------+-------------+---------+-------------+------+----------+-------+
|  1 | SIMPLE      | user_1 | NULL       | ref  | IDX_AGE_ZIP   | IDX_AGE_ZIP | 12      | const,const |    1 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+-------------+---------+-------------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

```

   
  结论：
    
```
使用组合索引在 column 使用 AND 情况下明显的优于单个列的独立索引(索引合并)。
但是行之间使用 OR 索引合并优于组合索引(特别是selectivity比较优的情况)
```

   
 ### 3.大数据集的分页查询优化



 1. 只允许查询前N页
        ![image](https://github.com/qhwj2006/mysql-doc/raw/master/res/111.png)
     
   ```  
    优点：实现简单暴力；
  缺点：产品不会让这么干！！！
```
2. 不许跳页查询(只能上一页，下一页)
    ![image](https://github.com/qhwj2006/mysql-doc/raw/master/res/123.png)
    实现：
    每次下一页翻页的时候，将上一页的最后一条记录的Id回传
```
    sql：  select id, name, address, phone FROM customers WHERE id > 990 ORDER BY id LIMIT 10;
```
    优点：实现简单暴力,用户体验稍好；
    缺点：产品不会让这么干！！！

3.正常分页先查Id 在 in 查询
 
```
SELECT id, name, address, phone FROM customers INNER JOIN ( SELECT id FROM customers ORDER BY name LIMIT 10 OFFSET 990) AS my_results USING(id);
```


 ### 4.select count(*) from innodb table优化
    
  
    Q: 如果某种表要依据status字段统计数量，如：select status,count(*) from user group by status;  如status字段索引的区分度比较小，不适宜索引？ 
    A:强制走主键或辅助索引(http://mysql.taobao.org/monthly/2016/06/10/)? 亲测效果都不佳；
    

```
DELIMITER //

CREATE TRIGGER insert_user_tigger AFTER INSERT ON user
FOR EACH ROW UPDATE user_count SET total = total + 1 WHERE status = NEW.status
//
CREATE TRIGGER delete_user_tigger AFTER DELETE ON user
FOR EACH ROW UPDATE user_count SET total = total - 1 WHERE status = OLD.status;
//
CREATE TRIGGER update_user_tigger AFTER UPDATE ON user
FOR EACH ROW
BEGIN
    IF (OLD.status <> NEW.status)
    THEN
        UPDATE user_count SET total = total + IF(status = NEW.status, 1, -1) WHERE status IN (OLD.status, NEW.status);
    END IF;
END
//

```
   
    
### 5. 关于数据库NULL值

```
要尽量避免 NULL 
要尽可能地把字段定义为 NOT NULL。即使应用程序无须保存 NULL（没有值），也有许多表包含了可空列（Nullable Column）,这仅仅是因为它为默认选项。除非真的要保存 NULL，否则就把列定义为 NOT NULL。 

MySQL难以优化引用了可空列的查询，它会使索引、索引统计和值更加复杂。可空列需要更多的储存空间，还需要在MySQL内部进行特殊处理。当可空列被索引的时候，每条记录都需要一个额外的字节，还可能导致 MyISAM 中固定大小的索引(例如一个整数列上的索引)变成可变大小的索引。 

即使要在表中储存「没有值」的字段，还是有可能不使用 NULL 的。考虑使用 0、特殊值或空字符串来代替它。 

把 NULL 列改为 NOT NULL 带来的性能提升很小，所以除非确定它引入了问题，否则就不要把它当作优先的优化措施。然后，如果计划对列进行索引，就要尽量避免把它设置为可空。 
```


   
   
   
   
 # 4.应用篇
  
  **程序员应该关心的基础tips**
  #### 1. 关于Schema设计规范及SQL使用建议
```
    1. 所有的InnoDB表都设计一个无业务用途的自增列做主键，对于绝大多数场景都是如此。
    2. 字段长度满足需求前提下，尽可能选择长度小的。此外，字段属性尽量都加上NOT NULL约束，可一定程度提高性能；
    3. 尽可能不使用TEXT/BLOB类型，确实需要的话，建议拆分到子表中，不要和主表放在一起，避免SELECT * 的时候读性能太差。
    4. 读取数据时，只选取所需要的列，不要每次都SELECT *，避免产生严重的随机读问题，尤其是读到一些TEXT/BLOB列；
    5. 对一个VARCHAR(N)列创建索引时，通常取其50%（甚至更小）左右长度创建前缀索引就足以满足80%以上的查询需求了，没必要创建整列的全长度索引；
    6. 多用复合索引，少用多个独立索引，尤其是一些基数（Cardinality）太小（比如说，该列的唯一值总数少于255）的列就不要创建独立索引了；
    7. 类似分页功能的SQL，建议先用主键关联，然后返回结果集，效率会高很多；
```  


  #### 2. 店长线上业务BAD SQL 
``` 
 SELECT COUNT(1) FROM trade_info199 AS t  WHERE seller_nick = '青田食品专营店'  AND  buyer_nick IN 
( '雨夜_飘飘漫漫' , '~Yw5vwaQN0qHY0hyZcosgm0Wxx/3pjmualFybsmaZNMo=~1~' )AND (merge_tid = 0 OR merge_tid = tid );


 SELECT COUNT(1) FROM trade_info199 AS t  WHERE seller_nick = '青田食品专营店'  AND (merge_tid = 0 OR merge_tid = tid )
 
 优化 how ?
``` 


  #### 3. explain sql 关注点
  
总的来说，我们只需要关注结果中的几列：

列名   | 备注
---|---
type   | 本次查询表联接类型，从这里可以看到本次查询大概的效率
key  | 最终选择的索引，如果没有索引的话，本次查询效率通常很差
key_len  | 本次查询用于结果过滤的索引实际长度
rows   | 预计需要扫描的记录数，预计需要扫描的记录数越小越好
Extra   | 额外附加信息，主要确认是否出现 Using filesort、Using temporary 这两种情况
 

首先看下 type 有几种结果，分别表示什么意思：


类型   | 备注
---|---
ALL   | 执行full table scan，这事最差的一种方式
index   | 执行full index scan，并且可以通过索引完成结果扫描并且直接从索引中取的想要的结果数据，也就是可以避免回表，比ALL略好，因为索引文件通常比全部数据要来的小
range   | 利用索引进行范围查询，比index略好
index_subquery   | 子查询中可以用到索引
unique_subquery   | 子查询中可以用到唯一索引，效率比 index_subquery 更高些
index_merge   | 可以利用index merge特性用到多个索引，提高查询效率
ref_or_null   | 表连接类型是ref，但进行扫描的索引列中可能包含NULL值
fulltext   | 全文检索
ref   | 基于索引的等值查询，或者表间等值连接
eq_ref   | 表连接时基于主键或非NULL的唯一索引完成扫描，比ref略好
const   | 基于主键或唯一索引唯一值查询，最多返回一条结果，比eq_ref略好
system   | 查询对象表只有一行数据，这是最好的情

上面几种情况，从上到下一次是最差到最好。

再来看下Extra列中需要注意出现的几种情况：

关键字  | 备注
---|---
Using filesort | 将用外部排序而不是按照索引顺序排列结果，数据较少时从内存排序，否则需要在磁盘完成排序，代价非常高，需要添加合适的索引
Using temporary |  需要创建一个临时表来存储结果，这通常发生在对没有索引的列进行GROUP BY时，或者ORDER BY里的列不都在索引里，需要添加合适的索引
Using index	 |  表示MySQL使用覆盖索引避免全表扫描，不需要再到表中进行二次查找数据，这是比较好的结果之一。注意不要和type中的index类型混淆
Using where | 	通常是进行了全表/全索引扫描后再用WHERE子句完成结果过滤，需要添加合适的索引
Impossible WHERE | 对Where子句判断的结果总是false而不能选择任何数据，例如where 1=0，无需过多关注
Select tables optimized away | 使用某些聚合函数来访问存在索引的某个字段时，优化器会通过索引直接一次定位到所需要的数据行完成整个查询，例如MIN()\MAX()，这种也是比较好的结果之一
	
 
   
```
  
```

   
   
    


   







**reference:**
1. https://dzone.com/articles/every-programmer-should-know
2. https://www.cs.cornell.edu/projects/ladis2009/talks/dean-keynote-ladis2009.pdf
3. http://mdba.cn/2014/01/21/index-condition-pushdownicp%E7%B4%A2%E5%BC%95%E6%9D%A1%E4%BB%B6%E4%B8%8B%E6%8E%A8/
4. https://www.percona.com/blog/2009/09/19/multi-column-indexes-vs-index-merge/







# 1. 一条sql查询是如何利用索引的（究竟ICP push down了什么）？

### 理论基础

##### -     索引片及匹配列
        在sql真正被执行之前，数据库系统首先要确定的是如何访问数据。这包含：应该使用哪一个索引，索引的访问方式如何 等。
        
        查询成本很大程度取决于索引片的厚度，即谓词表达式确定的值域范围。
        如果索引片越厚，需要顺序扫描的索引页就越多，需要处理的索引记录就越多，而最大的开销还是来自于增加对表的同步读操作。
        如果索引片越窄，佳慧显著减少索引片访问的开销，但是主要成本还是节省在对表的同步读上。
        mysql优化器使用哪一个索引是基于成本考虑的，即优先使用索引片窄的索引。

##### -     索引过滤及过滤列
        
        流程逻辑：
            1. 在 where 语句中，该列是否至少拥有一个足够简单的谓词与之对应？
                如果有：那么这是一个匹配列。
                如果没有：那么这个列以及后面的索引列都是非匹配列。
            2. 如果该谓词是一个范围谓词，那么剩余的索引列都是非匹配列。
            3. 对于最后一个匹配列之后的索引列，如果有一个足够简单的谓词与其对应，那么该列为过滤列。
            
            例：有 IDX_A_B_C_D  查询 where A=:a and B>=:b and C=:c
            A列：出现了一个等值谓词，列A是匹配列，将被用于定于索引片。
            B列：也简单到足以成为匹配列，所以它也将被用于定义索引片。
            C列：由于B列是一个范围谓词，所以它不能参与匹配过程(即它不能定义索引片)。不过第3条指出，列C能够避免不必要的表访问，因为它可以参与索引片的过滤过程。
                 从效果上而言，列C的作用与列A和列B几乎同等重要，只是列C无法参与索引片的定义，增加了索引片的厚度而已。
            
        
        这里的困难是什么？
            足够简单的标准是什么？不同的DBMS的定义标准是不一致的。    
         

   


### MySql的索引片（where 条件的提取）
-     Index Key
        MySQL是用来确定扫描的数据范围，实际就是可以利用到的MySQL索引部分，体现在Key Length
-     Index Filter (ICP 过程)
        MySQL用来确定哪些数据是可以用索引去过滤，在启用ICP后，可以用上索引的部分    
-     Table Filter
        MySQL无法用索引过滤，回表取回行数据后，到server层进行数据过滤    



---


-     Index Key
        Index Key是用来确定MySQL的一个扫描范围，分为上边界和下边界(索引片)。
        MySQL利用=、>=、> 来确定下边界（first key），利用最左原则，首先判断第一个索引键值在where条件中是否存在，如果存在，则判断比较符号，
        如果为(=,>=)中的一种，加入下边界的界定，然后继续判断下一个索引键，如果存在且是(>)，则将该键值加入到下边界的界定，停止匹配下一个索引键；
        如果不存在，直接停止下边界匹配
                exp:
                    idx_c1_c2_c3(c1,c2,c3)
                    where c1>=1 and c2>2 and c3=1
                        –> first key (c1,c2)
                        –> c1为 ‘>=’ ，加入下边界界定，继续匹配下一个
                        –> c2 为 ‘>’，加入下边界界定，停止匹配
  
        上边界（last key）和下边界（first key）类似，首先判断是否是否是(=,<=)中的一种，如果是，加入界定，
        继续下一个索引键值匹配，如果是(<)，加入界定，停止匹配。
                exp:
                    idx_c1_c2_c3(c1,c2,c3)
                    where c1<=1 and c2=2 and c3<3
                        –> last key (c1,c2,c3)
                        –> c1为 ‘<=’，加入上边界界定，继续匹配下一个
                        –> c2为 ‘=’加入上边界界定，继续匹配下一个
                        –> c3 为 ‘<‘，加入上边界界定，停止匹配


-     Index Filter
        字面理解就是可以用索引去过滤。也就是字段在索引键值中，但是无法用去确定Index Key的部分。
                exp:
                    idex_c1_c2_c3
                    where c1>=1 and c2<=2 and c3 =1
                    index key –> c1
                    index filter–> c2 c3
        这里为什么index filter 只是c1呢？因为c2 是用来确定上边界的，但是上边界的c1没有出现(<=,=)，而下边界中，c1是>=,c2没有出现，
        因此index key 只有c1字段。c2,c3 都出现在索引中，被当做index filter。  

-     Table Filter
        无法利用索引完成过滤，就只能用table filter。此时引擎层会将行数据返回到server层，然后server层进行table filter。



# 2. MySql数据库锁(InnoDB)

### 理论基础

#####      MySQL InnoDB MVCC
     MySQL InnoDB存储引擎，实现的是基于多版本的并发控制协议——MVCC (Multi-Version Concurrency Control)
    (与MVCC相对的，是基于锁的并发控制，Lock-Based Concurrency Control)。MVCC最大的好处：读不加锁，读写不冲突。
    在读多写少的OLTP应用中，读写不冲突是非常重要的，极大的增加了系统的并发性能，这也是为什么现阶段，几乎所有的RDBMS，都支持了MVCC。

    在MVCC并发控制中，读操作可以分成两类：快照读 (snapshot read)与当前读 (current read)。快照读，读取的是记录的可见版本 (有可能是历史版本)，不用加锁。
    当前读，读取的是记录的最新版本，并且，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录。 

    快照读(Consistent Nonlocking Reads)：
            简单的select操作，属于快照读，不加锁。
            select * from table where ?;
            
    当前读(Locking Reads)：
            特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。
            select * from table where ? lock in share mode;
            select * from table where ? for update;
            insert into table values (…);
            update table set ? where ?;
            delete from table where ?;
            当前读，读取记录的最新版本。并且，读取之后，还需要保证其他并发事务不能修改当前记录，对读取记录加锁。
            其中，除了第一条语句，对读取记录加S锁 (共享锁)外，其他的操作，都加的是X锁 (排它锁)。
    
    

    
    
    更多参见：
            https://dev.mysql.com/doc/refman/5.6/en/innodb-consistent-read.html
            https://dev.mysql.com/doc/refman/5.6/en/innodb-locking-reads.html
            
    
#####      MySQL InnoDB 事务隔离级别
    Read Uncommited
        一个事务可以读取另一个未提交事务的数据录,存在脏读问题。此隔离级别，不会使用，忽略。

    Read Committed (RC)
        若有事务对数据进行更新（UPDATE）操作时，读操作事务要等待这个更新操作事务提交后才能读取数据，可以解决脏读问题。
        针对快照度，RC隔离级别每个事物读取到的是最新的。
        针对当前读，RC隔离级别保证对读取到的记录加锁 (记录锁)，存在幻读现象。

    Repeatable Read (RR)
        针对快照度，RR隔离级别每个事物读取到是最开始的timepoint的状态。
        针对当前读，RR隔离级别保证对读取到的记录加锁 (记录锁)，同时保证对读取的范围加锁，新的满足查询条件的记录不能够插入 (间隙锁)，不存在幻读现象。

    Serializable
        从MVCC并发控制退化为基于锁的并发控制。不区别快照读与当前读，所有的读操作均为当前读，读加读锁 (S锁)，写加写锁 (X锁)。
        
 ![image](https://raw.githubusercontent.com/qhwj2006/mysql-doc/master/res/snaptshotread.png)
        
       
#####      一条简单SQL的加锁实现分析 

    SQL1：select * from t1 where id = 10;
    SQL2：delete from t1 where id = 10;
    
    SQL1：不加锁。因为MySQL是使用多版本并发控制的，读不加锁。
    SQL2：要依据 事务的隔离级别 && id 是否是主键 索引键 考虑。
    
    
    组合一：id主键 + RC 
       id主键并且是RC 那么只在id=10上加 X 锁。
     
    组合二：id唯一索引 + RC
       先在唯一键索引上 X 锁，然后在id=10 对应的数据列上加 X 锁。
    
    组合三：id非唯一索引+RC
       先在id索引列 等于10 的索引列都加 X锁，然后在对应的主数据列上索引。
       
    组合四：id无索引+RC
       由于id列上没有索引，因此只能走聚簇索引，进行全部扫描。从图中可以看到，满足删除条件的记录有两条，但是，聚簇索引上所有的记录，都被加上了X锁。
       注意：
        为了效率考量，MySQL做了优化，对于不满足条件的记录，会在判断后放锁，最终持有的，是满足条件的记录上的锁，
       但是不满足条件的记录上的加锁/放锁动作不会省略。同时，优化也违背了2PL的约束。
       
    组合五：id主键+RR
        与组合一 一致
    
    组合六：id唯一索引+RR
        与组合二 一致    
    
    组合七：id非唯一索引+RR
        Repeatable Read隔离级别下，id列上有一个非唯一索引，对应SQL：delete from t1 where id = 10; 首先，通过id索引定位到第一条满足查询条件的记录，加记录上的X锁，加GAP上的GAP锁，然后加主键聚簇索引上的记录X锁，然后返回；然后读取下一条，重复进行。直至进行到第一条不满足条件的记录[11,f]，此时，不需要加记录X锁，但是仍旧需要加GAP锁，最后返回结束。
    
 ![image](http://photo.yupoo.com/hedengcheng/DnJ6R7wu/medish.jpg)  
 
 
        组合八：id无索引+RR
            Repeatable Read隔离级别下的最后一种情况，id列上没有索引。此时SQL：delete from t1 where id = 10; 没有其他的路径可以选择，只能进行全表扫描。最终的加锁情况，如下图所示：

![image](http://photo.yupoo.com/hedengcheng/DnJ6Rf3q/medish.jpg)


    死锁原理:
        死锁的发生与否，并不在于事务中有多少条SQL语句，死锁的关键在于：两个(或以上)的Session加锁的顺序不一致。
        而使用本文上面提到的，分析MySQL每条SQL语句的加锁规则，分析出每条语句的加锁顺序，
        然后检查多个并发SQL间是否存在以相反的顺序加锁的情况，就可以分析出各种潜在的死锁情况，也可以分析出线上死锁发生的原因。
  



   
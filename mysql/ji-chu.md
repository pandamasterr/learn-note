# mysql 查询执行的基础

![](/assets/mysql查询执行的基础.png)




### 语法解析器和预处理

** 解析器 **处理语法和解析查询, 生成一课对应的 "解析树". ---> mysql 语法层面 .

** 预处理器 ** 进一步检查解析树的合法性. 如: 数据表和数据列 是否存在, 别名是否有歧义等, 还会进行权限认证.

### 查询优化器

将 解析树 转换为 执行计划.
    
    一条查询可以有多种执行方式, 都会返回同样的结果. 优化器的作用就是找到最好的执行计划.
    (当然也有可能做出最差的决定)
    
mysql 是基于成本的优化器, 其尝试预测一个查询使用某种执行计划是的成本, 并选择其中最小的那个.
    
导致 mysql 优化器选择错误的执行计划的原因.

    * 统计信息不准确. ---> 如InnoDB 是不能提供准确的数据行数的.
    
    * 执行计划中的成本估算不等同于实际执行的成本. 因为执行计划的成本 本来就是 "估算的".
    
    * 优化器的"最优", 和你的"最优" 可能不一样. 一般希望的都是执行时间尽可能的短 = 最优. 但优化器不一定这么认为.
    
    * 不会考虑并发查询的情况.
     
    * 有时候也不会去做基于成本的优化. 一些固定规则: 全文索引的 MATCH()字句. 如果相关列使用了全文索引,
    即是 使用别的索引 + where条件会快很多, 可 mysql 还是会使用 全文索引.
    
    * 不会考虑 存储过程 和 用户自定义函数 的成本. 
    
    * 有时候, 可能无法算出所有的可能的执行计划. 所有有可能错过最优的执行计划.(多表的关联查询等)

** 静态优化 **

    对解析树进行分析优化. 比如将where条件做等价的变换, 或在where条件中带入一些常数等. 
    静态优化在第一次完成后就一直有效. 
    

** 动态优化 **

    动态优化和查询的上下文有关, 如索引中条目对应的数据行数. 每次查询的时候都需要重新获取.
    
    
#### mysql 能够处理的优化类型:



* 重新定义关联表的关联顺序
mysql 并不总是按照查询中指定的顺序进行关联查询的.


* 外链接 ---> 内连接

* 等价变换
比如一堆 and < > 条件之间的关系

* 优化 count(), min(), 和 max()
例如 count(*): 找到某一列的最小值,  找到该列B-Tree索引最左端的记录.

* 预估并转化为常数表达式
例如数学表达式等.

* 覆盖索引扫描
当索引中的列包含查询所需要的列时. mysql 就可以使用索引返回需要的数据, 不用再去查询数据行.

* 提前终止查询
满足查询的时候,  就会提前终止查询.  如使用了  limit 的时候.

* 等值传播
... join a.id = b.id where b.id > 20;
===> where a.id > 20 and b.id > 20;

* IN()
mysql 中  in() 不等于 多个or  当 in() 中的值比较多时, 速度会比 多个or 快.


### 执行计划
是 mysql 生成的一颗指令书, 交给存储引擎去处理,然后返回结果. 
 
### 排序优化 
如果 mysql 不能使用索引来排序的话, 那么他会自己排序. filesort. 如果数据量小, 会在内存排序. 数据量大的话会使用磁盘. 但都是 filesort.


mysql 对每一个排序记录都会分配一个足够长的定长空间来存放. 这个空间大小 大于等于 其最长的字符串的大小.
 
** 对于关联表的排序, 如果所有的排序列都是第一张表的话. mysql 会在关联处理第一张表的时候就排序. 
否则, 其他情况就是处理完所有的关联表数据后,在排序. 此时会生成临时表. 如果有Limit 那么也会等拍完序之后再应用.  比如 取出了 1000条数据排序好了, 缺只要前20条数据. **


### tips

** 关联查询 **
    
    * on 的字句上最好有索引
    * group by 和 order by 的列最好是在一张表.
  

 
 
 
 
 
 
 
 
 
 
 
 
 
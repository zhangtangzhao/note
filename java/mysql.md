---
sort: 6
---

# Mysql

## Mysql 性能优化

### 慢查询
1. 查询了不需要的记录  比如Limit 1000,20 Mysql 会扫描1020行然后丢到前1000
2. 总是取出全部列 比如select * ，因为二级索引只会保存一部分列数据，而剩下会根据聚簇索引进行回表，回表是随机IO
3. 重复查询相同的数据

慢查询配置
show VARIABLES like 'slow_query_log';
set GLOBAL slow_query_log=1;
show VARIABLES like '%long_query_time%';
show VARIABLES like '%log_queries_not_using_indexes%'
set global log_output='FILE,TABLE';

slow_query_log 启动停止慢查询日志
slow_query_log_file 指定慢查询日志的存储路径(默认和数据文件放在一起)
long_query_time 指定记录慢查询日志SQL执行时间的伐值(单位：秒，默认10S)
log_queries_not_using_indexes 是否记录未使用索引的SQL
log_output 日志存放的地方【TABLE】【FILE】【FILE,TABLE】

### 执行计划 explain
1. table： explain语句输出的每条记录都对应着某个单表的访问方法，该记录的table列代表着该表的表名
2. id: 查询语句中每出现一个SELECT关键字，Mysql就会为它分配一个唯一的id值
3. select_type
	| 属性 | 描述 |
	| ---- | ---- |
	| SIMPLE | 简单的select查询，不使用union及子查询 |
	| PRIMARY | 最外层的select查询 |
	| UNION | UNION 中的第二个或随后的 select 查询,不 依赖于外部查询的结果集 |
	| UNION RESULT | UNION 结果集 |
	| SUBQUERY | 子查询中的第一个 select 查询,不依赖于外 部查询的结果集 |
	| DEPENDENT UNION | UNION 中的第二个或随后的 select 查询,依赖于外部查询的结果集 |
	| DEPENDENT SUBQUERY | 子查询中的第一个 select 查询,依赖于外部查询的结果集 |
	| DERIVED | 用于 from 子句里有子查询的情况。 MySQL 会 递归执行这些子查询, 把结果放在临时表里。|
	| MATERIALIZED | 物化子查询 |
	| UNCACHEABLE SUBQUERY | 结果集不能被缓存的子查询,必须重新为外层查询的每一行进行评估,出现极少。|
	| UNCACHEABLE UNION | UNION 中的第二个或随后的select 查询,属于不可缓存的子查询，出现极少。 |
	
4. partitions 和分区表有关，一般情况下我们的查询语句的执行计划partitions都为null
5. type： 执行计划的一条记录就代表着MySQL对某个表的执行查询时的访问方法/访问类型，其中的type列就表明了这个访问方法/访问类型是个什么东西，是较为重要的一个指标
   结果值从最好到最坏依次是：
	system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL 
	出现比较多的是system>const>eq_ref>ref>range>index>ALL
	一般来说，得保证查询至少达到range级别，最好能达到ref。
    | 属性 | 描述 |
	| ---- | ---- |
	| system | 表只有一行记录(等于系统表)，这是const类型的特列 |
	| const | 根据主键或者唯一二级索引列与常数进行等值匹配 |
	| eq_ref | 在连接查询时，如果被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行访问的〈如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较)，则对该被驱动表的访问方法就是eq_ref |
	| ref | 当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是ref。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他属于查找和扫描的混合体 |
	| fulltext | 全文索引 |
	| ref_or_null | 有时候我们不仅想找出某个二级索引列的值等于某个常数的记录，还想把该列的值为NULL的记录也找出来，就像这个查询：SELECT * FROM order_exp WHERE idx_order_no= 'abc' OR idx_order_no IS NULL; |
	| index_merge | 一般情况下对于某个表的查询只能使用到一个索引，在某些场景下可以使用索引合并的方式来执行查询 |
	| unique_subquery | 类似于两表连接中被驱动表的eg_ref访问方法,unique _subquery是针对在一些包含IN子查询的查询语句中，如果查询优化器决定将IN子查询转换为EXISTS子查询，而且子查询可以使用到主键进行等值匹配。 |
	| index_subquery | index_subquery与unique_subquery类似，只不过访问⼦查询中的表时使⽤的是普通的索引 |
	| range | 如果使用索引获取某些范围区间的记录，那么就可能使用到range访问方法，一般就是在你的where语句中出现了between、<、>、in等的查询 |
	| index | 当我们可以使用索引覆盖，但需要扫描全部的索引记录时，该表的访问方法就是index |
	| all | 全表扫描 |

6. possible_key与key ： possible_keys列表示在某个查询语句中，对某个表执行单表查询时可能用到的索引有哪些，key列表示实际用到的索引有哪些，如果为NULL，则没有使用索引
7. key_len : 列表示当优化器决定使用某个索引执行查询时，该索引记录的最大长度
8. ref : 当使用索引列等值匹配的条件去执行查询时，也就是在访问方法是const、eg_ref、ref、ref_or_null、unique_sutbquery、index_subopery其中之一时,ref列展示的就是与索引列作等值匹配的是谁
9. rows : 如果查询优化器决定使用全表扫描的方式对某个表执行查询时，执行计划的rows列就代表预计需要扫描的行数，如果使用索引来执行查询时，执行计划的rows列就代表预计扫描的索引记录行数。
10. filtered : 查询优化器预测有多少条记录满⾜其余的搜索条件
11. Extra : Extra列是用来说明一些额外信息的，我们可以通过这些额外信息来更准确的理解MySQL到底将如何执行给定的查询语句
			No tables used: 当查询语句的没有FROM子句时将会提示该额外信息。
			Impossible WHERE : 查询语句的WHERE子句永远为FALSE时将会提示该额外信息。
			No matching min/max row : 当查询列表处有MIN或者MAX聚集函数，但是并没有符合WHERE子句中的搜索条件的记录时，将会提示该额外信息。
			Using index : 当我们的查询列表以及搜索条件中只包含属于某个索引的列，也就是在可以使用索引覆盖的情况下，在Extra列将会提示该额外信息。
			Using index condition : 有些搜索条件中虽然出现了索引列，但却不能使用到索引
			Using where : 当我们使用全表扫描来执行对某个表的查询，并且该语句的WHERE子句中有针对该表的搜索条件时，在Extra列中会提示上述额外信息
			Using join buffer (Block Nested Loop) : 在连接查询执行过程中，当被驱动表不能有效的利用索引加快访问速度，MySQL一般会为其分配一块名叫join buffer的内存块来加快查询速度。
			Not exists : 当我们使用左（外）连接时，如果WHERE子句中包含要求被驱动表的某个列等于NULL值的搜索条件，而且那个列又是不允许存储NULL值的，那么在该表的执行计划的Extra列就会提示Not exists额外信息
			Using intersect(...)、Using union(...)和Using sort_union(...) : 如果执行计划的Extra列出现了Using intersect(...)提示，说明准备使用Intersect索引合并的方式执行查询，括号中的...表示需要进行索引合并的索引名称；如果出现了Using union(...)提示，说明准备使用Union索引合并的方式执行查询；出现了Using sort_union(...)提示，说明准备使用Sort-Union索引合并的方式执行查询。
			Zero limit : 当我们的LIMIT子句的参数为0时，表示压根儿不打算从表中读出任何记录，将会提示该额外信息。
			Using filesort : 有一些情况下对结果集中的记录进行排序是可以使用到索引的
			Using temporary : 在许多查询的执行过程中，MySQL可能会借助临时表来完成一些功能，比如去重、排序之类的。
			Start temporary, End temporary : 有子查询时，查询优化器会优先尝试将IN子查询转换成semi-join，而semi-join又有好多种执行策略，当执行策略为DuplicateWeedout时，也就是通过建立临时表来实现为外层查询中的记录进行去重操作时，驱动表查询执行计划的Extra列将显示Start temporary提示，被驱动表查询执行计划的Extra列将显示End temporary提示。
			LooseScan : 在将In子查询转为semi-join时，如果采用的是LooseScan执行策略，则在驱动表执行计划的Extra列就是显示LooseScan提示。
			FirstMatch(tbl_name) : 在将In子查询转为semi-join时，如果采用的是FirstMatch执行策略，则在被驱动表执行计划的Extra列就是显示FirstMatch(tbl_name)提示












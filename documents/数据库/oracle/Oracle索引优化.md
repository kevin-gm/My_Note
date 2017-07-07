如何某表的某个字段有主键约束和唯一性约束，则Oracle 则会自动在相应的约束列上建议唯一索引。数据库索引主要进行提高访问速度。

建设原则:

　1、索引应该经常建在Where 子句经常用到的列上。如果某个大表经常使用某个字段进行查询，并且检索行数小于总表行数的5%。则应该考虑。

　2、对于两表连接的字段，应该建立索引。如果经常在某表的一个字段进行Order By 则也经过进行索引。

　3、不应该在小表上建设索引。

优缺点:
　1、索引主要进行提高数据的查询速度。 当进行DML时，会更新索引。因此索引越多，则DML越慢，其需要维护索引。 因此在创建索引及DML需要权衡。

创建索引:
　单一索引:Create Index <Index-Name> On <Table_Name>(Column_Name);

　复合索引: Create Index i_deptno_job on emp(deptno,job); —>在emp表的deptno、job列建立索引。

　　select * from emp where deptno=66 and job='sals' ->走索引。

　　select * from emp where deptno=66 OR job='sals' ->将进行全表扫描。不走索引

　　select * from emp where deptno=66 ->走索引。

　　select * from emp where job='sals' ->进行全表扫描、不走索引。

　　如果在where 子句中有OR 操作符或单独引用Job 列(索引列的后面列) 则将不会走索引，将会进行全表扫描。

 


Sql 优化:

当oracle数据库拿到SQL语句时，其会根据查询优化器分析该语句，并根据分析结果生成查询执行计划。
也就是说，数据库是执行的查询计划，而不是Sql语句。
查询优化器有rule-based-optimizer(基于规则的查询优化器) 和Cost-Based-optimizer(基于成本的查询优化器)。
其中基于规则的查询优化器在10g版本中消失。
对于规则查询，其最后查询的是全表扫描。而CBO则会根据统计信息进行最后的选择。


1、先执行From ->Where ->Group By->Order By

2、执行From 字句是从右往左进行执行。因此必须选择记录条数最少的表放在右边。这是为什么呢？　　

3、对于Where字句其执行顺序是从后向前执行、因此可以过滤最大数量记录的条件必须写在Where子句的末尾，而对于多表之间的连接，则写在之前。
因为这样进行连接时，可以去掉大多不重复的项。　　

4. SELECT子句中避免使用(*)ORACLE在解析的过程中, 会将’*’ 依次转换成所有的列名, 这个工作是通过查询数据字典完成的, 这意味着将耗费更多的时间

5、索引失效的情况:
　① Not Null/Null 如果某列建立索引,当进行Select * from emp where depto is not null/is null。 则会是索引失效。
　② 索引列上不要使用函数,SELECT Col FROM tbl WHERE substr(name ,1 ,3 ) = 'ABC' 
或者SELECT Col FROM tbl WHERE name LIKE '%ABC%' 而SELECT Col FROM tbl WHERE name LIKE 'ABC%' 会使用索引。

　③ 索引列上不能进行计算SELECT Col FROM tbl WHERE col / 10 > 10 则会使索引失效，应该改成
SELECT Col FROM tbl WHERE col > 10 * 10

　④ 索引列上不要使用NOT （ != 、 <> ）如:SELECT Col FROM tbl WHERE col ! = 10 
应该 改成：SELECT Col FROM tbl WHERE col > 10 OR col < 10 。

6、用UNION替换OR(适用于索引列)
　 union:是将两个查询的结果集进行追加在一起，它不会引起列的变化。 由于是追加操作，需要两个结果集的列数应该是相关的，
并且相应列的数据类型也应该相当的。union 返回两个结果集，同时将两个结果集重复的项进行消除。 如果不进行消除，用UNOIN ALL.

通常情况下, 用UNION替换WHERE子句中的OR将会起到较好的效果. 对索引列使用OR将造成全表扫描. 注意, 以上规则只针对多个索引列有效. 
如果有column没有被索引, 查询效率可能会因为你没有选择OR而降低. 在下面的例子中, LOC_ID 和REGION上都建有索引.

　　高效:
　　SELECT LOC_ID , LOC_DESC , REGION
　　FROM LOCATION
　　WHERE LOC_ID = 10
　　UNION
　　SELECT LOC_ID , LOC_DESC , REGION
　　FROM LOCATION
　　WHERE REGION = “MELBOURNE”

　　低效:
　　SELECT LOC_ID , LOC_DESC , REGION
　　FROM LOCATION
　　WHERE LOC_ID = 10 OR REGION = “MELBOURNE”
　　如果你坚持要用OR, 那就需要返回记录最少的索引列写在最前面.

7. 用EXISTS替代IN、用NOT EXISTS替代NOT IN
在许多基于基础表的查询中, 为了满足一个条件, 往往需要对另一个表进行联接. 在这种情况下, 使用EXISTS(或NOT EXISTS)通常将提高查询的效率. 
在子查询中, NOT IN子句将执行一个内部的排序和合并. 无论在哪种情况下, NOT IN都是最低效的(因为它对子查询中的表执行了一个全表遍历). 
为了避免使用NOT IN, 我们可以把它改写成外连接(Outer Joins)或NOT EXISTS.

例子：

高效: SELECT * FROM EMP (基础表) WHERE EMPNO > 0 AND EXISTS (SELECT ‘X’ FROM DEPT WHERE DEPT.DEPTNO = EMP.DEPTNO AND LOC = ‘MELB’)

低效: SELECT * FROM EMP (基础表) WHERE EMPNO > 0 AND DEPTNO IN(SELECT DEPTNO FROM DEPT WHERE LOC = ‘MELB’)
## SQL的执行顺序

- 第一步：执行FROM
- 第二步：WHERE条件过滤
-  第三步：GROUP BY分组
- 第四步：执行SELECT投影列，聚集函数
- 第五步：HAVING条件过滤
- 第六步：执行ORDER BY 排序



## exists和in

**in 是把外表和内表作hash join，而exists是对外表作loop，每次loop再对内表进行查询。**
 如：

```SQL
A：
select * from t1 a where exists (select * from t2 b where b.id = a.id)

B：
select * from t1 a where a.id in (select b.id from t2 b)
```

对于A，用到了t2上的id索引，exists执行次数为t1.length，不缓存exists的结果集。
对于B，用到了t1上的id索引，首先执行in语句，然后将结果缓存起来，之后遍历t1表，将满足结果的加入结果集，所以执行次数为t1.length*t2.length次。
**因此对t1表大t2表小的情况使用in，t2表小t1表大的情况使用exists**

## not exists和not in

```SQL
A：
select * from t1 a where not exists (select * from t2 b where b.id = a.id)

B：
select * from t1 a where a.id not in (select b.id from t2 b)
```

对于A，和exists一样，用到了t2上的id索引，exists()执行次数为t1.length，不缓存exists()的结果集。
而对于B，因为not in实质上等于`!= and != ···`，因为!=不会使用索引，故not in不会使用索引。
因此，**不管t1和t2大小如何，均使用not exists效率会更高。**
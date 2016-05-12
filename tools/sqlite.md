#SQLite

## SQLite中的SQL

### 语法
```sql
select id from foods where name='JujyFruit';
|____| |___________| |_____________________|
 动词       主语              谓语
```

### 常量

> 'Jerry' 字符常量
> -1, 3.14 数字常量
> x'0abc' 二进制常量

### 关键字，标识符

> select, update, insert, create, drop, begin 等是关键字
> table, index, view 等是标识符

### 注释

```sql
-- This is a comment on one line
/* This is a comment spanning
tow lines */
```

### 创建表

```sql
create [temp] table table_name (column_definitions [, constraints]);
```

### 修改表

```sql
alter table table_name { rename to name | add column column_def }
```

### 数据库查询

#### 关系操作

1. **基本操作**
> Restriction (限制)
> Projection (投影)
> Cartesian Product (笛卡尔积)
> Union (联合)
> Difference (差)
> Rename (重命名)

2. **附加操作**
> Intersection (交叉)
> Natural Join (自然连接)
> Assign (赋值)

3. **扩展操作**
> Generalized Projection (广义投影) 
> Left Outer Join (左外连接)
> Right Outer Join (右外连接) -- not support in SQLite
> Full Outer Join (全外连接) -- not support in SQLite

#### select 命令

```sql
select [distinct] heading
from tables
where predicate
group by columns
having predicate
order by columns
limit count,offset;
```

##### SQLite中的select处理流程

![Select Flow](images/select_flow.png "SQLite中的select处理流程")

##### 过滤

###### 值

> 文字值
> 变量
> 表达式
> 函数结果

###### 操作符

![Operators](images/operators.png "二元操作符")

```sql
select * from foods where  name='JujyFruit' and type_id=9;
```

###### 限定和排序

```sql
-- like (%) 可以任意0个或多个字符匹配，(_)可以任意单个字符匹配
select id, name from foods where name not like '%ac%p%';

-- asc (默认的升序) desc (降序)，当第一个出现重复，按照第二、第三...排序
-- limit 和 offset 一起使用时，可以用逗号替代offset关键字
select * from foods where name like 'B%'
order by type_id desc, name limit 10 offset 1; -- 等效 limit 1,10
```

###### 函数(Function)和聚合(Aggregate)

> Function: abs(), upper(), lower(), length()
> Aggregates: avg(), count(), min(), max() 对表中的每一行做某种运算

```sql
select id, upper(name),length(name) from foods
where length(name) < 5 limit 5;

select avg(length(name)) from foods;
```

###### 分组(Grouping)

> group by 介于 where 和 select 子句中间

```sql
-- 获取每个type_id组的记录数目
select type_id, count(*) from foods group by type_id;
```
> having 从 group by 中过滤组的方式与 where 子句从 from 子句过滤行的方式相同

```sql
-- 食品小于20个的食品类型
select type_id, count(*) from foods
group by type_id having count(*) < 20;
```
###### 去掉重复

> distinct 处理 select 的结果并过滤掉其中的重复行

```sql
select distinct type_id from foods;
```

###### 多表连接

```sql
-- join_type (inner join, left outer join, cross join)
select heading from left_table join_type right_table on join_condition;

-- 对于第一个表中的每一行， 数据库都要查询第二个表的所有行
-- 交叉连接, 语法不推荐
select foods.name, food_types.name
from foods, food_types
where foods.type_id=food_types.id limit 10;

-- 内连接
select * from foods inner join food_types on foods.id = food_types.id;

-- 左外连接, foods是左表, 所有行都包含在结果中，foods_episodes无法提供的行以null补充
select * from foods left outer join foods_episodes
on foods.id = foods_episodes.food_id;
```

###### 名称和别名

```sql
select f.name as food, e1.name, e1.season, e2.name, e2.season
from episodes e1, foods_episodes fe1, foods f,
     episodes e2, foods_episodes fe2
where
  -- Get foods in season 4
  (e1.id = fe1.episode_id and e1.season = 4) and fe1.food_id = f.id
  -- Link foods with all other epsisodes
  and (fe1.food_id = fe2.food_id)
  -- Link with their respective episodes and filter out e1's season
  and (fe2.episode_id = e2.id AND e2.season != e1.season)
order by food;
```

###### 子查询

> 指select语句中又嵌套select语句

```sql
-- order by 字句，根据所在食品所属组的食品数排序
select * from foods f
order by (select count(type_id)
from foods where type_id=f.type_id) desc;

-- from 字句
select f.name, types.name from foods f
inner join (select * from food_types where id=6) types
on f.type_id=types.id;
```

###### 复合查询

> 涉及的关系的字段数据必须相同
> 只能有一个 order by 字句, 并且在复合查询的最尾部
> union (联合, A和B)、intersect (交叉, 即在A也在B)、except (差集, 在A不在B)

```sql
-- episodes位于前10,但不在3～5季节期间的食品
select f.* from foods f
inner join
  (select food_id, count(food_id) as count from foods_episodes
     group by food_id
     order by count(food_id) desc limit 10) top_foods
  on f.id=top_foods.food_id
except
select f.* from foods f
  inner join foods_episodes fe on f.id = fe.food_id
  inner join episodes e on fe.episode_id = e.id
  where e.season between 3 and 5
order by f.name;
```

###### 条件结果

> case 如果满足的条件有多个，也只执行第一个

```sql
select name, (select
                case
                    when count(*) > 4 then 'Very High'
                    when count(*) = 4 then 'High'
                    when count(*) in (2,3) then 'Moderate'
                    else 'Low'
                end
                from foods_episodes
                where food_id=f.id) frequency
from foods f
where frequency like '%High';
```

###### SQLite中的null

> 只能通过 is null 或者 is not null 检测 null 是否存在， 不能使用 （var = null) 判断
> coalesce 返回第一个非 null 值
> nullif(a,b) 如果a等于b,则返回null，否则返回a

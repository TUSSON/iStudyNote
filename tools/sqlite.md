#SQLite

## SQLite中的SQL

### 语法
```
select id from foods where name='JujyFruit';
|____| |___________| |_____________________|
 动词       主语              谓语
```

### 常量

- 'Jerry' 字符常量
- -1, 3.14 数字常量
- x'0abc' 二进制常量

### 关键字，标识符

- select, update, insert, create, drop, begin 等是关键字
- table, index, view 等是标识符

### 注释

`-- This is a comment on one line`
```
/* This is a comment spanning
tow lines */
```

### 创建表

`create [temp] table table_name (column_definitions [, constraints]);`

### 修改表

`alter table table_name { rename to name | add column column_def }`

### 数据库查询

#### 关系操作

1. 基本操作
    * Restriction (限制)
    * Projection (投影)
    * Cartesian Product (笛卡尔积)
    * Union (联合)
    * Difference (差)
    * Rename (重命名)

2. 附加操作
    * Intersection (交叉)
    * Natural Join (自然连接)
    * Assign (赋值)

3. 扩展操作
    * Generalized Projection (广义投影) 
    * Left Outer Join (左外连接)
    * Right Outer Join (右外连接) -- SQLite not support
    * Full Outer Join (全外连接) -- SQLite not support


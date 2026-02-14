---
title: MySql
copyright: true
date: 2023-09-14 23:49:13
updated: 2023-09-17 22:17:13
tags:
    - 数据库
    - 课程笔记
categories:
    - 数据库
    - 课程学习笔记
excerpt: 课程学习使用的是MySql, 所以这里就按照数据库从创建到使用的过程来总结一些MySql的使用方法. 
#index_img:
#banner_img:
---
> 写在前面: 作者习惯于使用小写的sql命令, 所以下面的内容都将使用小写sql命令. 所有sql命令都会在mysql中运行测试, 保证所有内容的正确性与可行性. 解释语法时, 使用尖括号('<>')来代表必需的参数, 使用中括号/方括号('[]')来代表可选参数, 使用三个句点('...')表示重复上面的参数.

# 数据库

数据库是关系的集合, 也就是关系表的集合, 我们后面在进行增删改查操作时都是在一个数据库中对一个或多个关系表进行的, 所以我们需要先创建一个数据库用来存储我们所有的关系表.

## 创建数据库

语法: `create database <数据库的名字>`
实例: 创建一个大学数据库

```sql
create database university;
```

## 使用数据库

语法: `use <数据库的名字>`
实例: 使用刚刚创建的大学数据库

```sql
use university;
```

## 删除数据库

语法:

```sql
drop database <数据库名>
```

# 数据表

我们大部分的工作都将围绕数据表展开, 所以这很重要.

## 创建数据表

语法:

```sql
create table <数据表名字>(
    <数据名> <数据类型> [数据约束][, 
    <数据名> <数据类型> [数据约束]]
    [...]
    [关系表的一些设置]
);
```

实例: 创建一个教室表, 并设置主码为建筑名和房号(关于其中的数据类型和设置可以往后看看再回头看这个)

```sql
create table classroom(
    building varchar (15),
    room_number varchar (7),
    capacity numeric (4,0),
    primary key (building, room_number)
);
```

### 数据类型

详细的去看菜鸟教程即可, 点击链接跳转 -> [mysql数据类型](https://www.runoob.com/mysql/mysql-data-types.html) , 这里简要的整理一下.

1. 数值类型: 就是各种数字, 如整数, 小数等. 它们都有各自的取值范围, 所以选择合适的数值类型很重要
2. 日期和时间: 其中的日期格式都有严格的规定, 如 `DATETIME`必须是YYYY-MM-DD hh-mm-ss
3. 字符串: 不同类型的大小不同, 常用的是 `char`和 `varchar`, 它们都要在其后跟一个小括号括起来的数字代表字符串的长度

### 数据约束

#### check 约束

语法: `check (约束条件)`
实例: 下面的例子约束 budget > 0, 如果试图插入一个不满足约束条件的数据或者将数据改为不符合约束条件的, 都将会报错. (如果非要插入, 那么就要删除约束)

```sql
create table department
(dept_name varchar (20),
building varchar (15),
budget numeric (12,2) check (budget > 0),
primary key (dept_name));
```

### 关系表设置

> 这一部分慢慢更新

#### 主码

唯一区分元组(表中的一行)的最小的属性(表中的一列)集合, 主码可以包含一个或多个属性.

在数据定义完后, 指定主码, 语法: `primary key (<属性1>[, 属性2][...])`

在上面的教室表中就指定了两个属性作为一个主码, 因为只有楼栋和房号一起才能确定唯一一个教室.

#### 外码

## 插入数据

语法:

```sql
insert into <关系表>
[(属性1, 属性2, 属性3, ...)]
values
(<属性1的值, 属性2的值, 属性3的值, ...>);
```

### 默认插入

**需要按表中的顺序提供所有属性的值**

对于关系表 `classroom(building, room_number,  capacity)`, 插入一个数据:

```sql
insert into classroom
values
('Packard', 101, 500);
```

### 指定属性插入

对于关系表 `classroom(building, room_number,  capacity)`, 插入一个数据:

```sql
insert into classroom
(building, capacity, room_number)
values
('Packard', 500, 101);
```

当然, 在允许一些属性为空值(null)或者提供了默认值的情况下, 可以在插入时不提供一些属性的值.

## 查找数据

语法:

```sql
select <属性>[, <属性>, ...]
from <关系表名>
[where子句]
[limit子句]
[offset子句]
[order by子句]
```

`where`子句添加选择条件

`limit`子句限制选择出来的个数

`order by`子句指定按照属性的排序方式, 例如 `order by salary desc, name asc`按照薪资降序, 薪资相同的按照姓名升序排序

如果要去重可以使用 `select distinct`

显式的选择所有数据(不去重)使用 `select all`

当 `from`子句中包含多个关系表时, 这些关系表会做笛卡尔积. 如果两个关系中存在同名的属性, 应该使用 *表名.属性* 的方式去区分不同的属性.

### 查看所有内容

实例:查看教室表中的所有内容

```sql
select *
from classroom;
```

### 查找时添加一个属性并赋值

在 `select`属性时添加一个 `<值> as <属性名>`, 下面的例子中添加了一个 `grade`属性并赋值为 `null`. 这里因为要用到union而右表缺少了一个 `grade`属性, 为了保证两个表结构一致, 所以要添加一个属性.

```sql
select *
from takes
union
select ID, course_id, sec_id, semester, year, null as grade
from student join section
where (course_id, sec_id, semester, year)=('CS-101', '1', 'Fall', 2017);
```

## 删除数据

语法:

```sql
delete from <关系表名>
[where <判断条件>];
```

- 在不指明 `where`子句时, 将会删除指定表中的所有数据, 添加后会删除满足判断条件的元组
- 删除时需要注意where的条件, 如果不是主码可能会误删
- 一定要根据非主码来删除数据, 需要把安全更新设置关闭: `SET SQL_SAFE_UPDATES = 0(或者flase);`, 打开操作就置为1(或者true)

## 删除数据表

```sql
drop table <关系表名>
```

# 约束

## 添加约束

在创建数据表的时候我们可以添加约束, 现在考虑创建数据表后, 我们突然发现需要为表中的某一个或多个属性添加约束, 那么该怎么做?

### check 约束

语法:

```sql
alter table <数据表名>
add constraint [约束名] check (<约束条件>);
```

## 删除约束

语法:

```sql
alter table c<数据表名>
drop constraint <约束名>;
```

# 连接

## 自然连接

将两个关系表做笛卡尔积后, 合并相同名称的属性, 同时只保留这些名称的值相同的元组

比如: `instructor(ID, course_id)`和 `teacher(ID, name)`自然连接, 下面两种方法是等价的:

```sql
-- 法一:
select *
from instructor, teacher
where instructor.ID = teacher.ID;
-- 法二:
select *
from instructor natural join teacher;
```

**自定义构造一个自然连接**

```sql
<table1> join <table2> using (<属性1>[, 属性2, 属性3, ...])
```

使用 `join using`将会按照指定的属性进行保留选择

# 集合

具有完全相同属性的表间的操作

## 并

`union`连接两个表, 会根据集合的性质去重, 如果不想去重就使用 `union all`

## 交

`intersect`连接两个表, 自动去重, 如果想保留两个表的内容使用 `intersect all`

## 差

`except`连接两个表, 从前表中删除后表的内容, 使用 `except all`保留重复的元组(前提是前表在减完后还有重复元组)

# 字符串

## 通配符

使用 `like`来实现模式匹配, 其中:

- `%` 匹配任意字符串
- `_` 匹配任意一个字符
- `\` 转移字符, 跟通配符表示通配符本身, 可以用 `ESCAPE`指定

## 正则表达式

- `.` 匹配任何单个字符
- `[...]` 匹配方括号内的任意字符, 例如 `[abc]`, `[0-9]` 其中'-'表示一个范围
- `*` 匹配0个或任意个在它之前的东西, 如 `X*` 匹配任意数量的X, `[0-9]*` 匹配任意个数字
- `^` 正则开始位置标识
- `$` 正则结束位置标识

## 字符串操作

- 连接: 使用 `||`
- 大小写转换: 使用 `upper(s)`, `lower(s)`
- 去除结尾空格: 使用 `trim(s)`

# 聚集函数

## 基本聚集

- avg 取平均
- min 取最小
- max 取最大
- sum 求和
- count 计数

聚集函数返回的是一个单属性的关系

聚集函数默认是不去重的(即添加 `all`关键字), 可以在聚集函数中添加 `distinct`关键字来去除重复的元组, 例如:

```sql
select count(distinct ID)
from teachers
```

## 分组聚集

`group by <属性1>[, 属性2...]`可以将关系表中的元组按照指定的属性聚集成组, 然后使用聚集函数将对每一个分组起作用, 最后聚集函数将返回一个单属性多元组的关系.

**重要:** 出现在 `select`中的没有被聚集函数聚集的属性, 必须都出现在 `group by`子句后, 下面是一个**错误案例**, 其中ID没有没聚集也没有出现在 `group by`子句后

```sql
select dept_name, ID, avf(salary)
from instructor
group by dept_name
```

## having子句

> **having**子句仅在 `group by`分组后才起作用

`having`子句将应用到 `group by`分组后的每一个组上

语句执行顺序: from $\to$ where $\to$ group by $\to$ $\to$ having $\to$ select

**重要:** 出现在 `having`子句后的没有被聚集函数聚集的属性, 必须都出现在 `group by`子句后

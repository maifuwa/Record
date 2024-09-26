## sql
SQL: 是结构化查询语言的缩写，用来访问和操作数据库系统&#x20;
DDL: 允许用户定义数据，也就是创建表、删除表、修改表结构这些操作&#x20;
DML: 为用户提供添加、删除、更新数据的能力，这些是应用程序对数据库的日常操作&#x20;
DQL: 允许用户查询数据，这也是通常最频繁的数据库日常操作
DCL：管理用户权限
### DDL
#### 创建数据库、表、索引
```sql
传输数据库
create database [if not exists] database_name;

创建表
create table [if not exists] table_name(
  table_column int [not null],
  table_column varchar [auto_increment],
  table_column varchat [default ''],
  primary key(table_column)
)[ENGINE=InnoDB]
创建表复制
create table table_name as DQL;

显式创建索引
create index index_name on table_name(...column);
修改表创建索引
alter table table_name add index index_name (...column);
创建具有唯一约束的索引
create unique index index_name on table_name (...column);
查看表的索引
show index in table_name;
删除表的索引
drop index index_name on table_name;
```

#### 创建视图、存储过程、函数、事件、触发器
```sql
创建视图(一个或多个基本表的查询结果组成,本身不存储数据，而是基于查询结果动态生成,可以被看作是一个具有特定查询结果的命名查询)
create view view_name as
  DQL;
  [with check option]   -- 防止视图被修改
删除视图
drop view view_name;
更新视图
create or replace view view_name as
  DQL;
使用视图(可以把它当表使用, 它存储在数据库中不在表里)
例: select * from view_name;
可更新视图(注意会直接影响到基础表  可更新前提：没有 distinct、min()、max()..、group by、having、union)
delete from view_name [where ] 
update view_name set column = newColumnValue [where ]
insert into view_name values ()

创建存储过程(预编译的数据库程序，它包含了一组SQL语句和控制流程语句，可以被多次调用和执行 存储在数据库里)
无参：
delimiter $  -- 将sql语句的结束符改为 $ 
create procedure procedure_name()
begin
  DQL or DML;
end $
delimiter ;
使用：call procedure_name();


带参(输入):
delimiter $ 
create procedure procedure_name(
  ...column varchar(),
  state char(2)   -- 只是为了表示多个参数写法 如果需要默认值,可以在begin里面写 if ... then ... else ... end if;
)
begin
  DQL or DML;    --直接在里面使用column即可,需要注意重名
end $
delimiter ;
使用: call procedure_name(...column)


带参(输出):
delimiter $
create procedure procedure_name(
  out ...column varchar(),
  state char(2)
)
begin
  DQL or DML;
end $
delimiter ;

在存储过程中创建本地变量使用into和set赋值 (会话中创建使用 @ 当会话结束 @ 创建的变量也没了)
create procedure procedure_name()
begin
  declare column_one decimal(x,y) default z;  -- 使用 declare 创建变量
  declare column_tow int;
  declare column_three decimal(x,y);

  select count(*), sum(balabala) into column_two, column_one from babala;

  set column_three = column_one/column_tow;
end $

删除存储过程
drop procedure [if exists] procedure_name;


函数  
-- 有 return 数据类型   
-- 有至少一个属性，除了下面的还有 reads sql data(表示每次从数据库读数据) modifies sql data(表示函数中有插入、更新、删除)
create function function_name (
  column int
)
returns int
deterministic -- 这个是确定性，表示输入一样的值会返回同样的值(一般用在不更具数据库的数据返回值里) 
begin
  DML or DQL;
  return column; --在最后要返回一个数据
end $

删除函数
drop function [if exists] function_name;


创建事件
delimiter $
create event event_name   -- 命名规则 发生时间间隔_执行动作
on schedule
  -- at '2021-05-01' 只执行一次
  every 1 day starts -- year month hour 
do begin
  DML;
end $
delimiter ;

查看、删除、修改事件
show events [like ''];
drop event [if exists] event_name;
alter event ... (与create相同)

alter event event_name disable;   -- 不启用事件
alter event event_name able;      -- 启用事件


创建触发器
delimiter $
create trigger trigger_name   -- 命名规则 tableName_after/before_(insert...)
  before insert on table_name
  for each row
begin 
  DML;    -- 可以使用 new old 返回插入或更新的新行和旧行 new.column
end $
delimiter ;

查看触发器
show triggers;

删除触发器
drop trigger [if exists] trigger_name;
```

**创建视图、存储过程、函数、事件、触发器例子**
```sql
创建视图
create view sales_by_client as
  select c.client_id, name, sum(invoice_total) as total_sases
  from clients c 
  join invoices i using (client_id)
  group by c.client_id, name;

创建存储过程 无参
delimiter $
create procedure get_client()
begin
  select * from client;
end $
delimiter ;

创建存储过程 输入参数
delimiter $
create procedure get_client_by_state (
  state char(2)   -- 默认是输入参数
)
begin
  [if state is null then set state = '' end if;]   -- 设置默认值

  select * from client c where c.state = state;    -- 直接使用即可
end $
delimiter ;

创建存储过程 输出参数
delimiter $
create procedure get_unpaid_invoices_for_client(
  client_id int,
  out invoices_count int,
  out invoices_total int
)
begin
  select count(*), sum(invoice_total) 
  into invoices_count, invoices_total
  from invoices i 
  where i.client_id = client_id and payment_total = 0;
end $
delimiter ;

调用存储过程 带参
set @invoices_count = 0;
set @invoices_total = 0;
call database_name.get_unpaid_invoices_for_client(3, @invoices_count, @invoices_total);
select @invoices_count, @invoices_total;

在存储过程中创建参数
delimiter $
create procedure get_risk_factor() 
begin 
  declare risk_factor decimal(9,2) [default 0] ;
  declare invoices_total decimal(9,2);
  declare invoices_count int;

  select count(*), sum(invoices_total)
  into invoices_count, invoices_total
  from invoices;

  set risk_factor = invoices_total / invoices_count * 5;

  select risk_factor;
end $
delimiter ;

创建函数
delimiter $
create function get_risk_factor_client(
  client_id int
)
returns integer
reads sql date
begin
  declare risk_factor decimal(9,2) [default 0] ;
  declare invoices_total decimal(9,2);
  declare invoices_count int;

  select count(*), sum(invoices_total)
  into invoices_count, invoices_total
  from invoices i
  where i.client_id = client_id;

  set risk_factor = invoices_total / invoices_count * 5;

  return ifnull(risk_factor, 0);
end $
delimiter ;

创建触发器
delimiter $
create trigger payments_after_insert
  after insert on payments
  for each row
begin 
  update invoices set payment_total = payment_total + new.amout
  where invoice_id = new.invoice_id;
end $
delimiter ;

创建事件
delimiter $
create event yearly_delete_stale_audit_rows
on schedule
  -- at '2023-07-3'
  every 1 year starts '2023-01-01' ends '2025-01-01'
do begin
  delete from payments_audit where action_date < now() - interval 1 year;
end $

delimiter ;
```
#### 表之间的关系
```sql
创建关系(外键)
create table table_name (
  column_one not null primary key，
  column_tow not null，
  foreign key foreign_name (column_tow)  -- 创建外键约束(填写的是当前表的表名) 命名规则：fk_table_name_column
    references foreign_table_name (column_tow)      -- 外键所在的表名
    on update cascade  --一同更改   还有 set null / no action / restrict(阻止主表更新)
    on delete no action -- 同上
);

添加、修改、删除已经存在的表
alter table table_name 
  add last_name varchar(20) after column;
  modify column first_name varchar(20) not null default '',
  drop column;

alter table table_name 
    add primary key (colunmn),
    drop primary key,
    drop foreign key foreign_name,
    add foreign key foreign_name (column)
      references table_name (colunm)
      on update cascade
      on delete on action;

  
更改存储引擎
alter table table_name engine = InnoDB;
```

### DQL
#### 查询写法
```sql
select ...column from table_name;  -- 如果要查询所有 select * from table_name;

排序使用: order by
如: select first_name from employees order by first_name desc, last_name;  -- 默认升序  降序 order by first_name desc;

可以直接在select中使用计算式、函数
如: select points + 10 from customers;

为展示出的列起别名使用： as
如: select points + 10 as new_points from customers;    -- 如果别名有空格，可以使用''\""  points + 10 as 'new points'

删除重复列使用: distinct
如: select distinct state from customers;

筛选数据使用: where
如: select * from users where user_id = '001';  -- 不等于有 != 和 <>

添加where后的并列条件使用: and or not     -- 优先级 and > or > not
如: select * from users where user_id = '001' and email = '001@001.com';

与集合进行比较使用: in
如: select * from users where user_id in ('001', '002', '003');

与集合最大值比较使用: all   -- 也可以直接在子查询里使用max()
如: select * from invoices where invoice_total > all (select invoice_total from invoices where client_id = 3);

与集合最小值比较使用: any   -- 也可以配合 = 代替 in
如: select * from clients where client_id = any (select client_id from invoices group by client_id having count(*) >= 2);

与区间段比较使用: between
如: select * from users where user_age between 10 and 14;

字符串比较和模糊匹配使用: like
如: select * from users where user_name like '_H%';   -- _ 表示任意一个字符， % 表示任意个任意字符  
-- 注意 mysql 在windows不区分大小写， 在linux默认区分大小写   不区分大小写时，字符串比较也不会区分大小写

正则表达式使用: regexp
如: select * from users where user_name regexp 'H';   -- 表示包含 'H'  '
--  ^H' 表示H为第一个字符   'H$' 表示H为最后一个字符   'H|S' 表示包含H和S  '[gim]e' 表示包含 'ge'、'ie'、'me'  '[a-c]d 表示包含'ad'、'bd'、'cd'  '^H|S'  'e[gidm]'

判断是否为空使用: is null
如: select * from users where user_email is not null;

分页查询使用: limit
如: select * from user limit 3;   -- 表示输出前3列  limit 3，10 表示跳过前面3条记录后返回10条记录    limit 10 offset 3 表示跳过3条记录后读取10条记录

跨数据库使用表: database_name.table_name

隐式连接
如: select * from order o, customers c where o.customer_id = c.customer_id;     -- 不写where就是笛卡尔序列，有很多无用组合

内连接使用: join \ inner join    -- 两个都可
如: select * from customers c join orders o on c.customer_id = o.customer_id;    -- 高版本mysql起别名可不加 as， 注意如果select same_column 需要表示是哪个表的 如 c.same_column

自连接      -- 就是自己和自己连接
如: select *, e.employee_id from employees e join employees m on e.manger_id = m.employee_id;

外连接使用: [outer] left join /  right join     -- 内连接 当不满足连接条件时，会自动忽略  可以用外连接显示 左连接就是左表全显示，右链接...
如: slect * from customers c left join order o on  c.customer_id = o.customer_id;  

自外连接
如: select *, e.employee_id from employees e left join employees m on e.manger_id = m.employee_id;

多表连接
如: select * from orders o 
        join customers c on o.customer_id = c.customer_id 
        join order_statuses os on o.status = os.order_statuses_id;

复合连接条件
如: select * from orders o join customers c on o.customer_id = c.customer_id and o.password = c.password;

简写连接条件使用: using
如:  select * from orders join customers using (customer_id);

合并查询结果使用: union
如: select order_id from orders union select customer_id from customer_id;

聚合函数         -- max()  min()  avg()  sum()  count()
如: select avg(user_id) from users;       
select count(payment_date) from invoices;  -- 会忽略空值  如果需要所有数量 count(*) 如果要获取没重复的 count(distinct column)

聚合分组使用: group by       -- 注意使用顺序
如: select client_id, sum(invoice_total) as invoice_sales 
    from invoices where invoice_date >= '2019-07-01'  group by client_id order by invoice_sales desc;

select invoice_date, sum(invoice_totall) as totall_sales, count(client_id) as all_client
from invoices group by invoice_date, client_id having all_client > 5;     
            -- 使用having筛选分组后的数据  having 中只能写select中有的属性

自动总计使用: with rollup 
如: select sum(invioce_totall) as totall_sales from invoices group by client_id with rollup;

子查询(where 语句中)
如: select * from products where unit_price > (
  select unit_price from products where product_id = 3;
)

相关子查询(配合 exists)
如: select * from clients c where exists (
  select client_id from invoices where client_id = c.client_id
)

子查询(select 语句中)
如: select invoice_id, invoice_total, 
      (select avg(invoice_total) from invoices) as  invoice_average,
      invoice_total - (select invoice_average) as diff_total
    from invoices;

子查询(from 语句中)
如: select * from (
      select invoice_id, invoice_total, 
      (select avg(invoice_total) from invoices) as  invoice_average,
      invoice_total - (select invoice_average) as diff_total
      from invoices;
    )

```
#### mysql 自带函数
```sql
数值函数
select round(5.73);    -- 6  四舍五入   指定精度：round(5.73, 1) 5.7 
select truncate(5.456, 2);    -- 5.45 截断数字
select ceiling(5.45);     -- 6 返回大于等于当前数的最小整数
select floor(5.3);        -- 5 返回小于等于当前数的最大整数
select abs(-12);          -- 12 绝对值
select rand();            -- (0, 1) 随机数

字符串函数
select length('sky');     -- 3 字符串长度 中文 * 3
select upper("sky");      -- SKY 改大写
select lower('sky');      -- sky 改小写
select ltrim('  sky');    -- sky 删除左边的空格
select rtrim('sky  ');    -- sky 删除右边的空格
select trim('  sky  ');   -- sky 删除两边的空格
select left('sky', 4);    -- sky 从左边截取4个字符，不足忽略  中文没影响
select right('sky', 3);   -- sky 从右边截取3个字符，不足忽略  中文没影响
select substring('sky', 1, 2);  -- 从第一个开始，截取2个   注意从1开始读
select locate('k', 'sky');    -- 2 查询字串，没找到返回 0
select replace('sky', 'ky', 'ql');  -- sql 替换字符
select concat('sky', 'sql');    -- skysql 连接字符

时间函数
select now();       -- 获取当前日期时间  如: 2023-06-22 11:23:45
select curdate();  -- 获取当前日期 如: 2023-06-22
select curtime();  -- 获取当前时间 如: 11:23:45
select year(now()); -- 获取年份（返回值是整数）  如: 2023   还有  month() day() hour() minute() second()  
select dayname(now());    -- 获取星期（返回字符串） 如: Monday  还有 monthname()     
select extract(day from now());   -- 其他数据库没有
select date_format(now(), '%m %d %Y');
select time_format(now(), '%H:%i%');
select date_add(now(), interval 1 day);   -- 日期计算  如: 2023-06-23 
select date_sub(now(), interval 1 day);   -- 等价于  date_add(now(), interval -1 day) 
select datediff('2023-6-22', '2023-6-25');  -- 不会考虑时间 后减前
select time_to_sec('09:00');     -- 返回从 00:00 到（）的秒数 计算时间间隔 time_to_sec(curdtime()) - time_to_sec('09:00')


其他函数
null值替换使用: ifnull( , )
如: select order_id, ifnull(shipper_id, 'not assigned') from orders;   -- 前为空，返回后者

单一比较使用： if( , , )
如: select order_id, order_date, 
      if (year(order_date) = year(now()), 'Active', 'Archived') as category   -- 为真返回前一个，假返回后一个
    from orders;

select order_id, if(year(order_date) < year(now()), 
      if (year(order_date) = year(now()) - 1, 'Last Year', 'Archived'), 'Active') 
from orders;    -- if 能嵌套使用

多个比较使用: case
如: select order_id,
      case 
        when year(order_date) = year(now()) then 'Active'
        when year(order_date) = year(now()) - 1 then 'Last Year'
        else 'Archived' 
      end as category
    from orders;

    select 
      case 
        when age < 25 or age is null then '25岁以下'
        when age >=  25 then '25岁及以上'
      end age_cut, 
      count(*) as number
    from user_profile
    group by age_cut;


SHOW VARIABLES LIKE '%time_zone%';  # 查看mysql时区
```

#### mysql 的数据类型
数据结构的选取要，能最小范围就最小范围

字符串:

| 数据类型       | 用在何处      | 储存长度max(length) |
| ---------- | --------- | --------------- |
| char       | 定长字符串     | 255B            |
| varchar    | 变长字符串     | 65535(64kB)     |
| mediumtext | 存储文本、短篇书籍 | 16MB            |
| longtext   | 存储长文本、日志  | 4GB             |
| tinytext   | 小文本       | 255b            |
| text       | 文本类型      | 64KB            |

> 附： varchar 定义一定要指定长度,一般小长度 50，大的255 字符长度: Englist 1b 欧洲和中东 2b 亚洲语言 3b

整数:

| 数据类型             | max(length) | 数值范围         |
| ---------------- | ----------- | ------------ |
| tinyint          | 1b          | \[-128, 127] |
| unsigned tinyint | 1b          | \[0, 255]    |
| smallint         | 2b          | \[-32K, 32K] |
| mediumint        | 3b          | \[-8m, 8m]   |
| int              | 4b          | \[-2B, 2B]   |
| bigint           | 8b          | \[-9z, 9z]   |

> 附: 当数值超过返回会报错

浮点数: decimal(p,s) 总共 p 位，小数位 s 位 精确值 其他浮点数 float 4b double 8b 并不是很精确

布尔类型 bool 或 boolean 其实就是tinytext 0 false 1 true

枚举和集合类型 enum(...) ... 就是这个类型的全部取值(避免使用) set(...) 同上

日期和时间

| 数据类型      | 用在何处 | 注意事项                        |
| --------- | ---- | --------------------------- |
| date      | 日期   | -                           |
| time      | 时间   | -                           |
| datetime  | 日期时间 | 日期在前时间在后 8b 随便存             |
| timestamp | 时间戳  | 记录某行插入或最近更新时间 4b 只能存储到2038年 |
| year      | 年份   | 四位整数                        |

blob类型 二进制长对象类型，一般用来存储大型二进制数据 但是一般不将数据存放在关系数据库中

| 数据类型       | 最大长度 |
| ---------- | ---- |
| tinyblob   | 255b |
| blob       | 65KB |
| mediumblob | 16MB |
| longblob   | 4GB  |

JSON类型(mysql 8.x 版引入) mysql 自动JSON格式化函数
```sql
JSON_OBJECT(
  'weight', '10',
  'dismensions', JSON_ARRAY(1, 2, 3),
  'manufacturer', JSON_OBJECT('name', 'sony')
)
```

### DML
```sql
插入
insert into table_name (column_one, column_tow...) value (default...);  -- values()可同时插入多行
insert into table_name values (...),(...);  -- table_name后面没写就按照表的顺序全添加


更新
update table_name set column_one = newData, column_tow = newData [where (...)];
update table_name set column_one = newData, column_tow = newData [where (DQL)];

删除行
delete from table_name where (...)\(DQL);

```

### DCL
```sql
创建用户
create user user_name@host identified by '123456789'   -- @127.0.0.1 \  @qiqifun.me \ @'%.qiqifun.me' 表示能从全部域名进入  \ @'%' 表示任意网段

查看用户
select * from mysql.user;

删除用户
drop user user_name@host;

修改密码
set password for user_name = '123456789';    -- 修改当前用户可以直接 set password = '12456789'

授予权限
grant select, insert, update, delete, execute, create view on database_name.* to user_name;
grant all on *.* to user_name;    -- 授予全部权限

查看权限
show grants for user_name;   -- 查看自己的 show grants;

撤销权限
revoke select, insert.... on database_name.* from user_name;
```

### 事物并发

事务的四个性质: 原子性、一致性、隔离性、持久性

mysql 有一个自动开启事务查看 `show variables like 'autocommit';` 关闭可使用 `set autocommit = off`
```sql
start transaction;    -- 开启事务
DML;
commit;               -- 提交将改动提交到数据库中 如果有错误就会自动 rollback
   -- 当测试时可写 rollback;
```

并发问题: **脏读**: 一个事务读取了尚未被提交的数据 **不可重复读**: 一次事务中读取了两次数据，数据结果不一致 **幻读**: 当执行查询操作后，数据库又插入了一条符号要求的数据

解决并发问题需要建立事务隔离等级 什么都解决不了 read uncommitted -- 不加锁 速度最快 解决脏读 **read committed** 解决不可重复读 **repeatable read** -- mysql 默认 解决幻读 **serializable** -- 最严格 当回影响此次事务的事务执行时，回等待其执行完在执行 性能不行

查看系统隔离等级 `show variables like 'transaction_isolation';` 为下一个事务设置事务隔离等级 `set transaction isolation level read committed;` 要为这次会话设置 `set session transaction isolation level read committed;` 为所有会话的所有新事务设置 `set global transaction isolation level read committed;`

死锁 死锁与隔离等级无关 就是两个事务的记录相互被锁 为了减少死锁，在更新记录时尽量依照相同的顺序 和 尽量简化事务，减少事务的执行时间

### Mysql
#### 基础架构
#### 缓存
`mysql`查询缓存，在生产环境中建议关闭，在`5.7.20`默认关闭、`8.0及之后`被删除
> 实际项目建议使用本地缓存(Caffeine)或分布式缓存(如：Eedis)
- 查询缓存会将查询语句和结果集保存到内存
- 缓存的结果是通过 sessions 共享的，所以一个 client 查询的缓存结果，另一个 client 也可以使用
- SQL 必须完全一致才会导致查询缓存命中...
优点：
1. 查询缓存查询，除了查询权限验证开销基本无其他花费
2. 从内存中取结果速度快
缺点：
1. 对每条`select`语句进行`hash`计算，查询语句多了开销大
2. 如果表的变更速度快，缓存失效率高
3. 查询语句不同，但结果相同的查询都会被缓存到
4. 相关系统设置不合理，会造成大量内存碎片
## nosql
NoSQL全称是Not Only SQL（不仅仅是SQL）它是一种非关系型数据库&#x20;

缺优点：
* 不保证关系数据的ACID特性
* 并不遵循SQL标准
* 消除数据之间关联性
* 非常易于扩展
* 数据模型更加灵活
* 高可用

一般分为：键值存储数据库、列存储数据库、文档型数据库、图形数据库
## Redis
Redis 是键值储存数据库，它所有的数据全部存放在内存中，因此性能大大高于关系型数据库，并且它也可以支持数据持久化，还支持横向扩展、主从复制等

### redis.config
[redis.conf 下载地址](https://redis.io/docs/management/config/)
常见配置(默认项)
`port 6379` 监听6379端口
`bind 127.0.0.1 -::1` ip白名单，标注的ip才能访问.注释允许任意ip
`daemonize no` 不启用守护进程，yes 会自动写入PID到/var/run/redis.pid
>When running under daemontools, use `daemonize no`.

`# maxmemory <bytes>` 设置最大占用内存，默认注释需自行配置
> 在设置时需要考虑redis除数据存储之外的开销和碎片开销，不建议设置为内存大小。此外还需要为linux开启swap(大小和内存大小一致)
> 达到最大内存后，会根据maxmemory-policy删除key，如果删除不了或maxmemory-policy为`noeviction`，redis在执行需要获取内存的操作会报错，读操作正常

`# requirepass foobared` 添加访问密码
`protected-mode yes` 保护模式
### 数据结构
[数据结构文档](https://redis.io/docs/data-types/)、[redis commands速查](https://redis.io/commands/)
五种基本数据结构：String、List、Hash、Set、Sorted Set
四种特殊类型：Bitmap、Bitfields、HyperLogLog、Geospatial

常用通用命令：`del`、`expire`
String常用命令：`set`、`setnx`、`get`、`mset`、`mget`、`strlen`、`incr`、`decr`
List常用命令：`rpush`、`lpush`、`lset`、`lpop`、`rpop`、`llen`、`lrange`
Hash常用命令：`hset`、`hsetnx`、`hmset`、`hget`、`hmget`、`hgetall`、`hexists`、`hdel`、`hlen`、`hincrby`
Set常用命令：`sadd`、`smembers`、`scard`、`sismember`、`sinter`、`sinterstore`、`sunion`、`sunionstore`、`sdiff`、`sdiffstore`、`spop`、`srandmember`
Sorted Set常用命令：`zadd`、`zcard`、`zscore`、`zinterstore`、`zunionstore`、`zdiffstore`、`zrange`、`zrevrange`、`zrevrank`

Bitmap常用命令：`setbit`、`getbit`、`bitcount`、`bitop`
HyperLogLog常用命令：`pfadd`、`pfcount`、`pfmerge`
Geospatial常用命令：`geoadd`、`geopos`、`geodist`、`georadius`、`georadiusbymember`

> 虽然是不同数据结构但key重复仍然会报错
### 数据持久化
RDB 将数据以xxx.rdb文件保存在本地
优缺点: 加载速度快、数据体积小**但**存储速度慢大量消耗资源、会发生数据丢失
```shell
save      # 直接保存,会占用一定时间
bgsave    # 后台保存

# 可在配置文件配置 save 自动保存(配置的都是后台保存)
	save 3600 1    # 3600秒内有1次修改则执行一次
save 300 100   # 同时生效
save 60 10000
rdbcompression no # 是否开启压缩，建议不开启 会占用cup
dbfilename dump.rdb # RDB文件名称
dir ./ # 文件保存的路径目录
```

AOF 将每次执行的指令以日志的形式保存在本地，每次重启将所有命令依次执行
优缺点: 存储速度快、消耗资源少、支持实时存储**但**加载速度慢、数据体积大
```shell
appendonly yes   # 开启数据持久化

appendfsync everysec   #每秒保存一次  always(每次写操作执行)   no(系统决定)

appendfilename "appendonly.aof" # AOF文件的名称
auto-aof-rewrite-percentage 100   # AOF文件比上次文件增长超过多少百分百比则出发重写
auto-aof-rewrite-min-size 64mb    # AOF文件体积到达多少体积触发重写
```

> AOF和RDB并不冲突可以同时使用
### 应用场景
常见应用场景：分布式缓存、分布式锁、限流、消息队列、延时队列、分布式session、统计UV、计数器...

分布式锁使用`setnx`实现互斥
限流使用`redisson`提供的`RRateLimiter`
消息队列可以使用`List`的`lpush`、`rpop`或`Pub/Sub`或`Stream`
延时队列可以使用`Sorted Set`将时间作为`score`
统计UV可以使用`HyperLogLog`
统计登录可以使用`Bitmap`
### 高可用
redis有三种方式来实现高可用：主从复制(Replication)、哨兵模式(Sentinel)和集群模式(Cluster)

> 在使用docekr或其他具有端口映射容器，port最好配置为1:1来保证redis的自动检测
#### 主从复制
[官方文档](https://redis.io/docs/management/replication/)
在主从复制模式中，主节点负责接收客户端的写入操作，并将写入的数据复制到所有从节点，而从节点则负责复制主节点的数据并处理客户端的读取请求。

创建集群：
只需要在从节点的redis.config修改命令为
```shell
replicaof host port  # replicaof 192.168.1.1 6379

masterauth master-password   # 主节点密码，如果有的话
```
> See the example `redis.conf` shipped with the Redis distribution for more information.

也可以使用redis-cli来创建，在从节点执行`replicaof host post`
> 将从节点转为主节点？`replicaof on one` 
> `slaveof`命令同样职责(在redis 5.0.0被废弃了~~，但为了向下兼容仍然保留~~)

查看集群连接情况`info`和`role`

优点：
- 实现了读写分离，提高了系统并发量
- 实现了数据冗余，提高数据可靠性。
缺点：
- 当master宕机，需要手动从slave中选择一个设置为master，并指定其他slave连接新的master
- 不支持横向扩展来缓解写压力和缓存量过大

> 主从复制模式下，确保master有数据持久化或不会自动重启。如果master以空数据集重启，slave的数据也会被清除

问题：主从复制下从节点会主动删除过期节点吗？主从节点之间如何数据同步？
#### 哨兵
[官方文档](https://redis.io/docs/management/sentinel/)
Sentinel是redis的一种运行模式，不提供读写服务。他提供监控、故障转移、通知、配置提供等服务来帮助redis集群自动化故障转移

创建一个Sentinel
```bash
redis-sentinel /path/to/sentinel.conf
# 或者直接从redis创建
redis-server /path/to/sentinel.conf --sentinel
```
> sentinel.conf是必需的，没有无法启动。默认端口：26379 
> 建议将sentinel设置成单数且大于等于3台

sentinel.conf
```shell
port 26379  
# master password
sentinel auth-pass master_name password
# master_name ip/host port quorum
sentinel monitor master redis_master 6379 1  
sentinel down-after-milliseconds master 5000  
sentinel failover-timeout master 60000  
sentinel parallel-syncs master 1  

# enbale host analyze need NDS is good, since redis 6.2
sentinel resolve-hostnames yes
```
> `quorum`指当多少个sentinel认为master宕机了，master被全体sentinel认为宕机

问题：Sentinel如何检测节点下线？Sentinel如何选项新master？Sentinel如何从节点中选择Leader？

#### 集群
[官方文档](https://redis.io/docs/management/scaling/)
当并发量、缓存的数据量很大，就需要使用Cluster模式来进行横向扩展(动态扩容和缩容)。通过部署多台master，master之间平等同时对外提供读/写服务，将缓存的数据均匀的分配到这些实例上，而客户端的请求则通过路由规则转发到目标实例(客户端不需要维护路由规则，集成配套SDK即可)

> 为了保证集群整体高可用，可以为每一个master再分配几个slave
> cluster具备自动故障转移和节点重新平衡的高可用，所以不需要sentinel

~~未完成~~
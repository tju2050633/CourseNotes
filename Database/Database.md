[toc]

例题精选：

- P8 e.➗
- P9 a.“任何”连接用自然连接，b.“全部”连接用➗，c.d.关系代数的聚合函数，聚合时group by的属性如何选择？
- P11 a. except，b. 同一个表的连接和join using
- P12 e. 找分公司所在城市完全相同的公司，子查询、except、not exists
- P13 update用法
- P14 a. with用法。答案有点错，子查询应该要限定branch_name="Brooklyn"
- P17 c. delete用法
- P18 b. P9 b.➗的另一种方式，判定except后是否还有剩余。d. 从member计算会员总数，从borrowed中计算借阅数量，从而避免没有借阅的人不出现在borrowed中。
- P21 a. 外码创建（or）
- P23 e. g. 3NF BCNF分解（？存疑）
- P27表格法证明lossless
- P30 关系模式综合大题
- P36 索引策略（元组个数存疑）



# 关系查询语句

### 基础语法

- 基于数学上两类运算
  - 关系代数
  - 关系演算

- 基本算符：
  - $\cup -\Pi\sigma\times$
  - $\Join\cap\div$不是，可以从这些推导出来

- Selection $\sigma$

- Projection $\Pi$

- 笛卡尔积$\times$
  - $r\times s =\{tq|t\in r\ and\ q\in s\}$
  
- 自然连接$\Join$
  - join条件写右下角标
    - $r\Join_{\theta} s = \sigma_{\theta}(r\times s) $
  - 左（右、全）外连接：左（右、全）边两个顶点画两根横线
  
- 并$\cup$
  - $r\cup s =\{t|t\in r\ or\ t\in s\}$
  
- 交$\cap$
  - $r\cap s =\{t|t\in r\ and\ t\in s\}$

- 差$-$

  - $r-s =\{t|t\in r\ and\ t\notin s\}$

- 重命名$\rho_{X}(E)$将表达式E重命名为X

- 除$\div$：$r(R),s(S),S\subset R，则r\div s是满足t\times s\subseteq r的最大关系t(R-S)$

  - 想象为笛卡尔积的逆运算

  - 例：$r(ID,course\_id)=\Pi_{ID,course\_id}(takes)$是所有学生的选课关系，$s(course\_id)=\Pi_{course\_id}(\sigma_{dept\_name='Biology'}(course))$是生物系开的所有课号，则$r\div s$返回选了所有生物系开的课的学生学号

    

### 聚合函数

- avg
- min
- max
- sum
- count

一般形式：

$_{G_1,G_2,...,G_n}G'_{F_1(A_1),F_1(A_1),...,F_n(A_n)}(E)$

- E：任意关系代数表达式
- $G_n$：分组属性
- $F_n$ ：聚合函数
- $A_n$：属性名

eg.

> 找出每个部门的平均工资

$_{dept\_name}G'_{avg(salary)}(instructor)$

### 例子

eg.

> 找出比任意其他教师工资低的教师工资

$\Pi_{T.salary}(\sigma_{T.salary<S.salary}(\rho_T(instructor)\times \rho_S(instructor)))$

> 找出最高工资

$\Pi_{salary}(instructor)-\Pi_{T.salary}(\sigma_{T.salary<S.salary}(\rho_T(instructor)\times \rho_S(instructor)))$

eg.

>```
>使用教材中的大学数据库模式，用关系代数查询来找出多于一个教师授课的课程段，分别用
>以下两种方式来表达:
>```
>
>1. 使用聚集函数 2. 不使用聚集函数

**1**

teaches和section自然连接，按sec_id分组，找出多于1个教师的section，然后查询其sec_id和用COUNT查询教师数量。

$\Pi_{sec\_id, count(id)\ as\ ins\_count}
(\sigma_{count(distinct\ id) > 1} (section ⨝ teaches))$

```sql
select sec_id
from section natural join teaches
group by sec_id
having count(distinct id) > 1;
```

**2**

teaches和自己join，取相同section、不同教师的元组拼接

$\Pi_{t_1.sec\_id} (\sigma_{t_1.sec\_id = t_2.sec\_id\ \and\ t1.id <> t2.id} (teaches ⨝ teaches))$

```sql
select distinct t1.sec_id
from teaches t1 join teaches t2 
on t1.sec_id = t2.sec_id and t1.id <> t2.id;
```





# 初级SQL



### 数据类型

- char(n) 
- varchar(n) 
- int.
- smallint. 
- numeric(p,d) 
- float(n)

- null
  - 任何包含null的算术表达式结果为null
  - **is null**和**is not null**判定
  - null既不是true也不是false
  - where的条件如果为unknown则视为false

### 定义

**创建**

- 数据类型
- 非null
- 主码
- 外码

```sql
create table instructor(
		ID char(5),
		name varchar(20) not null, 
		dept_name varchar(20), 
		salary numeric(8,2), 
		primary key(ID),
		foreign key(dept_name) references department 
 )
```

eg.

> Student(Sno Char(7),Sname Char(8),Ssex Char(2), Sage Int,Sdept Char(2))
>
> Course(Cno Char(8), Cname Char (20), Ccredit Int)
>
> SC(Sno Char(7), Cno Char(8), Grade INT)
>
> (1)用SQL语句建立数据表SC，以(Sno, Cno)作为主码;

```sql
create table SC(
Sno char(7),
Cno char(8),
Grade integer,
primary key (Sno, Cno))
```

**插入**

```sql
insert into table_name(c1, c2 ...)
values (v1, v2 ...)
```

> (2)以你的个人信息向Student表插入一条记录。其中系别字段(Sdept)的值需为软件工程系 的系别代码SE;

```sql
insert into Student values('2050633', '卢嘉霖', '男', 20, 'SE')
```

**修改元组**

```sql
update table_name
set c1=v1, c2=v2 ...
where [condition]
```

> 对事故报告编号为“AR2197”中的车牌是“AABB2020”的车辆损坏保险费用更新到9900美元。

```sql
update participated
set damage_amount = 9900
where report_number = 'AR2197' and license = 'AABB2020' 
```

**修改列**

```sql
# 增加列
alter table table_name 
add column_name datatype;
# 删除列
alter table table_name 
drop column_name;
# 改变数据类型
alter table table_name 
modify column column_name datatype;
```

**删除元组**

```sql
delete from table_name
where [condition]
```

不加条件，则删除表中所有元组，但表本身不变。

**删除表**

```sql
drop table table_name
```



### 基础查询

```sql
# 执行顺序：from, where, select, group by, having, order

select distinct column1 as c1, column2 as c2
from table_name t1, table_name t2
where country='CN' and city<>"Shanghai" 
group by cloumn3
having aggregate_fn(c4)
order by c1 desc, c2 asc
```

- 条件：
  - 等于= 
  - 不等于<> 
  - 范围内(not) between ：where salary between 100 and 200
  - 字符串匹配(not) like ：%任意字符串，_任意字符
  - 指定可能值集合(not) in：in (子查询)，in ('value1', 'value2')

### 聚合函数

- group by
  - 根据一组属性分组

- avg
- min
- max
- sum
- count
- 忽略null的聚合函数：
  - sum
  - max
  - avg




### Join

- 指定列的join：

```sql
select c1, c2
from table1 join table2 using c3 #指定在c3上join，即两个表该列要相等
```

- inner join：inner可省略，指定连接条件

```sql
select c
from table1 t1 inner join table2 t2 on t1.s = t2.s
```

- left join、right join和full join：左侧（右侧、两边）表如果有未匹配到的元组，也会显示出来，多出来的列为null

eg.

> 找出所有至少选修了一门Comp.Sci.课程的学生姓名，保证结果中没有重复的姓名。

```sql
select distinct name
from student natural join takes join course using (course_id) 
where course.title = 'Comp.Sci.';
```



### WITH

```sql
WITH cte_name (column1, column2, ...) AS (
    -- 查询定义
    SELECT column1, column2, ...
    FROM table_name
    WHERE condition
)
-- 主查询
SELECT ...
FROM ...

```

eg.

> 使用with子句而不是函数调用来写如下查询:返回教师数大于12的系的名称和预算。

```sql
with instructor_counter(dept_name, instructor_number) as 
	(select dept_name, count(ID)
	from department natural join instructor
	group by dept_name)
select dept_name, budget
from instructor_counter natural join department 
where instructor_number > 12;
```





### 集合运算

- union：合并两个select结果，两者必须有相同数量的列、数据类型和顺序，union默认distinct，可以用union all保留重复

```sql
select s from t1
union
select s from t2
```

- intersect：求交集

```sql
select s from t1
intersect
select s from t2
```

- except：差集

```sql
select s from t1
except
select s from t2
```

Eg.

> 找出所有没有选修在2009年春季之前开设的任何课程的学生的ID和姓名。

```sql
(select ID, name
from student)
except
(select ID, name
from student natural join takes where takes.year < 2009);
```

> ```
> 考虑如下的保险公司数据库，其中加下划线的是主码。
> ```
>
> person (driver_id, name, address)
>  car (license, model, year)
>  accident (report_number, date, location) owns (driver_id, license)
>
> participated (report_number, license, driver_id, damage_amount)
>
> 为这个关系数据库构造出如下SQL查询:
>
> 1. 找出和你拥有的车有关的交通事故数量。

```sql
select count(report_number)
from participated join owns using (license)
where owns.driver_id = 'your_id';
```

- all/some/any：将子查询结果与外部查询条件比较

```sql
select c1 
from t1
where c1 > some( # 或any、all
	select c1 
  from t2
)
```

- (not)  exists：检查子查询是否有结果

```sql
select c1 
from t1
where exists(
	select c1 
  from t2
  where [condition] # 通常涉及t1和t2内元组的比较
)
```



# 中级SQL

### 视图

建立视图后，可以像访问普通表一样使用它。

视图只存储了查询定义，而不存储实际的数据。

```sql
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table1
WHERE condition;
```

Eg.

> 建立一个信息系统系(系别代码IS)所有不及格(Grade<60)学生的视图vwStudent。

```sql
create view vwStudent(Sno, Sname, Ssex, Sage, Sdept, Cno, Grade) as
	(select Sno, Sname, Ssex, Sage, Sdept, Cno, Grade
	from Student natural join SC
	where Sdept = 'IS' and Grade < 60)
```





### 事务

**定义**

- 一组数据库操作的逻辑单元
- 这些操作要么全部成功执行，要么全部回滚（撤销）
- 确保数据库的一致性和完整性，并支持并发操作和数据的一致性处理

**ACID特性**

- 原子性A：事务中的所有操作要么全部成功，要么全部回滚，没有部分成功的情况。
- 一致性C：事务的执行将数据库从一个一致的状态转换到另一个一致的状态，不破坏数据库的完整性。
- 隔离性I：事务的执行是相互隔离的，即并发执行的事务互不干扰，每个事务都感觉不到其他事务的存在。
- 持久性D：一旦事务成功提交，其结果将永久保存在数据库中，即使系统故障也不会丢失。

**标志**

- 事务由 BEGIN TRANSACTION 或简写的 START TRANSACTION 语句开始，并以 COMMIT 或 ROLLBACK 语句结束。
- BEGIN 或 START TRANSACTION 标记了一个事务的开始，将所有后续的操作作为一个逻辑单元执行。
- COMMIT 用于提交事务，将所有的操作结果永久保存到数据库中。
- ROLLBACK 用于回滚事务，撤销所有未提交的操作，将数据库恢复到事务开始前的状态。



### 索引

**创建、修改和删除**

```sql
CREATE INDEX index_name ON table_name (column1, column2, ...)
```

```sql
ALTER TABLE table_name ADD INDEX index_name (column)
```

```sql
DROP INDEX index_name ON table_name
```



**类型**

- B-Tree 索引：最常见的索引类型，适用于等值查询和范围查询。
- 哈希索引：适用于等值查询，使用哈希函数将索引键映射到索引位置。
- 全文索引：用于全文搜索，支持文本内容的关键字搜索。
- 空间索引：用于地理位置数据的空间查询，支持地理位置相关的查询。





# 高级SQL



### 函数

**定义**

```sql
# 函数名称，参数名列表和数据类型
CREATE FUNCTION function_name ([parameter1 datatype [, parameter2 datatype [, ...]]])
	# 返回值类型
  RETURNS return_datatype
  # 非重点
  [LANGUAGE { SQL | language_name }]
  [DETERMINISTIC | NOT DETERMINISTIC]
  [SQL DATA ACCESS { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }]
  [COMMENT 'string']
  # 函数定义
  BEGIN
    -- 函数逻辑
    RETURN return_value;
  END;

```

eg.

> 定义一个函数，给定department，返回老师人数

```sql
create function dept_count(dept_name varchar(20))
returns integer
begin
	# 声明变量
	declare d_count integer;
	# 执行查询
	select count(*) into d_count
	from instructor
	where instructor.dept_name=dept_name
	# 返回结果
	return d_count
end
```

使用：

> 找出有12个老师以上的department

```sql
select dept_name
from department
where dept_count(dept_name) > 12 # 就像普通聚合函数一样用
```



**函数也可以返回一个表，使用时函数结果放from后面当普通表用即可**



### 过程

和函数类似。

```sql
create procedure dept_count_procedure(in dept_name varchar(20), out d_count integer)
begin
	select count(*) into d_count
	from instructor
	where instructor.dept_name=dept_name
end

# 过程中也可以是插入、删除等操作，如果查询则需要声明out参数

# 使用

declare d_count integer;
call dept_count_procedure("Comp. Sci.", d_count);
```



### 触发器

**定义**

特定事件发生时自动触发执行



**应用场景**

1. 强制数据完整性：通过在插入、更新或删除数据之前执行检查，触发器可以强制实施业务规则和完整性约束。
2. 日志记录和审计：通过在数据操作发生时记录相关信息，触发器可以用于创建审计日志或记录数据更改历史。
3. 数据复制和同步：通过在源表上创建触发器，在数据插入、更新或删除时同步操作到其他相关表。
4. 自动计算和衍生数据：触发器可以自动计算和更新数据，例如自动更新总计、计算平均值等。



**范式**

```sql
CREATE TRIGGER trigger_name # 触发器名称
{BEFORE | AFTER} {INSERT | UPDATE | DELETE} # 触发时机：before，after；事件：插入、更新、删除
ON table_name	# 触发器关联表
[FOR EACH ROW] # 指示触发器对每行数据都触发
[WHEN (condition)] # 指示触发器执行条件
BEGIN
  -- 触发器逻辑
END;

```

eg.

```sql
# 插入timeslot时检查其id是否在time_slot表里
create trigger timeslot_check 
after insert on section
referencing new row as nrow
for each row
when(nrow.time_slot_id not in (select time_slot_id from time_slot))
begin
	rollback
end

# 在orders表插入数据后自动更新order_summary表的order_count值
create trigger update_order_count
after insert
on orders
for each row
begin
	update order_summary set order_count = order_count + 1
end
```

# 实体-关系模型

### 数据库三级模式

- 外模式：用户视图或子模式，定义了用户或应用程序对数据库的部分可见性
- 模式/概念模式：数据库的中间层，也称为全局模式或逻辑模式。它定义了整个数据库的逻辑结构和组织方式，描述了所有数据之间的关系、实体以及它们之间的约束和规则。
- 内模式：是数据库的最底层，也称为存储模式或物理模式

- 数据库的二级映像：
  - 外模式/模式映像
  - 模式/内模式映像



### 建模

- 数据库可以建模为：
  - 实体的集合
  - 实体之间的关系
- **实体集**是一组具有相同属性的同类实体的集合。

### 关系集

- **关系**是几个实体之间的关联。

- **关系集**是从实体集中取出的 $$n \geq 2$$ 个实体之间的数学关系，其中
  $$
  \{(e_1,e_2,...e_n)\ |\ e_1 \in E_1,e_2 \in E_2,...,e_n \in E_n \}
  $$
  其中 $$(e_1,e_2,...e_n)$$ 是一个关系。

  **例子**

  $$(44553,22222) \in advisor$$

- 属性也可以是关系集的属性之一。



### 关系集的度

- **二元关系**
  - 包含两个实体集（或度为二）
  - 数据库系统中的大多数关系集都是二元的

### 属性

- 实体由一组属性表示，即由实体集的所有成员所具有的描述性属性。

  **例子**

  1. instructor=(ID, name, street, city, salary)
  2. course=(course_id, title, credits)

- **域**：每个属性允许的取值集合

- 属性类型：

  1. 简单和组合属性
  2. 单值和多值属性
  3. 派生属性：根据出生日期计算年龄

### 映射基数约束

- 表示通过关系集可以将另一个实体关联到的实体数量。
- 在描述**二元关系集**时最有用。
- 对于二元关系集：
  - 一对一
  - 一对多
  - 多对一
  - 多对多

### 键

- 实体集的**超键**是一组一个或多个属性，其值可以唯一确定每个实体。
- 实体集的**候选键**是一个最小的超键。
  - *instructor* 的候选键是 *ID*
  - *course* 的候选键是 *course_id*
- 从候选键中选择一个作为**主键**。

### 关系集的键

- 参与实体集的主键的组合形成了关系集的超键。

  - (*s_id, i_id*) 是 advisor 的超键

  - *注意*：这意味着**一对实体集在特定关系集中最多只能有一个关系**。

    例如：如果我们希望跟踪学生与导师之间的多个会议日期，我们不能假设每个会议都有一个关系。但是我们可以使用多值属性。

- 在确定候选键时，必须考虑关系集的映射基数。

### 冗余属性

- 该属性复制了关系中存在的信息，并应从 *instructor* 中删除。

- 但是：在转换回表时，某些情况下会重新引入该属性。



### 带有属性的关系集

<img src='https://z3.ax1x.com/2021/06/24/RMZYVK.png'>

### 角色

<img src='https://z3.ax1x.com/2021/06/24/RMeueP.png'>

### m对n

<img src='https://z3.ax1x.com/2021/06/24/RMehFO.png'>

<img src='https://z3.ax1x.com/2021/06/24/RMeTld.png'>

<img src='https://z3.ax1x.com/2021/06/24/RMuTl4.png'>

### 参与度

- 完全参与（由双线表示）：实体集中的每个实体都参与了至少一个关系集中的关系。
- 部分参与：有些实体可能不参与任何关系集。

<img src='https://z3.ax1x.com/2021/06/24/RMulFK.png'>



### 三元关系

<img src='https://z3.ax1x.com/2021/06/24/RMKinA.png'>



### 三元关系的基数约束

- 我们允许在三元（或更高阶）关系中最多有一条箭头，用来表示基数约束。
- 如果有多于一条箭头，有两种定义含义的方式。
  - A、B和C之间的三元关系R，带有指向B和C的箭头，可能意味着：
    1. 每个A实体与B和C中的唯一实体相关联。
    2. 每对(A, B)实体与唯一的C实体相关联，且每对(A, C)实体与唯一的B实体相关联。
  - 为避免混淆，我们禁止多于一条箭头。

### 弱实体集

- 如果一个实体集没有主键，则称其为**弱实体集**。
- 弱实体集的存在取决于一个**标识实体集**的存在。
  - 它必须通过从标识集到弱实体集的总体、一对多的关系集与标识集相关联。
  - 使用双菱形图示来表示**标识关系**。
- 弱实体集的**鉴别符**(或*部分键*)是区分所有弱实体集实体的属性集合。

<img src='https://z3.ax1x.com/2021/06/24/RMGbod.png'>

- 我们用**虚线**来标记弱实体集的鉴别符。
- 我们将标识关系放在一个**双重**菱形中。
- 课程节的主键为 -(*course_id, sec_id, semester, year*)

### 大学模式ER图

<img src='https://z3.ax1x.com/2021/06/24/RMJKw4.png'>

### 转化为关系模式

- 实体集和关系集可以统一表示为代表数据库内容的关系模式。
- 对于每个实体集和关系集，都有一个唯一的模式，其名称与相应的实体集或关系集相同。
- **1对多**的关系集可以不需要，将1的主键和关系的属性作为多的属性之一
- **1对1**同理，任选一个1当成多
- **多对多**需要把关系作为一个模式，包括两边的主键和关系的属性

### 用简单属性表示实体集

- 强实体集转化为具有相同属性的模式：

  *student(<u>ID</u>, name, tot_cred)*

- 弱实体集变成一个包括标识强实体集主键列的表格：

  *section(<u>course_id, sec_id, semester, year</u>)*

### 表示关系集

- 多对多的关系集表示为一个具有参与实体集的主键属性和关系集的任何描述性属性的模式。

  例如：关系集*advisor*的模式：

  *advisor = (<u>s_id, i_id</u>)*

<img src='https://z3.ax1x.com/2021/06/24/RMUHPJ.png'>

### 模式的冗余性

- 在多对一和一对多的关系集中，如果在"多"一侧是总参与度的，可以通过在"多"一侧添加一个额外的属性，其中包含"一"一侧的主键，来表示这种关系。
- 对于一对一的关系集，可以选择任何一侧作为"多"一侧。
- 如果在"多"一侧的参与度是*部分参与度*，则将模式替换为在对应于"多"一侧的模式中添加一个额外的属性可能导致空值。
- 不需要弱实体集。

### 复合属性和多值属性

- 复合属性通过为每个组成属性创建单独的属性来展开。
- 实体*E*的多值属性*M*通过一个单独的模式*EM*来表示
  - 模式*EM*具有与*E*的主键对应的属性，以及与多值属性*M*对应的属性。

### 将非二元关系转化为二元形式

一般而言，可以通过创建一个人工实体集来使用二元关系来表示任何非二元关系。



<img src='https://z3.ax1x.com/2021/06/24/RMDFD1.png'>



# 关系数据库设计



### 不良设计的问题



- 数据冗余
- 插入异常：开了计算机系，但因为还没招生，计算机系插不进表中
- 删除异常：计算机系学生全部毕业，则删除所有学生的同时会把计算机系删除



### 函数依赖理论



**函数依赖**：只要元组的属性（组）A确定，那么其属性（组）B唯一确定，则$A\rightarrow B$。

A比作员工，B比作部门，则员工号$\rightarrow$部门名。任何两个元组，员工号相同，则部门名一定相同。

所以A种类≥B种类，A到B是树的叶子到根。

- **平凡**函数依赖：B是A的子集

- **完全**函数依赖：A是属性组，只有A中所有属性才能唯一确定B，A的任何子集都不行
- **部分**函数依赖：相反，A中部分属性可以确定B
- **传递**函数依赖：$A\rightarrow B$，$B\rightarrow C$，则$A\rightarrow C$
- **码**：一个属性（组），关系中其余属性都**完全函数依赖**于它
  - 如树状结构的叶子，员工号确定了，其部门、经理、总管等一系列上层都能确定
  - 找关系中的码，需要1个属性、2个属性...全部来一遍，但是如果已知属性组的子集是码则不需要考虑了，因为码需要完全函数依赖
  - **主属性**：所有码中包含的属性
  - **非主属性**：除了码中包含的属性外的属性

**Armstrong公理**

- 自反：$若\beta \subseteq \alpha，则\alpha \rightarrow \beta$(平凡函数依赖)
- 增补：$若\alpha \rightarrow \beta，则\gamma\alpha \rightarrow \gamma \beta$（两边增加一组属性也依赖）
- 传递：不解释，但注意$\alpha \rightarrow \beta，\beta \rightarrow \gamma$外还需满足$\alpha 不能决定 \beta$。因为防止$\alpha 和 \beta$一一对应，没有传递依赖。
- 合并：左侧相同，可以合并成左侧——>右侧的并集
- 分解：合并的逆运算
- 伪传递：$若\alpha \rightarrow \beta且\gamma \beta \rightarrow \delta，则\alpha \gamma \rightarrow \delta$(第一个条件用增补，然后传递)



用这些推导可以求**依赖集F的闭包**、**属性集合的闭包**



### 第一范式

> 数据库表每一列都是不可分割的**原子数据项**。

如果表中有一列：系，下面有包含两个属性：系名，系主任，那么要把系去掉，系名和系主任作为两个原子属性存在。







### 第二范式

> 在1NF基础上，消除非主属性对主属性的**部分函数依赖**。

原：（学号，姓名，系名，系主任，课名，分数）

（学号，课名）是关系的码，它们唯一确定其他属性。

但（学号，课名）$\rightarrow $ （姓名，系名，系主任）的同时，学号$\rightarrow $ （姓名，系名，系主任）。

因此**这些依赖关系对主属性是部分函数依赖**。

分解：（学号，课名，分数），（学号，姓名，系名，系主任）。

分解方法依然是有问题的分离出去，把部分函数依赖分离成单独的关系，剩下的属性就不存在对主属性的部分函数依赖了。

- 数据冗余：很好地减少了冗余
- 插入异常：未解决
- 删除异常：未解决



### 第三范式

> 函数依赖a—>b满足其中一项：1. 平凡，即b属于a 2. a是关系的超码 3.b-a中的每个属性A都包含于R的一个候选码

> 在2NF基础上，消除非主属性对主属性的**传递函数依赖**。

第二范式分解后的表：

- （学号，课名，分数）

依赖关系：（学号，课名）$\rightarrow $ 分数

**因为只有一个函数依赖，不可能产生传递函数依赖。**



- （学号，姓名，系名，系主任）

依赖关系：学号$\rightarrow $ （姓名，系名，系主任）

系主任对姓名传递函数依赖，因为学号$\rightarrow $ 姓名，姓名$\rightarrow $ 系名，所以学号$\rightarrow $ 系主任。



分解：（学号，姓名，系名），（系名，系主任）

此时的依赖关系：学号$\rightarrow $ 姓名，学号$\rightarrow $ 系名，系名$\rightarrow $ 系主任。不存在传递函数依赖。



- 插入异常：得到改善
- 删除异常：得到改善



### BC范式

> 函数依赖a—>b满足其中一项：1. 平凡，即b属于a 2. a是关系的超码

> 在3NF基础上，消除**主属性对主属性**的部分函数依赖和传递函数依赖。

原：（仓库名，管理员，物品名，数量）

码：（仓库名，物品名），（管理员，物品名）

两个码都可以确定唯一的非主属性数量。

这里物品名不是非主属性，因为同一种物品可以放在不同仓库（但一个学生不能在不同学院），所以物品名是主属性。

满足第二范式：数量对主属性没有部分函数依赖

满足第三范式：数量对主属性没有传递函数依赖

存在的问题：

- 删除异常：删掉仓库中所有种类物品后，仓库也要删除
- 插入异常：新建仓库，没有物品，则仓库插不进，也不能分配管理员
- 修改异常：仓库更换管理员，需要逐条记录修改



存在主属性对主属性的部分函数依赖：（管理员，物品名）$\rightarrow $ 仓库，但其实管理员$\rightarrow $仓库

分解：仓库关系（仓库名，管理员），库存关系（仓库名，物品名，数量）

问题解决：

- 删除：仓库有专门的表，删除物品不会再导致仓库被删除
- 插入：新建的仓库插入仓库表，也可以分配管理员
- 修改：只需要在仓库表修改一次管理员

### 第四范式

> 第四范式定义：满足1NF的前提下，关系每个非平凡多值依赖X$\rightarrow $ $\rightarrow $ Y，X都含有码，则为4NF。
>
> X含有码，则X能唯一确定Y，多值依赖成为函数依赖，所以说4NF允许的非平凡多值依赖是函数依赖。

> 多值依赖X, Y, Z是U子集，Z=U-X-Y，X$\rightarrow $ $\rightarrow $ Z当且仅当给定（X，Y）能确定Z的**一组值**，但这个值仅由X，而与Y无关。如果Y为空集，则X$\rightarrow $ $\rightarrow $ Z是平凡的。
>
> 一组值改为一个值，就成了函数依赖。

原：课程C，教师T，书B，每门课有不同老师、老师可以教不同课程，每个老师有不同书、不同老师可以有相同的书。

满足1NF原子数据。不存在函数依赖，C、T、B都是主属性，满足2NF、3NF、BCNF。

问题：

- 冗余
- 插入：一门课增加一个老师，要插入多个元组（多本书）
- 删除：一个老师删除一本书，要删除多条元组

存在多值依赖：给定C和B，可以确定一组T，但T与B无关，只与C有关。即C$\rightarrow $ $\rightarrow $ T。



分解：

TC，BC。三个问题都改善。



### 候选码

关系模式R，依赖集F

- L：只出现在F左侧的属性
- R：只出现在F右侧的属性
- LR：出现在两边的属性
- NON：没有出现在F中的属性

求候选码集合：

1. L和NON构成并集X，如果X的闭包是U，则X为候选码，完毕，否则执行2
2. 对LR中的每个属性A，如果AX的闭包为U，则AX加入候选码集合，且LR-A
3. 对LR中的每两个属性Z，如果ZX的闭包为U，则ZX加入候选码集合。这里要注意**LR不需要去掉Z**，且要检查**ZX是否包含了其他候选码**，这种不能加入。
4. 每三个。。。





### 正则覆盖

> 依赖集F的正则覆盖Fc也是一个依赖集，Fc逻辑蕴含F中所有依赖
>
> 求Fc就是把F进行简化、提纯的过程。
>
> Fc特点：1.函数依赖不含无关属性 2.左半部分唯一，不能重复



**步骤：**

- 用合并律把左部相同的函数依赖合并
- 合并后在函数依赖集中找一个**无关属性**，删除
- 重复1、2直到集合不变化



**无关属性：**

- 如果要判断的属性在右侧，则在**删除后的F'中**，计算该依赖的左部集合闭包$\alpha +$ ，如果其中包含**要判断的属性**，则其无关
  - 即：F'这个依赖的左部集合的闭包中已经有该无关属性了，所以不需要出现在右侧，删除后也可以通过闭包推出来
  - 例：$A\rightarrow BC$，$C\rightarrow AB$，则第一个依赖的B可以删除，因为删除后可以在F'中计算出A的闭包含有B（通过C的传递依赖）
- 如果要判断的属性在左侧，则在删除前的F中，计算该依赖左侧集合（要判断的属性除外）的闭包$\alpha +$ ，如果其中包含**右侧所有属性**，则其无关
  - 即：在F中，左侧这些属性已经可以唯一确定右侧所有属性了，就不需要再有这个多余的属性
  - 例：F{Z→X，X→P，XY→WP，XYP→ZW}，考虑最后一个依赖的P，现在在F中求XY的闭包，XY→XYWP→ XYWPZ，包含ZW，所以P是无关属性



### 分解算法



**无损连接分解**

- 分解后的关系用自然连接即可完全还原

- 条件：分解后2个子模式的属性的交集，是R1或R2的码
- 即当且仅当下面至少一个属于F+：
  - $R_1 \cap R_2 \rightarrow R_1$
  - $R_1 \cap R_2 \rightarrow R_2$
- 表格法
  - 构造初始表格：行为每个关系的属性集，列为每个属性
  - 先填满b，下标为b的行列坐标
  - 再填A，行的属性集包含的属性依次填a，下标为属性序号
  - 对每个函数依赖，把b改成a（每行都要改，即依赖左边为a的，把右侧改为a）
  - 重复上一步，直到出现一行全a，则证明了lossless
  

> 已知关系模式$ R<U, F>$，$U=\{A, B, C, D, E, G\}，F=\{ A→B, C→A, CD→E, D→G\}$，现有一个分解$\rho=\{AB, AC, CDE, DG\}$，请判断该分解是否具有 无损连接性，并给出判断依据和判断过程。

- 构造初始表

|      | A        | B        | C        | D        | E        | G        |
| ---- | -------- | -------- | -------- | -------- | -------- | -------- |
| AB   | $a_1$    | $a_2$    | $b_{13}$ | $b_{14}$ | $b_{15}$ | $b_{16}$ |
| AC   | $a_1$    | $b_{22}$ | $a_3$    | $b_{24}$ | $b_{25}$ | $b_{26}$ |
| CDE  | $b_{31}$ | $b_{32}$ | $a_3$    | $a_4$    | $a_5$    | $b_{36}$ |
| DG   | $b_{41}$ | $b_{42}$ | $b_{43}$ | $a_4$    | $b_{45}$ | $a_6$    |



- 由 A→B，有 $b_{22}$ 改为 $a_2$;由 C→A，有 $b_{31}$ 改为 $a_1$;CD→E 表中无变化;由 D→G，有 $b_{36}$ 改为 $a_6$;则得到变化后的中间表格

|      | A        | B        | C        | D        | E        | G        |
| ---- | -------- | -------- | -------- | -------- | -------- | -------- |
| AB   | $a_1$    | $a_2$    | $b_{13}$ | $b_{14}$ | $b_{15}$ | $b_{16}$ |
| AC   | $a_1$    | $*a_2$   | $a_3$    | $b_{24}$ | $b_{25}$ | $b_{26}$ |
| CDE  | $*a_1$   | $b_{32}$ | $a_3$    | $a_4$    | $a_5$    | $*a_6$   |
| DG   | $b_{41}$ | $b_{42}$ | $b_{43}$ | $a_4$    | $b_{45}$ | $a_6$    |



- 再由 A→B，有 $b_{32}$ 改为 $a_2$

|      | A        | B        | C        | D        | E        | G        |
| ---- | -------- | -------- | -------- | -------- | -------- | -------- |
| AB   | $a_1$    | $a_2$    | $b_{13}$ | $b_{14}$ | $b_{15}$ | $b_{16}$ |
| AC   | $a_1$    | $*a_2$   | $a_3$    | $b_{24}$ | $b_{25}$ | $b_{26}$ |
| CDE  | $*a_1$   | $**a_2$  | $a_3$    | $a_4$    | $a_5$    | $*a_6$   |
| DG   | $b_{41}$ | $b_{42}$ | $b_{43}$ | $a_4$    | $b_{45}$ | $a_6$    |



- 此时第三行全为a，说明lossless



**保持函数依赖**

1. 将原始关系模式的函数依赖集合记为F，将分解后的关系模式的属性集合记为R1, R2, ..., Rn。
2. 对于每个函数依赖X -> Y ∈ F，检查分解后的关系模式中是否存在包含X的关系模式和包含Y的关系模式，即Ri 和 Rj（i ≠ j）。
3. 如果在Ri 和 Rj 中可以找到包含X的属性集合和包含Y的属性集合，那么说明该分解保持函数依赖

**BCNF分解**

- 初始化result为R
- 计算F+（BCNF使用的是F+，不是Fc）
- 找result中不属于BCNF的Ri，令$\alpha \rightarrow \beta$ 是Ri上一个非平凡函数依赖，$\alpha$ 不是Ri的主码（$\alpha \rightarrow R_i$ 不属于F+），且$\alpha \cap \beta$为空集
- 把Ri分解成Ri-$\beta$ 和（$\alpha, \beta$）
- 重复直到每个Ri都属于BCNF且分解是无损连接
- BCNF可能不保持依赖
- 伪代码：

```python
result = {R}
F+ = F的闭包
while result中存在模式Ri不属于BCNF:
    令a->b是Ri上一个非平凡函数依赖，有a->Ri不属于F+(a不是候选码)，且a交b为空（非平凡）
    分解为(Ri - b) 和 (a, b)两个模式
```



eg.

> 假定我们有一个数据库设计使用了以下的class模式:
>  class (course_id, title, dept_name, credits, sec_id, semester, year, building, room_number,
>
> capacity, time_slot_id)
>
> 我们要求在class上成立的函数依赖集为:
>  course_id→ title, dept_name, credits
>  building, room_number→capacity
>  course_id, sec_id, semester, year→building, room_number, time_slot_id
>
> 请使用BCNF分解算法对class进行分解。

```
从函数依赖集可以看出，
{course_id, sec_id, semester, year}是class模式上的超码，而{building, room_number}和 {course_id}不是class模式上的超码，course_id→ title, dept_name, credits和building, room_number→capacity也不是平凡的函数依赖。
由BCNF分解算法知，对于course_id→ title, dept_name, credits，可以将class分解为 course(course_id, title, dept_name, credits)
class_1(course_id, sec_id, semester, year, building, room_number, capacity, time_slot_id) 对于building, room_number→capacity，可以进一步将class_1分解为
classroom(building, room_number, capacity)
section(course_id, sec_id, semester, year, building, room_number, time_slot_id) 此时所得的三个模式满足BCNF范式，故最终的分解为:
course(course_id, title, dept_name, credits)
classroom(building, room_number, capacity)
section(course_id, sec_id, semester, year, building, room_number, time_slot_id)
```

例：

> r(A,B,C,D,E,F)
>
> F=(A->BCD,BC->DE,B->D,D->A)
>
> 初始化为R(A,B,C,D,E,F)
>
> 对A->BCD分解一次，找到未在依赖集出现的F属性构成(A,F)：
>
> BCNF分解：(A,B,C,D)(A,E)(A,F)

**3NF分解**

- 初始化result为空集

- 计算Fc（3NF只需要Fc）
- 对Fc每个函数依赖$\alpha \rightarrow \beta$，$\alpha \beta$作为一个关系，R加入result
- 如果每个关系都不包含R的候选码，就新增一个关系为R的任意候选码，加入result（**有了候选码，就可以无损连接分解**）
- 删除所有包含于另一个模式中的模式（**这一步很重要，3NF分解会生成很多模式，每个FD一个，最后可能要合并很多次**）
- 伪代码：

```python
Fc = F的正则覆盖

# 根据函数依赖进行分解，逐步从空集构造模式集合
i = 0
for 函数依赖a->b in Fc:
  if 没有模式包含ab:
    i++
    Ri = (a, b)

# 候选码需要包含在模式中
if 没有模式包含R的候选码:
    i++
    Ri = R的任意候选码
    
# 删除包含于另一个模式中的模式
while 还有包含于另一个模式中的模式:
  if Rj 包含于另一个模式 Rk 中:
    Rj = Ri
    i--
    
return (R1, ..., Ri)
```



> 设工厂里有一个记录职工每天日产量的关系模式: R(职工编号，日期，日产量，车间编号，车间主任)。
>
> 如果规定:每个职工每天只有一个日产量;每个职工只能隶属于一个车间;每个车间只有一 个车间主任。分析R是否达到3NF。如果达到，请逐条对应3NF的定义进行验证说明;如果没 有达到，则对其进行分解，使分解后的关系模式达到3NF。

```
根据题目描述，在R上成立的函数依赖集为:
职工编号，日期 → 日产量
职工编号 → 车间编号
车间编号 → 车间主任
容易导出函数依赖：
职工编号，日期 → 职工编号，日期，日产量，车间编号，车间主任
故{职工编号，日期}是R上的超码，第一条函数依赖满足3NF。
而职工编号或车间编号不是R上的超码，且车间编号及车间主任不在R上的候选码中，故R未 达到3NF。
对R进行分解，将R分解为: 
R1(职工编号，日期，日产量) 
R2(职工编号，车间编号)
R3(车间编号，车间主任)
此时R1、R2和R3满足3NF。
```

例：

> r(A,B,C,D,E,F)
>
> Fc=(A->BC,B->DE,D->A)
>
> 初始化为空集
>
> 每个依赖构成一个R，最后加上F
>
> 3NF分解：(A,B,C)(B,D,E)(A,D)(A,F)

# 存储和文件结构

### 物理存储介质的分类

可以将存储介质区分为以下两类：

- 易失性存储介质
- 非易失性存储介质

### 物理存储介质

- 缓存（Cache）：最快且成本最高的存储形式；易失性；由计算机系统硬件管理。
- 主内存（Main memory）：
  - 访问速度快
  - 通常容量较小
  - 易失性
- 闪存（Flash memory）：
  - 数据在断电后仍然保留
  - 读取速度与主内存大致相同
  - 写入速度较慢（几微秒），擦除更慢
- 磁盘（Magnetic disk）：
  - 数据存储在旋转的盘片上，并通过磁性进行读写
  - 数据需要从磁盘移到主内存进行访问，然后写回磁盘进行存储
  - **直接访问**
- 光学存储（Optical storage）：
  - 非易失性，数据通过激光从旋转的光盘上读取
- 磁带存储（Tape storage）：
  - 非易失性
  - **顺序访问**

### 存储层次结构

- 主存储器（Primary storage）：最快的介质，但易失性
- 辅助存储器（Secondary storage）：非易失性，访问时间适中
- 第三级存储器（Tertiary storage）：非易失性，访问时间较慢

### 磁硬盘机制

### 硬盘性能指标

- 访问时间：从发出读取或写入请求到数据传输开始所需的时间
  - 寻道时间
  - 旋转延迟
- 数据传输速率
- 平均故障时间

### 磁盘块访问的优化

- 块（Block）：来自同一磁道的连续扇区序列
  - 数据在磁盘和主内存之间以块为单位进行传输
- 数据臂调度
- 文件组织：通过组织块以对应数据访问方式，优化块的访问时间

### RAID

- **独立磁盘冗余阵列**（Redundant Arrays of Independent Disks）

### 文件组织

数据库以一组*文件*的形式存储。每个文件是一系列*记录*的序列。每个记录是一系列字段的序列。

### 定长记录

### 空闲列表

### 变长记录



# 索引

### 基本概念

- **搜索键**：用于在文件中查找记录的属性或属性集。
- **索引文件**由以下形式的记录（称为**索引项**）组成：

- 两种基本类型的索引：

  - **有序索引**
  - **哈希索引**
- 优点
  - 加快数据的检索速度
  - 创建唯一性索引，保证数据库表中每一行数据的唯一性;
  - 加速表和表之间的连接
  - 在使用**分组**和排序子句进行数据检索时，可以显著减少时间。
  
- 缺点
  - 占用数据表以外的物理存储空间
  - 创建索引和维护索引要花费一定的时间
  - 当对表进行更新操作时，索引需要被重建，这样降低了数据的维护速度。
  

### 有序索引

在**有序索引**中，索引项按搜索键值排序存储。

- **主索引**：在顺序有序文件中，其搜索键指定文件的顺序。
  - 也称为**聚集索引**
  - 主索引的搜索键通常是但不一定是主键。
- **次索引**：其搜索键指定与文件的顺序不同的顺序的索引。也称为**非聚集索引**。

- 主索引是对数据库表中的主键（Primary Key）进行建立的索引。主键是表中用于唯一标识每个记录的字段或字段组合。主索引是基于主键构建的索引，它确定了表中记录的物理存储顺序。主索引的特点是唯一性和有序性，它使得根据主键值进行检索和排序变得高效。通常情况下，数据库表只能有一个主索引。

- 辅助索引是基于非主键字段构建的索引。它用于加快对非主键字段的检索速度。辅助索引可以是单个字段或多个字段的组合，不同于主索引，它并不决定表中记录的物理存储顺序。辅助索引允许根据非主键字段的值快速定位到符合条件的记录。在一个表中可以有多个辅助索引，它们可以覆盖多个字段或满足不同的查询需求。

> 辅助索引并不适合用于顺序扫描，因为它们是为了加速对特定字段的检索而设计的，而非用于顺序访问整个表。
>
> 当使用辅助索引进行顺序扫描时，实际上是通过辅助索引来逐个定位记录，并根据辅助索引中的排序顺序来访问表中的记录。这种方式效率较低的原因主要有两个：
>
> 1. 辅助索引通常是根据特定字段进行排序的，而不是按照物理存储顺序排序。因此，在顺序扫描时，需要频繁地跳转到辅助索引指向的记录位置，而不是按照连续的物理块读取数据，这会增加磁盘I/O操作的次数，降低访问效率。
> 2. 顺序扫描时，需要遍历整个表的记录，而辅助索引只能提供对单个字段的快速定位，而无法提供连续记录的快速访问。这种逐个定位记录的过程会增加额外的索引查找操作，导致效率下降。
>
> 相比较而言，针对顺序扫描操作，使用主索引（如果主键是按照物理存储顺序排序的）或者直接对表进行顺序访问会更高效，因为它们可以直接按照物理存储顺序连续地读取数据，减少了额外的索引查找和跳转开销。
>
> 因此，辅助索引的主要应用场景是在针对特定字段的查询和范围查询上，而不是用于顺序扫描整个表。对于需要进行顺序访问的操作，最好直接对表进行全表扫描或使用主索引来提高效率。

- 索引顺序文件：带有主索引的有序顺序文件。

eg.

>什么是主索引?什么是辅助索引?使用辅助索引进行顺序扫描，效率高吗?
>
>您的答案:
>
>主索引也叫聚集索引，是指在与顺序存储文件的存储顺序一致的搜索码上建立的索引。
>辅助索引是指建立索引的搜索码的顺序与文件的存储顺序不一致的索引。
>使用辅助索引进行顺序扫描的效率不高，因为建立索引的搜索码的顺序与文件的存储顺序不
>一致，从而需要反复读取索引中的数据指针和存储的数据文件，其效率远低于在主索引上进
>行的顺序扫描。

### 稠密索引文件

**稠密索引**：文件中的每个搜索键值都有索引记录。

### 疏散索引文件

**疏散索引**：仅包含某些搜索键值的索引记录。

- 当记录按搜索键顺序排序时适用

要定位具有搜索键值*K*的记录，我们可以：

- 查找具有小于*K*的最大搜索键值的索引记录
- 从索引记录指向的记录开始顺序搜索文件

与稠密索引相比：

- 占用更少的空间，插入和删除的维护开销较小。
- 通常比稠密索引更慢以定位记录。

**良好的折衷方案**：稀疏索引中的每个块都有一个索引项，对应于块中的最小搜索键值：

### 二级索引示例

- 索引记录指向包含该特定搜索键值的所有实际记录的**桶（bucket）**。
- 次索引必须是稠密的。

### 多值索引

- 如果主索引不适合内存中，访问变得昂贵。
- 解决方案：将保存在磁盘上的主索引视为顺序文件，并在其上构建稀疏索引。
  - 外部索引：主索引的稀疏索引
  - 内部索引：主索引文件

### 索引更新：删除

单级索引条目删除：

- 稠密索引：删除搜索键与文件记录的删除类似
- 疏散索引：
  - 如果索引中存在搜索键的条目，则通过用文件中的下一个搜索键值替换索引中的条目来删除该条目
  - 如果下一个搜索键值已经具有索引条目，则删除该条目而不是替换它。

### 索引更新：插入

单级索引

插入：

- 稠密索引：如果索引中不存在搜索键值，则插入它。
- 疏散索引：如果索引存储了文件的每个块的条目，则不需要对索引进行任何更改，除非创建了新块。
  - 如果创建了新块，则将插入新块中出现的第一个搜索键值到索引中。

### 次索引

我们可以为每个搜索键值设置一个次索引记录。

### B+树索引文件

$$B^+$$-树是一个满足以下属性的根树：

- 从根到叶子的所有路径长度相同。
- 每个非根非叶子节点具有$$n/2$$到$$n$$个子节点。
- 叶子节点具有$$(n-2)/2$$到$$(n-1)$$个值。
- 特殊情况：
  - 如果根不是叶子，则它至少有2个子节点。
  - 如果根是叶子（即树中没有其他节点），它可以有0到$$(n-1)$$个子节点。



### B+树操作

- 查询
  - 从根节点进行对比，根据对比结构决定去哪个子节点
  - 每次查找访问磁盘次数固定，都要找到叶子上
  - B树则不稳定，类似普通搜索树
  - 区间查询：先单个元素查询到区间下界，然后在叶子上顺序查找
- 插入
  - 若插入节点的关键字数目未满，则直接插入
  - 若已满，则分裂为 `⌊M/2⌋` 和 `⌈M/2⌉`两部分，将`⌈M/2⌉`的关键字上移至其双亲结点。如阶M=3，则插入后分裂为2、2两部分，后面部分的最小元素插入到父节点（假设索引是子节点中最小元素，如果是最大元素，则把前面部分的最大元素插入父节点）
  - 对父节点的插入同理
  - 如果插入值为节点最大/最小，破坏了索引结构，则需要更新父节点中的索引元素
- 删除

### $$B^+$$-树节点结构

典型的节点：

- $$K_i$$ 是搜索键值
- $$P_i$$ 是指向子节点（对于非叶子节点）或指向记录或记录桶（对于叶子节点）的指针
- 节点中的搜索键按顺序排列：
  $$
  K_1<K_2<K_3<...<K_{n-1}
  $$

### $$B^+$$-树中的叶子节点

叶子节点的属性：

### $$B^+$$-树中的非叶子节点

非叶子节点在叶子节点上形成多级稀疏索引。对于具有n个指针的非叶子节点：

- 指向$$P_1$$子树中的所有搜索键小于$$K_1$$的搜索键
- 对于$$2 \leq i \leq n-1$$，指向$$P_i$$指向的子树中的所有搜索键值大于或等于$$K_{i-1}$$且小于$$K_i$$
- 指向$$P_n$$指向的子树中的所有搜索键值大于或等于$$K_{n-1}$$

### B树索引文件

- B树只允许搜索键值出现一次；消除了搜索键的冗余存储。
- 非叶子节点中的搜索键在B树中没有其他位置出现；必须为非叶子节点中的每个搜索键包括一个附加的指针字段。
- 用于索引的 B-树 存在缺陷，它的所有中间结点均存储的是数据指针（指向包含键值的磁盘文件块的指针），与该键值一起存储在B-树的结点中。这就会导致可以存储在 B-树中的结点目数极大地减少了，从而增加 B-树的层数，进而增加了记录的搜索时间
- B+树通过仅在树的叶子结点中存储数据指针而消除了上述缺陷
- B+树把叶子节点通过指针串起来，帮助 B+树快速访问磁盘记录。相比于 B-树的中序遍历，B+树只需要像遍历单链表一样扫描一遍叶子结点。
- B+树进行区间查找时更加简便实用。

### 静态哈希

**桶**是包含一个或多个记录的存储单元（桶通常是一个磁盘块）。

在**哈希文件组织**中，我们使用**哈希函数**根据其搜索键

值直接获取记录的桶。



# 事务

### 事务概念

**事务**是一次程序执行的*单元*，它访问并可能更新各种数据项。

### ACID属性

为了保持数据的完整性，数据库系统必须确保：

- **原子性**。事务的所有操作要么正确反映在数据库中，要么都不反映在数据库中。
- **一致性**。隔离地执行事务可以保持数据库的一致性。
- **隔离性**。尽管多个事务可能同时执行，但每个事务必须对其他同时执行的事务不可见。
  - 也就是说，对于每对事务$$T_i$$和$$T_j$$，$$T_i$$看起来要么在$$T_i$$开始执行之前完成了执行，要么在$$T_i$$完成执行之后开始执行。
  - **给数据写加上互斥锁，互斥锁只在事务T提交后才释放，否则可能因为回滚而造成脏读**
- **持久性**。成功完成事务后，它对数据库所做的更改将持久存在，即使发生系统故障。

### 数据读取问题

- **脏读**
  - 事务A对数据修改但未提交，事务B进行读，然后事务A回滚，此时B读到脏数据
- **不可重复读**
  - 事务A对同一数据的两次前后读，因为事务B在间隔中对数据修改了，所以A两次读结果不一样
- **幻读**
  - 和不可重复读类似，但是是B新增了数据，A前后读的数据量不一样
  - 两者区别：**不可重复读**是读取了其他事务修改的数据，针对update；**幻读**是读取了其他事务新增的数据，针对insert和delete
    - 不可重复读：使用**行级锁**，锁定该行，事务A多次读取操作完成后才释放该锁，这个时候才允许其他事务更改刚才的数据
    - 幻读：使用表级锁，锁定整张表，事务A多次读取数据总量之后才释放该锁，这个时候才允许其他事务新增数据

### 隔离性强度

- 串行化(Serializable，SQLite默认模式）：最高级别的隔离。两个同时发生的事务100%隔离，每个事务有自己的"世界", 串行执行。
- 可重复读（Repeatable read，MySQL默认模式）：如果一个事务成功执行并且添加了新数据(事务提交)，这些数据对其他正在执行的事务是可见的。但是如果事务成功修改了一条数据，修改结果对正在运行的事务不可见。所以，事务之间只是在新数据方面突破了隔离，对已存在的数据仍旧隔离。
- 读取已提交（Read committed，Oracle、PostgreSQL、SQL Server默认模式）：可重复读+新的隔离突破。如果事务A读取了数据D，然后数据D被事务B修改（或删除）并提交，事务A再次读取数据D时数据的变化（或删除）是可见的。这叫不可重复读（non-repeatable read）。
- 读取未提交（Read uncommitted）：最低级别的隔离，是读取已提交+新的隔离突破。如果事务A读取了数据D，然后数据D被事务B修改（但并未提交，事务B仍在运行中），事务A再次读取数据D时，数据修改是可见的。如果事务B回滚，那么事务A第二次读取的数据D是无意义的，因为那是事务B所做的从未发生的修改（已经回滚了嘛）。这叫脏读（dirty read）。

|          | 脏读 | 不可重复读 | 幻读 |
| -------- | ---- | ---------- | ---- |
| 读未提交 | yes  | yes        | yes  |
| 读已提交 | no   | yes        | yes  |
| 可重复读 | no   | no         | yes  |
| 串行化   | no   | no         | no   |



### 事务状态

- 活动状态
- 部分提交状态
- 失败状态
- 中止状态
- 已提交状态

### 日程

**日程**：是一系列指令，指定并发事务的指令按时间顺序执行的顺序。

### 可串行化性

如果一个日程等价于一个串行日程，那么它就是可串行化的。不同形式的日程等价性产生了以下概念：

- 冲突可串行化
- 视图可串行化

### 简化的事务视图

我们简化的日程只包含**读取**和**写入**指令。

### 冲突指令

事务$$T_i$$和$$T_j$$的指令$$I_i$$和$$I_j$$，如果存在某个被$$I_i$$和$$I_j$$同时访问的项目*Q*，并且这些指令中至少有一条写入了*Q*，则它们**冲突**。

### 冲突可串行化

如果一个日程*S*可以通过一系列不冲突的指令转换为一个日程*S'*，我们称*S*和*S'*是**冲突等价**的。

如果一个日程*S*与一个串行日程冲突等价，那么我们称*S*是**冲突可串行化**的。

### 视图可串行化



### 可恢复的日程

如果事务*Tj*读取了事务*Ti*之前写入的数据项，则*Ti*的提交操作出现在*Tj*的提交操作之前。

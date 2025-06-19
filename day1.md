# 📅 Day 01 刷题总结

## ✅ 题目：175. Combine Two Tables

### 📝 题目描述

将两个表（Person 和 Address）通过 PersonId 进行连接，输出所有人的姓、名、城市和州。若某人无地址信息，则城市和州返回 NULL。

### 💡 解题思路

- 使用 `LEFT JOIN` 是关键，确保 **Person 表的每一行都被返回**。
- Address 表若没有对应的 personId，会自动补 NULL。
- `ON` 条件连接 PersonId。

### ✅ SQL 关键点

- `LEFT JOIN`：保留左表（Person）所有数据。
- 字段选择顺序：firstName, lastName, city, state。

SELECT Person.FirstName,Person.LastName,Address.City,Address.state 
FROM Person
left JOIN Address ON Person.PersonId = Address.PersonId;

### 🧠 收获总结
-`LEFT JOIN` 常用于"包含所有A，无论是否有B"的场景


## ✅ 题目：176. Second Highest Salary

### 📝 题目描述

查询并返回 Employee 表中第二高的 **不同** 薪水。
- 若不存在第二高的薪水，应返回 `null`。

### 💡 解题思路

- 使用 `DISTINCT` 排除重复薪水。
- 方法一：利用 `LIMIT` 和 `OFFSET` 精准定位到第二高薪。
- 方法二：用子查询先求最大薪水，再找小于最大薪水中的最大值。

### ✅ SQL 解法一（OFFSET）

```sql
SELECT 
    (SELECT DISTINCT salary 
     FROM Employee 
     ORDER BY salary DESC 
     LIMIT 1 OFFSET 1) AS SecondHighestSalary;
```

- `DISTINCT salary`：去重薪水。
- `ORDER BY salary DESC`：从高到低排序。
- `OFFSET 1 LIMIT 1`：跳过第一高薪，取第二高。

### ✅ SQL 解法二（子查询）

```sql
SELECT MAX(salary) AS SecondHighestSalary 
FROM Employee 
WHERE salary < (SELECT MAX(salary) FROM Employee);
```

- 子查询先求出全表最大薪水。
- 外层查询中，排除最大值，再取剩余中的最大值。
- 若所有薪水都一样或仅一位员工，该查询会返回 NULL。

### 🧠 收获总结

- 熟悉了 `LIMIT + OFFSET` 在排序中定位某一条记录的技巧。
- 理解了子查询配合聚合函数的灵活用法。
- 本题是对 SQL 排序、去重与条件筛选的综合考察。

---

## ✅ 题目：177. Get Nth Highest Salary

### 📝 题目描述

编写一个解决方案查询 Employee 表中第 **n** 高的 **不同** 薪水。
- 如果少于 **n** 个不同薪水，查询结果应为 `null`。
- 返回列名为 `getNthHighestSalary(n)`。

示例：
```
Employee:
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
n = 2
输出:
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
```

### 💡 解题思路

- **方案一（LIMIT + OFFSET）**：对去重后的薪水排序，使用 `LIMIT 1 OFFSET n-1` 定位第 n 条。
- **方案二（窗口函数 / 子查询）**：使用窗口函数 `DENSE_RANK()` 计算排名；或通过子查询多层嵌套获取第 n 大。

### ✅ SQL 解法一（LIMIT + OFFSET）

```sql
-- MySQL / SQLite 写法
SELECT (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET n-1
) AS getNthHighestSalary(n);
```

- `DISTINCT salary`：去重。
- `ORDER BY salary DESC`：降序排列。
- `LIMIT 1 OFFSET n-1`：跳过前 n-1 个，取第 n 个。

### ✅ SQL 解法二（窗口函数：DENSE_RANK）

```sql
-- 支持窗口函数的数据库（如 MySQL 8+, SQL Server, PostgreSQL）
SELECT salary AS getNthHighestSalary(n)
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rk
    FROM Employee
) t
WHERE rk = n;
```

- `DENSE_RANK()`：对不同薪水进行连续排名。
- 在外层 `WHERE` 中筛选排名等于 n 的行。
- 若不存在，则结果集为空，可在客户端返回 `null`。

### 🧠 收获总结

- 掌握了利用 `LIMIT+OFFSET` 定位任意排名的方法。
- 熟悉窗口函数 `DENSE_RANK()` 的用法，更具通用性。
- 深化了对去重、排序和排名的综合理解。

---

---

✅ **题目：178. Rank Scores**
📝 **题目描述**
对 Scores 表中的每个分数 score，计算其排名 rank，排名规则是分数越高排名越靠前，分数相同时排名相同。
输出 score 和对应 rank，按 score 降序排序。

💡 **解题思路**
使用窗口函数 RANK() OVER (ORDER BY score DESC) 给分数排序，自动处理相同分数并列排名。

✅ **SQL 关键点**

* 窗口函数 RANK() 用于带并列排名。
* ORDER BY score DESC 控制排名从高分开始。
* 输出字段顺序：score, rank。

```sql
SELECT score, RANK() OVER (ORDER BY score DESC) AS rank
FROM Scores;
```

🧠 **收获总结**

* 熟练使用窗口函数实现排名。
* 理解排名函数（RANK、DENSE\_RANK、ROW\_NUMBER）区别。
* 能够解决带并列名次的排序问题。

---

明白了！下面是按你指定的标准格式，重新整理后的 **180. Consecutive Numbers** SQL 题解：

---

✅ **题目：180. Consecutive Numbers**
📝 **题目描述**
表 `Logs` 包含两个字段：`id`（主键，自增）和 `num`（数字）。
找出所有在表中**至少连续出现三次**的相同数字 `num`。

💡 **解题思路**
使用窗口函数 `LAG` 取出当前行前两行的 `num` 值，判断是否三行都是同一个值，即为目标记录。

✅ **SQL 关键点**

* 使用窗口函数 `LAG` 对当前行取前两行值。
* 三个连续的相同行表示为 `num = prev1 = prev2`。
* 使用 `DISTINCT` 去重，只保留出现过的数字一次。

---

**✅ 解法一：窗口函数 + LAG**

```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM (
  SELECT 
    num,
    LAG(num, 1) OVER (ORDER BY id) AS prev1,
    LAG(num, 2) OVER (ORDER BY id) AS prev2
  FROM Logs
) t
WHERE num = prev1 AND num = prev2;
```

🧠 **收获总结**

* `LAG` 是处理相邻行比较的利器。
* 通过三行窗口对比实现对“连续性”的判断。
* 相比子查询方式，这种写法更高效、可读性更好。

---
明白了！我帮你补充完善，给出两种解法的 SQL 关键点，整合成统一规范的格式：

---

✅ **题目：181. Employees Earning More Than Their Managers**
📝 **题目描述**
`Employee` 表记录员工及其经理信息，找出工资比经理高的员工姓名。返回列名为 `Employee`。

💡 **解题思路**
利用表的自连接，将员工和其经理关联，通过比较工资筛选符合条件的员工。

---

✅ **SQL 关键点**

* **方法一（显式 JOIN）**

  * 使用显式 `JOIN` 将员工表与自己连接，条件为 `e.ManagerId = m.Id`。
  * 利用 `WHERE e.Salary > m.Salary` 过滤工资更高的员工。
  * 返回员工姓名。

* **方法二（隐式连接）**

  * 使用逗号分隔的隐式连接写法，`FROM Employee e1, Employee e2`。
  * 连接条件写在 `WHERE` 子句中：`e1.ManagerId = e2.Id`。
  * 通过 `WHERE` 过滤 `e1.Salary > e2.Salary` 的员工。
  * 返回员工姓名。

---

### 方法一：显式 JOIN 写法

```sql
SELECT e.name AS Employee
FROM Employee e
JOIN Employee m ON e.ManagerId = m.Id
WHERE e.Salary > m.Salary;
```

---

### 方法二：隐式连接写法

```sql
SELECT e1.Name AS Employee
FROM Employee e1, Employee e2
WHERE e1.ManagerId = e2.Id
  AND e1.Salary > e2.Salary;
```

---

🧠 **收获总结**

* 理解并掌握自连接是解决此类上下级关联问题的核心。
* 显式 JOIN 语法更规范、易读，建议优先使用。
* 隐式连接是 SQL 早期语法，功能相同但易产生歧义。

---
明白了，我按照你之前笔记的风格和格式，帮你整理这道“查找重复电子邮件”的题目：

---

## ✅ 题目：182.查找重复电子邮件（Find Duplicate Emails）

### 📝 题目描述

给定 Person 表，包含唯一主键 `id` 和不含大写字母的 `email` 字段。

编写 SQL 查询找出所有重复出现的电子邮件（即出现次数超过1次的 email）。

结果中只返回重复的 email，顺序不限。

---

### 💡 解题思路

* 利用 `GROUP BY email` 聚合，将相同 email 归为一组。
* 使用 `HAVING COUNT(*) > 1` 筛选出现超过一次的邮件。
* 返回这些重复的 email 即可。

---

### ✅ SQL 解法一（GROUP BY + HAVING）

```sql
SELECT email AS Email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

* `GROUP BY email` 对邮件分组。
* `HAVING COUNT(*) > 1` 过滤出重复的邮件。
* 返回列名为 `Email`。

---

### ✅ SQL 解法二（使用 EXISTS）

```sql
SELECT DISTINCT p1.email AS Email
FROM Person p1
WHERE EXISTS (
    SELECT 1
    FROM Person p2
    WHERE p2.email = p1.email
      AND p2.id <> p1.id
);
```

* 通过 `EXISTS` 判断是否有相同 email 但不同 id 的记录。
* 使用 `DISTINCT` 去重，确保每个重复 email 只出现一次。
*SELECT 1：这里的 1 是一个常量，表示只要查询能找到符合条件的记录，就返回这个常量。
---

### 🧠 收获总结

* `GROUP BY` 与 `HAVING` 是统计重复值的常用利器。
* `EXISTS` 子查询也可实现重复查找，灵活性高。
* 熟练掌握聚合和子查询，提升查询能力。

---




✅ **题目：183. Customers Who Never Order**
📝 **题目描述**
Customer 表和 Orders 表，找出所有从未下过订单的客户。
Customers 表：

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
在 SQL 中，id 是该表的主键。
该表的每一行都表示客户的 ID 和名称。
Orders 表：

+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| customerId  | int  |
+-------------+------+
在 SQL 中，id 是该表的主键。
customerId 是 Customers 表中 ID 的外键( Pandas 中的连接键)。
该表的每一行都表示订单的 ID 和订购该订单的客户的 ID。
 
结果格式如下所示。
示例 1：
输入：
Customers table:
+----+-------+
| id | name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders table:
+----+------------+
| id | customerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
输出：
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+

💡 **解题思路**

* 方法一：使用 LEFT JOIN 找出订单为 null 的客户。
* 方法二：使用 NOT EXISTS 子查询排除有订单的客户。

**方法一：LEFT JOIN**

```sql
SELECT a.name as Customers
FROM Customers a
LEFT JOIN Orders ON a.id = Orders.customerId
WHERE Orders.customerId IS NULL;
```

### 【替代方法：使用 NOT IN 子查询】
#### **核心逻辑**：
✓ **集合排除**：从未下订单的客户集合 = 全部客户 - 已下订单的客户  
✓ **子查询获取**：先获取所有下过订单的客户ID  

#### **SQL 解决方案**：
```sql
SELECT name AS Customers
FROM Customers
WHERE id NOT IN (
    SELECT DISTINCT customerId 
    FROM Orders 
    WHERE customerId IS NOT NULL
);
```

#### **执行解析**：
→ **步骤分解**：  
 1. 内层查询：  
  `SELECT DISTINCT customerId FROM Orders`  
  → 获取所有下过订单的客户ID（去重）  
  ✓ `WHERE customerId IS NOT NULL` 确保排除空值干扰  
 2. 外层查询：  
  `WHERE id NOT IN (...)`  
  → 筛选不在"已下单客户"列表中的客户  
  
🧠 **收获总结**
*必须添加 `WHERE customerId IS NOT NULL`
* LEFT JOIN 结合 NULL 判断实现反向筛选。
* NOT EXISTS 是典型的反向查询写法。
* 理解两种方式效率差异及应用场景。

---

✅ **题目：184. Department Highest Salary**
📝 **题目描述**
Employee 表包含 id、name、salary 和 departmentId。
查询每个部门的最高工资员工姓名和工资。
表： Employee

+--------------+---------+
| 列名          | 类型    |
+--------------+---------+
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |
+--------------+---------+
在 SQL 中，id是此表的主键。
departmentId 是 Department 表中 id 的外键（在 Pandas 中称为 join key）。
此表的每一行都表示员工的 id、姓名和工资。它还包含他们所在部门的 id。
 

表： Department

+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
在 SQL 中，id 是此表的主键列。
此表的每一行都表示一个部门的 id 及其名称。
 

查找出每个部门中薪资最高的员工。
按 任意顺序 返回结果表。
查询结果格式如下例所示。

 

示例 1:

输入：
Employee 表:
+----+-------+--------+--------------+
| id | name  | salary | departmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Jim   | 90000  | 1            |
| 3  | Henry | 80000  | 2            |
| 4  | Sam   | 60000  | 2            |
| 5  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
Department 表:
+----+-------+
| id | name  |
+----+-------+
| 1  | IT    |
| 2  | Sales |
+----+-------+
输出：
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
| IT         | Max      | 90000  |
+------------+----------+--------+
解释：Max 和 Jim 在 IT 部门的工资都是最高的，Henry 在销售部的工资最高。

按照你提供的笔记风格，我已将 LeetCode 第 184 题整理如下：

---


### 💡 解题思路

* **步骤 1**：将 `Employee` 表与 `Department` 表通过 `departmentId = id` 进行连接。
* **步骤 2**：通过子查询找出每个部门的最高工资。
* **步骤 3**：将最高工资与员工表匹配，筛出每个部门中所有达到该工资的员工。

---

### ✅ SQL 解法一（子查询 + 连接）

```sql
SELECT d.name AS Department,
       e.name AS Employee,
       e.salary AS Salary
FROM Employee e
JOIN Department d
  ON e.departmentId = d.id
WHERE (e.departmentId, e.salary) IN (
    SELECT departmentId, MAX(salary)
    FROM Employee
    GROUP BY departmentId
);
```

* 先 `JOIN` 两张表获取员工和部门对应关系。
* 使用 `(departmentId, salary) IN (...)` 配对子查询结果，找出部门内最高薪资员工。

---

### ✅ SQL 解法二（窗口函数）

```sql
SELECT Department, Employee, Salary
FROM (
    SELECT d.name AS Department,
           e.name AS Employee,
           e.salary AS Salary,
           DENSE_RANK() OVER (PARTITION BY e.departmentId ORDER BY e.salary DESC) AS rk
    FROM Employee e
    JOIN Department d
      ON e.departmentId = d.id
) t
WHERE rk = 1;
```

* `DENSE_RANK()` 按部门对工资排名，排名为 1 即为最高工资。
* 在外层查询中取出 `rk = 1` 的记录，即为每个部门的最高薪资员工。

---

### 🧠 收获总结

* 学会通过 `(列1, 列2) IN (子查询)` 组合键查找复合条件匹配。
* 掌握 `DENSE_RANK()` 窗口函数的用法，可方便处理排名相关的问题。
* 对分组聚合与多表连接配合使用有了更深入理解。

---

如果你需要继续整理下一个题目，请随时告诉我题号！我会保持同样的风格继续。你也可以指定优先用哪种解法。


---

✅ **题目：185. Department Top Three Salaries**
📝 **题目描述**
Employee(Id, Name, Salary, DepartmentId) 表和 Department(Id, Name) 表。查询每个部门前三高的所有员工姓名、薪水和部门名（同薪并列，且并列不跳号）。
表: Employee

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |
+--------------+---------+
id 是该表的主键列(具有唯一值的列)。
departmentId 是 Department 表中 ID 的外键（reference 列）。
该表的每一行都表示员工的ID、姓名和工资。它还包含了他们部门的ID。
 

表: Department

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
id 是该表的主键列(具有唯一值的列)。
该表的每一行表示部门ID和部门名。
 

公司的主管们感兴趣的是公司每个部门中谁赚的钱最多。一个部门的 高收入者 是指一个员工的工资在该部门的 不同 工资中 排名前三 。

编写解决方案，找出每个部门中 收入高的员工 。

以 任意顺序 返回结果表。

返回结果格式如下所示。

 

示例 1:

输入: 
Employee 表:
+----+-------+--------+--------------+
| id | name  | salary | departmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |
+----+-------+--------+--------------+
Department  表:
+----+-------+
| id | name  |
+----+-------+
| 1  | IT    |
| 2  | Sales |
+----+-------+
输出: 
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
解释:
在IT部门:
- Max的工资最高
- 兰迪和乔都赚取第二高的独特的薪水
- 威尔的薪水是第三高的

在销售部:
- 亨利的工资最高
- 山姆的薪水第二高
- 没有第三高的工资，因为只有两名员工
 

提示：

没有姓名、薪资和部门 完全 相同的员工。
💡 **解题思路**
1. 窗口函数：对每个部门按薪水降序，使用 `DENSE_RANK()` 给行打标签，取 `<=3`。
2. 子查询计数：对每行在本部门中，比它薪水高的不同值个数少于 3，即排在前三。

**方法一：窗口函数**

```sql
SELECT d.Name    AS Department,
       e.Name    AS Employee,
       e.Salary
FROM (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY DepartmentId ORDER BY Salary DESC) AS rk
    FROM Employee
) e
JOIN Department d ON e.DepartmentId = d.Id
WHERE e.rk <= 3
ORDER BY d.Name, e.Salary DESC;
```

**方法二：子查询计数**

```sql
SELECT d.Name    AS Department,
       e.Name    AS Employee,
       e.Salary
FROM Employee e
JOIN Department d ON e.DepartmentId = d.Id
WHERE (
    SELECT COUNT(DISTINCT Salary)
    FROM Employee
    WHERE DepartmentId = e.DepartmentId
      AND Salary > e.Salary
) < 3
ORDER BY d.Name, e.Salary DESC;
```

🧠 **收获总结**

* 窗口函数写法最简洁，可灵活调整排名阈值；
* 子查询计数法虽稍繁，但不依赖高级特性，适用于任何支持子查询的数据库；
* 掌握并列排名和前 N 名的两种通用思路。

---

✅ **题目：196. Delete Duplicate Emails**
📝 **题目描述**
Person(Id, Email) 表。删除所有重复的邮箱记录，只保留每个邮箱最小 Id 的那条，删除后输出完整表。
示例 1:

输入: 
Person 表:
+----+------------------+
| id | email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
输出: 
+----+------------------+
| id | email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
解释: john@example.com重复两次。我们保留最小的Id = 1。

💡 **解题思路**
1. 自连接 DELETE：将表与自身配对，删除较大 Id 的重复行；
2. 子查询 DELETE：保留每组最小 Id，删除其余。

**方法一：自连接 DELETE**

```sql
DELETE p1
FROM Person p1
JOIN Person p2 
  ON p1.Email = p2.Email 
 AND p1.Id > p2.Id;
```

**方法二：子查询 DELETE**

```sql
DELETE FROM Person
WHERE Id NOT IN (
    SELECT MIN(Id)
    FROM Person
    GROUP BY Email
);
```

🧠 **收获总结**

* 自连接删除法直观高效；
* 子查询法通用，但部分数据库（如 MySQL）在同一语句中删除和查询可能需两步处理；
* 理解 DELETE 操作与 SELECT 聚合的结合使用。

---

✅ **题目：197. Rising Temperature**
📝 **题目描述**
Weather(Id, RecordDate, Temperature) 表。找出温度比前一天更高的所有日期 Id。
示例 1：

输入：
Weather 表：
+----+------------+-------------+
| id | recordDate | Temperature |
+----+------------+-------------+
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |
+----+------------+-------------+
输出：
+----+
| id |
+----+
| 2  |
| 4  |
+----+
解释：
2015-01-02 的温度比前一天高（10 -> 25）
2015-01-04 的温度比前一天高（20 -> 30）
💡 **解题思路**
1. 自连接：将表自身连接，关联日期相差 1 天的两行，比较温度；
2. 窗口函数：用 `LAG(Temperature)` 获取前一天温度，再做过滤。

**方法一：自连接**

```sql
SELECT w1.Id
FROM Weather w1
JOIN Weather w2 
  ON DATEDIFF(w1.RecordDate, w2.RecordDate) = 1
WHERE w1.Temperature > w2.Temperature;
```

**方法二：窗口函数**

```sql
SELECT Id
FROM (
    SELECT Id, Temperature,
           LAG(Temperature) OVER (ORDER BY RecordDate) AS prev_temp
    FROM Weather
) t
WHERE t.Temperature > t.prev_temp;
```

🧠 **收获总结**

* 自连接法直观，但依赖日期函数，适合旧版数据库；
* 窗口函数更简洁，对时间序列处理尤其高效；
* 学会两种方法，有助于多版本数据库兼容。

---

✅ **题目：511. Game Play Analysis I**
📝 **题目描述**
GamePlay(GameId, UserId, PlayTime) 表。按用户统计其第一次玩的游戏时长（最早 PlayTime），返回 UserId 和 FirstPlayTime。

💡 **解题思路**
1. 聚合子查询：对每个用户 `MIN(PlayTime)`；
2. 窗口函数：用 `ROW_NUMBER()` 按用户分区、时间升序取第一行。

**方法一：聚合子查询**

```sql
SELECT UserId,
       MIN(PlayTime) AS FirstPlayTime
FROM GamePlay
GROUP BY UserId;
```

**方法二：窗口函数**

```sql
SELECT UserId, PlayTime AS FirstPlayTime
FROM (
    SELECT UserId, PlayTime,
           ROW_NUMBER() OVER (PARTITION BY UserId ORDER BY PlayTime) AS rn
    FROM GamePlay
) t
WHERE t.rn = 1;
```

🧠 **收获总结**

* 聚合子查询法最直观、性能佳；
* 窗口函数法灵活，可扩展到取第 N 次等更多场景；
* 理解分区和排序在窗口函数中的应用。

---
## ✅ 题目：262. Trips and Users

### 📝 题目描述

取消率 的计算方式如下：(被司机或乘客取消的非禁止用户生成的订单数量) / (非禁止用户生成的订单总数)。

编写解决方案找出 "2013-10-01" 至 "2013-10-03" 期间有 至少 一次行程的非禁止用户（乘客和司机都必须未被禁止）的 取消率。非禁止用户即 banned 为 No 的用户，禁止用户即 banned 为 Yes 的用户。其中取消率 Cancellation Rate 需要四舍五入保留 两位小数 。

返回结果表中的数据 无顺序要求 。

结果格式如下例所示。

 

示例 1：

输入： 
Trips 表：
+----+-----------+-----------+---------+---------------------+------------+
| id | client_id | driver_id | city_id | status              | request_at |
+----+-----------+-----------+---------+---------------------+------------+
| 1  | 1         | 10        | 1       | completed           | 2013-10-01 |
| 2  | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 |
| 3  | 3         | 12        | 6       | completed           | 2013-10-01 |
| 4  | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 |
| 5  | 1         | 10        | 1       | completed           | 2013-10-02 |
| 6  | 2         | 11        | 6       | completed           | 2013-10-02 |
| 7  | 3         | 12        | 6       | completed           | 2013-10-02 |
| 8  | 2         | 12        | 12      | completed           | 2013-10-03 |
| 9  | 3         | 10        | 12      | completed           | 2013-10-03 |
| 10 | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 |
+----+-----------+-----------+---------+---------------------+------------+
Users 表：
+----------+--------+--------+
| users_id | banned | role   |
+----------+--------+--------+
| 1        | No     | client |
| 2        | Yes    | client |
| 3        | No     | client |
| 4        | No     | client |
| 10       | No     | driver |
| 11       | No     | driver |
| 12       | No     | driver |
| 13       | No     | driver |
+----------+--------+--------+
输出：
+------------+-------------------+
| Day        | Cancellation Rate |
+------------+-------------------+
| 2013-10-01 | 0.33              |
| 2013-10-02 | 0.00              |
| 2013-10-03 | 0.50              |
+------------+-------------------+
解释：
2013-10-01：
  - 共有 4 条请求，其中 2 条取消。
  - 然而，id=2 的请求是由禁止用户（user_id=2）发出的，所以计算时应当忽略它。
  - 因此，总共有 3 条非禁止请求参与计算，其中 1 条取消。
  - 取消率为 (1 / 3) = 0.33
2013-10-02：
  - 共有 3 条请求，其中 0 条取消。
  - 然而，id=6 的请求是由禁止用户发出的，所以计算时应当忽略它。
  - 因此，总共有 2 条非禁止请求参与计算，其中 0 条取消。
  - 取消率为 (0 / 2) = 0.00
2013-10-03：
  - 共有 3 条请求，其中 1 条取消。
  - 然而，id=8 的请求是由禁止用户发出的，所以计算时应当忽略它。
  - 因此，总共有 2 条非禁止请求参与计算，其中 1 条取消。
  - 取消率为 (1 / 2) = 0.50。

根据你统一的笔记风格，下面是对 LeetCode SQL 第 262 题（Trips + Users 表，计算取消率）的整理：

---

### 💡 解题思路

* 先筛选出 `banned = 'No'` 的乘客与司机。
* 将 `Trips` 表与 `Users` 表连接两次（分别为 client 和 driver），筛选出双方都为非禁止用户的行程。
* 仅统计 `"2013-10-01"` 到 `"2013-10-03"` 的行程。
* 使用 `CASE WHEN` 统计每天的取消总数与总行程数，计算 `取消率 = 取消数 / 总数`。
* 使用 `ROUND(..., 2)` 保留两位小数。

---

### ✅ SQL 解法一（连接 + 分组 + 聚合）

```sql
SELECT request_at AS Day,
       ROUND(
           SUM(CASE WHEN status != 'completed' THEN 1 ELSE 0 END) 
           / COUNT(*), 2
       ) AS `Cancellation Rate`
FROM Trips t
JOIN Users c ON t.client_id = c.users_id AND c.banned = 'No'
JOIN Users d ON t.driver_id = d.users_id AND d.banned = 'No'
WHERE request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY request_at;
```

* `JOIN Users c` 过滤非禁止乘客，`JOIN Users d` 过滤非禁止司机。
* `status != 'completed'` 代表行程被取消。
* `ROUND(..., 2)` 保留两位小数。
* `GROUP BY request_at` 实现按天统计。

---

### ✅ SQL 解法二（使用 CTE 分层过滤）

```sql
WITH ValidTrips AS (
    SELECT 
        t.id,
        t.client_id,
        t.driver_id,
        t.city_id,
        t.status,
        t.request_at
    FROM Trips t
    JOIN Users c ON t.client_id = c.users_id AND c.banned = 'No'
    JOIN Users d ON t.driver_id = d.users_id AND d.banned = 'No'
    WHERE t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
)
SELECT request_at AS Day,
       ROUND(
           SUM(CASE 
                   WHEN status IN ('cancelled_by_driver', 'cancelled_by_client') THEN 1 
                   ELSE 0 
               END) / COUNT(*), 
           2
       ) AS `Cancellation Rate`
FROM ValidTrips
GROUP BY request_at;

```
*SELECT * 在多次连接同一表时容易出现 列名冲突，应避免。
* 通过 CTE（公共表表达式）将筛选条件提取出来，使主查询更清晰。
* 本质逻辑与解法一一致，仅结构更清晰、易于维护。

---

### 🧠 收获总结

* 熟练掌握双表连接同一表两次的写法，用于 client 和 driver 的筛选。
* 学会用 `CASE WHEN` 与 `SUM`/`COUNT` 结合实现自定义统计逻辑。
* 理解 `ROUND` 和 `GROUP BY` 在每日汇总分析中的用法。
* 掌握用 CTE 提高 SQL 可读性和复用性的技巧。

---

下面是对 LeetCode 第 511 题的整理，完全按照你之前笔记的统一风格输出：

---

## ✅ 题目：511. Game Play Analysis I

### 📝 题目描述

给定活动表 `Activity`，记录玩家在游戏平台上的每日行为：

* `player_id`：玩家 ID
* `device_id`：登录设备
* `event_date`：行为发生日期
* `games_played`：当日打开的游戏数（可能为 0）
* 主键是 `(player_id, event_date)`。
查询每位玩家 第一次登录平台的日期。

查询结果的格式如下所示：

Activity 表：
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result 表：
+-----------+-------------+
| player_id | first_login |
+-----------+-------------+
| 1         | 2016-03-01  |
| 2         | 2017-06-25  |
| 3         | 2016-03-02  |
+-----------+-------------+
请查询每位玩家 **第一次登录平台的日期**。

---

### 💡 解题思路

* 每个玩家可能有多次活动记录。
* 使用 `GROUP BY` 按玩家分组，然后对每组取 `MIN(event_date)`，即为首次登录日期。
* 仅返回 `player_id` 和首次登录的日期列，命名为 `first_login`。

---

### ✅ SQL 解法一（GROUP BY + MIN）

```sql
SELECT player_id,
       MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

* `GROUP BY player_id` 将记录按玩家分组。
* `MIN(event_date)` 提取该玩家的最早登录日期。
* 返回列重命名为 `first_login`，符合题目要求。

---

### ✅ SQL 解法二（窗口函数）

```sql
SELECT DISTINCT player_id,
       FIRST_VALUE(event_date) OVER (
           PARTITION BY player_id ORDER BY event_date
       ) AS first_login
FROM Activity;
```

* `FIRST_VALUE(event_date)` 获取每个玩家按时间排序后的第一条日期。
* `DISTINCT player_id, first_login` 去除重复结果。
* 这种方法适用于后续如果还需要额外列（如设备）时的扩展场景。

---

### 🧠 收获总结

* 掌握了 `GROUP BY + MIN()` 实现分组最小值查询。
* 理解 `FIRST_VALUE()` 窗口函数在保留原始行信息的优势。
* 熟悉数据去重与聚合的组合方式，便于处理行为日志类问题。

---



---

## ✅ 题目：550. Game Play Analysis IV

### 📝 题目描述

给定玩家行为记录表 `Activity`：

* 每条记录表示某玩家在某天使用某设备登录并玩了若干游戏（可能为 0）。
* 主键为 `(player_id, event_date)`。

请计算**在首次登录的第二天再次登录的玩家比例**，保留小数点后 **两位**，字段命名为 `fraction`。


结果格式如下所示：

示例 1：

输入：
Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+
输出：
+-----------+
| fraction  |
+-----------+
| 0.33      |
+-----------+
解释：
只有 ID 为 1 的玩家在第一天登录后才重新登录，所以答案是 1/3 = 0.33
---



### 💡 解题思路

* 找出每个玩家的首次登录日期。
* 筛选所有在“首次登录的第二天”登录的记录。
* 使用 `COUNT(DISTINCT)` 计算满足条件的玩家数。
* 再除以总玩家数，得到比例。

---

### ✅ SQL 解法

```sql
SELECT 
  ROUND(
    COUNT(DISTINCT a.player_id) / 
    (SELECT COUNT(DISTINCT player_id) FROM Activity), 
    2
  ) AS fraction
FROM Activity a
JOIN (
  SELECT player_id, MIN(event_date) AS first_login
  FROM Activity
  GROUP BY player_id
) b
ON a.player_id = b.player_id
WHERE DATEDIFF(a.event_date, b.first_login) = 1;
```

* 子查询 `b` 得出每位玩家的首次登录时间。
* 主查询中 `DATEDIFF(a.event_date, b.first_login) = 1` 精确判断“是否为次日回访”。
* 分母 `SELECT COUNT(DISTINCT player_id)` 计算所有玩家总数，避免被 `JOIN` 过滤。
* 使用 `ROUND(..., 2)` 保留两位小数。

---

### 🧠 收获总结

* 使用 `DATEDIFF()` 判断具体的日期差是处理“次日登录”等问题的核心。
* 分子、分母统计范围不同，**分母不能受 JOIN 影响**，需单独统计。
* 掌握 `JOIN + 子查询` + `COUNT(DISTINCT)` 的组合使用，适合行为分析类题目。
* 注意精度处理，`ROUND(..., 2)` 是面试题中常见细节点。

---
以下是 LeetCode 第 570 题的整理内容，风格统一为你之前的笔记格式：

---

## ✅ 题目：570. Managers with at Least 5 Direct Reports

### 📝 题目描述

给定员工表 `Employee`：

* `id`：员工唯一编号
* `name`：员工姓名
* `department`：所属部门
* `managerId`：该员工直属经理的 `id`（可为 NULL）

找出所有**直接下属人数 ≥ 5** 的经理的姓名 `name`。
示例 1:

输入: 
Employee 表:
+-----+-------+------------+-----------+
| id  | name  | department | managerId |
+-----+-------+------------+-----------+
| 101 | John  | A          | Null      |
| 102 | Dan   | A          | 101       |
| 103 | James | A          | 101       |
| 104 | Amy   | A          | 101       |
| 105 | Anne  | A          | 101       |
| 106 | Ron   | B          | 101       |
+-----+-------+------------+-----------+
输出: 
+------+
| name |
+------+
| John |
+------+
---

### 💡 解题思路

* 员工的 `managerId` 表示他归属于哪位经理。
* 把 `managerId` 看作外键，按其分组统计出现次数即为“每位经理的直接下属数量”。
* 再将这些 `managerId` 与 `Employee` 表自身连接，取出对应的姓名。

---

### ✅ SQL 解法（GROUP BY + JOIN）

```sql
SELECT e.name
FROM Employee e
JOIN (
    SELECT managerId
    FROM Employee
    WHERE managerId IS NOT NULL
    GROUP BY managerId
    HAVING COUNT(*) >= 5
) m ON e.id = m.managerId;
```

* 内层子查询：

  * 对 `managerId` 分组，统计每位经理拥有的下属数量。
  * 使用 `HAVING COUNT(*) >= 5` 筛出符合条件的经理。
* 外层将这些 `managerId` 与主表连接，取得经理的 `name` 字段。

---

### 🧠 收获总结

* 熟练掌握“**自身连接**”技巧：一个表对自己的字段进行连接查询。
* `GROUP BY managerId` 是统计下属数量的经典用法。
* 使用 `HAVING` 代替 `WHERE` 对聚合结果做条件筛选。
* `IS NOT NULL` 可避免统计虚假的上级（如 NULL 表示无经理）。

---
以下是 LeetCode 第 577 题的整理，保持你笔记的统一风格：

---

## ✅ 题目：577. Employee Bonus

### 📝 题目描述

有两张表：

* `Employee(empId, name, supervisor, salary)`
* `Bonus(empId, bonus)`

`Bonus` 表中存有部分员工的奖金信息，`empId` 是唯一键，且是 `Employee` 的外键。

请查询每位员工的姓名和奖金，要求：

* 对于奖金少于 1000 的员工，显示其姓名和奖金数额；
* 对于没有奖金记录的员工，奖金显示为 `null`。
示例 1：

输入：
Employee table:
+-------+--------+------------+--------+
| empId | name   | supervisor | salary |
+-------+--------+------------+--------+
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |
+-------+--------+------------+--------+
Bonus table:
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
输出：
+------+-------+
| name | bonus |
+------+-------+
| Brad | null  |
| John | null  |
| Dan  | 500   |
+------+-------+
---

### 💡 解题思路

* 使用 `LEFT JOIN` 将 `Employee` 和 `Bonus` 关联，保证所有员工都显示。
* `WHERE` 或 `ON` 子句中筛选奖金少于 1000 或奖金为 `null`。
* 注意奖金可能为 `null`，代表无奖金记录。

---

### ✅ SQL 解法（LEFT JOIN + 条件筛选）

```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

* `LEFT JOIN` 保证员工全部列出，即使没有奖金信息。
* `WHERE` 过滤奖金少于 1000 或无奖金记录。
* 输出列为 `name` 和 `bonus`，符合题目格式。

---

### 🧠 收获总结

* 掌握 `LEFT JOIN` 的使用，确保主表（员工）数据全部保留。
* 处理 `NULL` 时，`IS NULL` 是判断是否无对应记录的关键。
* 明确过滤条件需包含 `OR b.bonus IS NULL`，否则无奖金员工会被排除。

---
以下是 LeetCode 第 584 题的整理，保持统一笔记风格：

---

## ✅ 题目：584. Find Customers Who Never Order

### 📝 题目描述

给定客户表 `Customer`：

* `id`：客户唯一 ID
* `name`：客户姓名
* `referee_id`：推荐该客户的客户 ID（可能为 NULL）

请找出 **没有被 id = 2 的客户推荐的客户的姓名**。
示例 1：

输入： 
Customer 表:
+----+------+------------+
| id | name | referee_id |
+----+------+------------+
| 1  | Will | null       |
| 2  | Jane | null       |
| 3  | Alex | 2          |
| 4  | Bill | null       |
| 5  | Zack | 1          |
| 6  | Mark | 2          |
+----+------+------------+
输出：
+------+
| name |
+------+
| Will |
| Jane |
| Bill |
| Zack |
+------+
---

### 💡 解题思路

* 查找所有客户中 `referee_id` 不等于 2 或者为 `NULL` 的客户。
* 即排除被客户 2 推荐的客户。
* 返回符合条件的客户姓名。

---

### ✅ SQL 解法（条件过滤）

```sql
SELECT name
FROM Customer
WHERE referee_id <> 2 OR referee_id IS NULL;
```

* `referee_id <> 2` 排除被客户 2 推荐的客户。
* 需要额外判断 `referee_id IS NULL` 以包含无推荐的客户。
* 结果无顺序要求。

---

### 🧠 收获总结

* 过滤条件中对 `NULL` 的判断非常重要，`NULL` 不等于任何值。
* `OR` 条件用以确保两个条件都能被满足。
* 理解“被某人推荐”是通过外键字段 `referee_id` 关联实现的。

---
以下是 LeetCode 第 585 题的整理，按照你之前的笔记风格输出：

---

## ✅ 题目：585. Investment in 2016

### 📝 题目描述

给定保险表 `Insurance`，包含字段：

* `pid`：投保编号（主键）
* `tiv_2015`：2015 年投保总金额
* `tiv_2016`：2016 年投保总金额
* `lat`, `lon`：投保人所在城市的经纬度，均不为空

请统计满足以下条件的投保人在 2016 年的投保金额总和（保留两位小数）：

1. 该投保人在 2015 年的投保金额 `tiv_2015` 至少与另一名投保人相同。
2. 该投保人所在城市的 `(lat, lon)` 与其他任何投保人均不同（即该城市唯一）。
示例 1：

输入：
Insurance 表：
+-----+----------+----------+-----+-----+
| pid | tiv_2015 | tiv_2016 | lat | lon |
+-----+----------+----------+-----+-----+
| 1   | 10       | 5        | 10  | 10  |
| 2   | 20       | 20       | 20  | 20  |
| 3   | 10       | 30       | 20  | 20  |
| 4   | 10       | 40       | 40  | 40  |
+-----+----------+----------+-----+-----+
输出：
+----------+
| tiv_2016 |
+----------+
| 45.00    |
+----------+
解释：
表中的第一条记录和最后一条记录都满足两个条件。
tiv_2015 值为 10 与第三条和第四条记录相同，且其位置是唯一的。

第二条记录不符合任何一个条件。其 tiv_2015 与其他投保人不同，并且位置与第三条记录相同，这也导致了第三条记录不符合题目要求。
因此，结果是第一条记录和最后一条记录的 tiv_2016 之和，即 45 。
---

### 💡 解题思路

* 使用自连接判断每个投保人的 `tiv_2015` 是否有至少另一个相同金额的投保人。
* 使用分组统计判断 `(lat, lon)` 是否唯一。
* 选出满足以上两条件的投保人，求其 `tiv_2016` 之和。
* 最后用 `ROUND(..., 2)` 保留两位小数。

---

### ✅ SQL 解法（自连接 + 分组过滤）

```sql
WITH UniqueCity AS (
    SELECT lat, lon
    FROM Insurance
    GROUP BY lat, lon
    HAVING COUNT(*) = 1
),
SameTiv2015 AS (
    SELECT DISTINCT a.pid
    FROM Insurance a
    JOIN Insurance b
      ON a.tiv_2015 = b.tiv_2015 AND a.pid <> b.pid
)
SELECT ROUND(SUM(i.tiv_2016), 2) AS tiv_2016
FROM Insurance i
JOIN UniqueCity uc ON i.lat = uc.lat AND i.lon = uc.lon
JOIN SameTiv2015 st ON i.pid = st.pid;
```

* `UniqueCity`：筛选唯一城市 `(lat, lon)`。
* `SameTiv2015`：筛选 `tiv_2015` 与其他投保人相同的投保人。
* 连接以上结果取交集，计算符合条件的 `tiv_2016` 总和。
* 使用 `ROUND` 保留两位小数。

---

### 🧠 收获总结

* 利用 `WITH` 语句（CTE）提高查询可读性和分步筛选。
* 通过自连接实现条件“存在另一条相同 `tiv_2015`”。
* `GROUP BY ... HAVING COUNT(*) = 1` 用于筛选唯一城市。
* 注意多条件交集的实现方式。

---
以下是 LeetCode 第 586 题的整理，格式保持与你之前笔记风格一致：

---

## ✅ 题目：586. Customer With the Most Orders

### 📝 题目描述

给定订单表 `Orders`：

* `order_number`：订单唯一编号（主键）
* `customer_number`：客户编号

请找出**下订单最多的客户的 `customer_number`**。

题目保证只有一个客户的订单数最多。
示例 1:

输入: 
Orders 表:
+--------------+-----------------+
| order_number | customer_number |
+--------------+-----------------+
| 1            | 1               |
| 2            | 2               |
| 3            | 3               |
| 4            | 3               |
+--------------+-----------------+
输出: 
+-----------------+
| customer_number |
+-----------------+
| 3               |
+-----------------+
解释: 
customer_number 为 '3' 的顾客有两个订单，比顾客 '1' 或者 '2' 都要多，因为他们只有一个订单。
所以结果是该顾客的 customer_number ，也就是 3 。
---

### 💡 解题思路

* 对 `customer_number` 分组，统计每个客户的订单数量。
* 找出订单数量的最大值。
* 返回拥有最大订单数的客户编号。

---

### ✅ SQL 解法一（分组 + LIMIT）

```sql
SELECT customer_number
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;
```

* 按订单数降序排序，取第一条记录。

---

### ✅ SQL 解法二（子查询 + HAVING）

```sql
SELECT customer_number
FROM Orders
GROUP BY customer_number
HAVING COUNT(*) = (
    SELECT MAX(order_count)
    FROM (
        SELECT COUNT(*) AS order_count
        FROM Orders
        GROUP BY customer_number
    ) t
);
```

* 先用子查询计算每个客户的订单数，找出最大订单数。
* 外层筛选订单数等于最大值的客户。

---

### 🧠 进阶 — 多客户并列最多订单

上面第二种写法可以直接支持多客户并列最多订单的情况，返回所有并列最大订单数客户。

---

### 🧠 收获总结

* `GROUP BY` 聚合统计是解决此类频次问题的基础。
* `ORDER BY ... LIMIT 1` 是最简单的获取最大值对应行的方法。
* 使用子查询结合 `HAVING` 支持多值匹配的情况。
* 题目区分单一最大客户和并列最大客户，思考代码灵活性。

---


## ✅ 题目：595. Big Countries

### 📝 题目描述

**World(name, continent, area, population, gdp)** 表。查询“大国家”的名称、人口和面积，定义为：面积超过 300 万或人口超过 2500 万。


### 💡 解题思路

* 直接在 `WHERE` 中用 `OR` 连接面积和人口条件。
* 也可拆成两个查询用 `UNION` 合并。

### ✅ SQL 关键点

* `WHERE area > 3000000 OR population > 25000000`
* 简洁直观的条件过滤。

```sql
SELECT name, population, area
FROM World
WHERE area > 3000000 OR population > 25000000;
```

### 🧠 收获总结

* 逻辑“或”条件直接写 `OR`。
* `UNION` 可拆分复杂查询，但语句更长。
* 根据实际场景灵活选择。

---

## ✅ 题目：596. Classes More Than 5 Students

### 📝 题目描述

**Courses(student, class)** 表。查询至少有5名不同学生参加的课程名称。

### 💡 解题思路

* 按课程分组，统计不同学生数。
* 使用 `COUNT(DISTINCT student)` 实现去重计数。

### ✅ SQL 关键点

* `GROUP BY class`
* `HAVING COUNT(DISTINCT student) >= 5`

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5;
```

### 🧠 收获总结

* 掌握分组聚合去重计数。
* `HAVING` 用于筛选分组后的结果。
* 去重统计是常见需求。

---

## ✅ 题目：601. Stadium Attendance

### 📝 题目描述

**Stadium(id, visit\_date, people)** 表记录每日人流量。找出任意连续三行及以上 `id` 均满足 `people >= 100` 的所有记录，按 `visit_date` 升序输出。

### 💡 解题思路

* 使用窗口函数行号差法标记连续区间。
* 相同组号表示连续满足条件的行。
* 筛选组内行数≥3的区间，输出对应记录。

### ✅ SQL 关键点

* 利用 `ROW_NUMBER()` 和 `id` 差值分组。
* `HAVING COUNT(*) >= 3` 过滤连续段。
* 联接原表输出所有连续记录。

```sql
SELECT sta.id, sta.visit_date, sta.people
FROM (
    SELECT MIN(id) AS minid, MAX(id) AS maxid, rk
    FROM (
        SELECT *, id - ROW_NUMBER() OVER (ORDER BY id) AS rk
        FROM Stadium
        WHERE people >= 100
    ) t
    GROUP BY rk
    HAVING COUNT(*) >= 3
) grp
JOIN Stadium sta ON sta.id BETWEEN grp.minid AND grp.maxid
ORDER BY sta.visit_date;
```

### 🧠 收获总结

* 掌握连续条件判定的行号差法。
* 窗口函数处理复杂序列问题很有用。
* 自连接法简单但局限较多。

---

## ✅ 题目：620. Not Boring Movies

### 📝 题目描述

**Cinema(id, movie, description, rating)** 表。有电影描述为 **boring** 的影片不感兴趣，找出描述不为 “boring” 且 `id` 为奇数的影片，按 `rating` 降序排列。
示例 1：

输入：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
输出：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
解释：
我们有三部电影，它们的 id 是奇数:1、3 和 5。id = 3 的电影是 boring 的，所以我们不把它包括在答案中。
### 💡 解题思路

* 过滤 `description <> 'boring'` 和奇数 `id`。
* 按评分降序排序。

### ✅ SQL 关键点

* 简单 `WHERE` 条件组合。
* `ORDER BY rating DESC`

```sql
SELECT id, movie, description, rating
FROM Cinema
WHERE description <> 'boring' AND id % 2 = 1
ORDER BY rating DESC;
```

### 🧠 收获总结

* 熟悉文本匹配与数值筛选。
* 灵活运用算术运算符和逻辑判断。

---

## ✅ 题目：626. Swap Seats

### 📝 题目描述

**Seat(id, student)** 表记录教室座位（`id` 连续递增）。小美希望交换相邻两位同学的座位，若学生总数为奇数，最后一人座位不变。输出按座位 `id` 重排的学生列表。
示例 1:

输入: 
Seat 表:
+----+---------+
| id | student |
+----+---------+
| 1  | Abbot   |
| 2  | Doris   |
| 3  | Emerson |
| 4  | Green   |
| 5  | Jeames  |
+----+---------+
输出: 
+----+---------+
| id | student |
+----+---------+
| 1  | Doris   |
| 2  | Abbot   |
| 3  | Green   |
| 4  | Emerson |
| 5  | Jeames  |
+----+---------+
解释:
请注意，如果学生人数为奇数，则不需要更换最后一名学生的座位。


---

### 💡 解题思路

* 利用窗口函数 `LEAD` 和 `LAG` 获取相邻行学生姓名。
* 对奇数 `id` 行，取下一行学生姓名；对偶数 `id` 行，取上一行学生姓名。
* 对于奇数行最后一条无下一行的情况，用 `COALESCE` 保持原学生姓名不变。

---

### ✅ SQL 关键点

* 使用 `CASE` 判断 `id` 的奇偶。
* `LEAD(student) OVER (ORDER BY id)` 获取下一行学生。
* `LAG(student) OVER (ORDER BY id)` 获取上一行学生。
* 用 `COALESCE` 处理最后一条奇数行无“下一行”情况。

---

### ✅ 参考 SQL 代码

```sql
SELECT
    id,
    CASE
        WHEN id % 2 = 1 THEN COALESCE(LEAD(student) OVER (ORDER BY id), student)
        ELSE LAG(student) OVER (ORDER BY id)
    END AS student
FROM Seat;
```

---

### 🧠 收获总结

* 窗口函数 `LEAD` 和 `LAG` 非常适合处理行内相邻数据。
* `COALESCE` 保证最后一条奇数行学生姓名不丢失，满足题目最后一人不交换的需求。
* 结合 `CASE` 灵活实现奇偶行的不同操作。

---

如需继续整理其他题目，欢迎告诉我题号！


## ✅ 题目：627. Swap Gender

### 📝 题目描述

**Salary(id, name, sex, salary)** 表。将 `sex` 列中 'm' 和 'f' 互换，所有男生变女生，女生变男生。只允许使用一条 `UPDATE` 语句。
示例 1:

输入：
Salary 表：
+----+------+-----+--------+
| id | name | sex | salary |
+----+------+-----+--------+
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
+----+------+-----+--------+
输出：
+----+------+-----+--------+
| id | name | sex | salary |
+----+------+-----+--------+
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
+----+------+-----+--------+
解释：
(1, A) 和 (3, C) 从 'm' 变为 'f' 。
(2, B) 和 (4, D) 从 'f' 变为 'm' 。
### 💡 解题思路

* 使用 `CASE WHEN` 或 `IF` 实现条件赋值。
* 单语句更新全部数据。

### ✅ SQL 关键点

* `UPDATE` 结合 `CASE` 或 `IF` 函数。

```sql
UPDATE Salary
SET sex = CASE sex WHEN 'm' THEN 'f' ELSE 'm' END;
```

### 🧠 收获总结

* 单条 `UPDATE` 实现数据互换。
* 条件表达式灵活替代多语句操作。

---






# 📅 Day 01 刷题总结

## ✅ 题目：175. Combine Two Tables

### 📝 题目描述

将两个表（Person 和 Address）通过 PersonId 进行连接，输出所有人的姓、名、城市和州。若某人无地址信息，则城市和州返回 NULL。

示例 1:

```
输入: 
Person表:
+----------+----------+-----------+
| personId | lastName | firstName |
+----------+----------+-----------+
| 1        | Wang     | Allen     |
| 2        | Alice    | Bob       |
+----------+----------+-----------+
Address表:
+-----------+----------+---------------+------------+
| addressId | personId | city          | state      |
+-----------+----------+---------------+------------+
| 1         | 2        | New York City | New York   |
| 2         | 3        | Leetcode      | California |
+-----------+----------+---------------+------------+
输出: 
+-----------+----------+---------------+----------+
| firstName | lastName | city          | state    |
+-----------+----------+---------------+----------+
| Allen     | Wang     | Null          | Null     |
| Bob       | Alice    | New York City | New York |
+-----------+----------+---------------+----------+
```

解释:
地址表中没有 personId = 1 的地址，所以它们的城市和州返回 null。addressId = 1 包含了 personId = 2 的地址信息。

### 💡 解题思路

* 使用 `LEFT JOIN` 是关键，确保 **Person 表的每一行都被返回**。
* Address 表若没有对应的 personId，会自动补 NULL。
* 连接条件使用 `ON Person.PersonId = Address.PersonId`。

### ✅ SQL 解法一（左连接）

```sql
SELECT 
    Person.FirstName AS firstName,
    Person.LastName AS lastName,
    Address.City,
    Address.State
FROM Person
LEFT JOIN Address ON Person.PersonId = Address.PersonId;
```

* `LEFT JOIN` 保留左表（Person）所有数据，包括无地址的行。
* 输出列按题目要求的顺序：firstName、lastName、city、state。

### 🧠 收获总结

* `LEFT JOIN` 常用于“包含所有左表数据”的场景，即使右表中无对应数据，也会返回 NULL。
* 明确列顺序，以符合输出格式要求。

---

## ✅ 题目：176. Second Highest Salary

### 📝 题目描述

查询并返回 Employee 表中第二高的 **不同** 薪水。

* 若不存在第二高的薪水，应返回 `null`。

示例 1:

```
输入：
Employee 表：
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
输出：
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

示例 2:

```
输入：
Employee 表：
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
+----+--------+
输出：
+---------------------+
| SecondHighestSalary |
+---------------------+
| null                |
+---------------------+
```

### 💡 解题思路

* 使用 `DISTINCT` 排除重复薪水。
* **方案一（LIMIT + OFFSET）**：对去重后的薪水排序，使用 `LIMIT 1 OFFSET 1` 定位到第二高薪水。
* **方案二（子查询）**：先找到最大薪水，然后从剩余薪水中取最大值。

### ✅ SQL 解法一（LIMIT + OFFSET）

```sql
SELECT 
    (SELECT DISTINCT salary 
     FROM Employee 
     ORDER BY salary DESC 
     LIMIT 1 OFFSET 1) AS SecondHighestSalary;
```

* `DISTINCT salary`：去重薪水。
* `ORDER BY salary DESC`：从高到低排序。
* `LIMIT 1 OFFSET 1`：跳过最高薪，取下一条记录。

### ✅ SQL 解法二（子查询）

```sql
SELECT MAX(salary) AS SecondHighestSalary 
FROM Employee 
WHERE salary < (SELECT MAX(salary) FROM Employee);
```

* 子查询先求出表中最大薪水。
* 外层查询中，排除最大值后取剩余中的最大值。
* 若所有薪水相同或仅一行，返回结果为 NULL。

### 🧠 收获总结

* 熟悉 `LIMIT + OFFSET` 定位排序中任意行的技巧。
* 理解子查询配合聚合函数的用法。
* 本题综合考察了排序、去重和条件筛选能力。

---

## ✅ 题目：177. Get Nth Highest Salary

### 📝 题目描述

编写一个解决方案查询 Employee 表中第 **n** 高的 **不同** 薪水。

* 如果少于 **n** 个不同薪水，查询结果应为 `null`。
* 返回列名为 `getNthHighestSalary(n)`。

示例:

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
+------------------------+
```

### 💡 解题思路

* **方案一（LIMIT + OFFSET）**：对去重后的薪水降序排序，使用 `LIMIT 1 OFFSET n-1` 定位第 n 条记录。
* **方案二（窗口函数 / 子查询）**：使用窗口函数 `DENSE_RANK()` 计算薪水排名，或通过子查询多层嵌套获取第 n 大薪水。

### ✅ SQL 解法一（LIMIT + OFFSET）

```sql
-- 以 MySQL/SQLite 语法为例
SELECT (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET n-1
) AS getNthHighestSalary(n);
```

* `DISTINCT salary`：去重。
* `ORDER BY salary DESC`：降序排列。
* `LIMIT 1 OFFSET n-1`：跳过前 n-1 个，取第 n 个。

### ✅ SQL 解法二（窗口函数：DENSE\_RANK）

```sql
-- 适用于支持窗口函数的数据库（如 MySQL 8+, SQL Server, PostgreSQL）
SELECT salary AS getNthHighestSalary(n)
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rk
    FROM Employee
) t
WHERE rk = n;
```

* `DENSE_RANK()`：对不同薪水进行连续排名。
* 在外层查询中筛选排名等于 n 的行。
* 若不存在则结果集为空，客户端可解释为 `null`。

### 🧠 收获总结

* 掌握利用 `LIMIT+OFFSET` 定位任意排名的方法。
* 熟悉窗口函数 `DENSE_RANK()` 的用法。
* 对去重、排序和排名的综合理解更加深入。

---

## ✅ 题目：178. Rank Scores

### 📝 题目描述

对 Scores 表中的每个分数 `score`，计算其排名 `rank`。排名规则是分数越高排名越靠前，分数相同时排名相同。输出 score 和对应 rank，按 score 降序排序。

示例 1:

```
输入: 
Scores 表:
+----+-------+
| id | score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
输出: 
+-------+------+
| score | rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

### 💡 解题思路

使用窗口函数 `RANK() OVER (ORDER BY score DESC)` 给分数排序，自动处理相同分数并列排名。

### ✅ SQL 解法一（窗口函数：RANK）

```sql
SELECT score, RANK() OVER (ORDER BY score DESC) AS rank
FROM Scores;
```

* 窗口函数 `RANK()` 为带并列名次的排序。
* `ORDER BY score DESC` 控制从高分到低分的排名顺序。

### 🧠 收获总结

* 熟练使用窗口函数实现并列排名。
* 理解 `RANK`、`DENSE_RANK`、`ROW_NUMBER` 的区别。
* 能够解决带并列名次的排序问题。

---

## ✅ 题目：180. Consecutive Numbers

### 📝 题目描述

表 `Logs` 包含两个字段：`id`（主键，自增）和 `num`（数字）。找出所有在表中**至少连续出现三次**的相同数字 `num`。

示例 1:

```
输入：
Logs 表：
+----+-----+
| id | num |
+----+-----+
| 1  | 1   |
| 2  | 1   |
| 3  | 1   |
| 4  | 2   |
| 5  | 1   |
| 6  | 2   |
| 7  | 2   |
+----+-----+
输出：
Result 表：
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```

解释：1 是唯一连续出现至少三次的数字。

### 💡 解题思路

使用窗口函数 `LAG` 来获取当前行前两行的 `num` 值，判断三行是否相等。

### ✅ SQL 解法一（窗口函数 + LAG）

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

* 使用 `LAG(num, k) OVER (ORDER BY id)` 获取前 k 行的 `num`。
* 当当前行及其前两行的 `num` 相等时，即表示连续出现至少三次。
* `SELECT DISTINCT` 确保结果中每个数字只出现一次。

### 🧠 收获总结

* `LAG` 是处理相邻行比较的利器。
* 通过比较三行的值实现对“连续出现”的判断。
* 与子查询方法相比，窗口函数方式更简洁且高效。

---

## ✅ 题目：181. Employees Earning More Than Their Managers

### 📝 题目描述

`Employee` 表记录员工及其经理信息，找出工资比经理高的员工姓名。返回列名为 `Employee`。

示例 1:

```
输入: 
Employee 表:
+----+-------+--------+-----------+
| id | name  | salary | managerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | Null      |
| 4  | Max   | 90000  | Null      |
+----+-------+--------+-----------+
输出: 
+----------+
| Employee |
+----------+
| Joe      |
+----------+
```

解释: Joe 是唯一挣得比经理多的员工。

### 💡 解题思路

利用表的自连接，将员工和其经理关联，通过比较工资筛选符合条件的员工。

### ✅ SQL 解法一（显式 JOIN）

```sql
SELECT e.name AS Employee
FROM Employee e
JOIN Employee m ON e.ManagerId = m.Id
WHERE e.Salary > m.Salary;
```

* 将 `Employee` 表自连接，其中 `e` 代表员工，`m` 代表其经理。
* 连接条件 `e.ManagerId = m.Id`。
* `WHERE e.Salary > m.Salary` 筛选出工资更高的员工。

### ✅ SQL 解法二（隐式连接）

```sql
SELECT e1.name AS Employee
FROM Employee e1, Employee e2
WHERE e1.ManagerId = e2.Id
  AND e1.Salary > e2.Salary;
```

* 通过逗号分隔的隐式连接也能将表与自身关联。
* 在 `WHERE` 子句中指定 `e1.ManagerId = e2.Id`。
* 过滤条件 `e1.Salary > e2.Salary` 保留工资更高的员工。

### 🧠 收获总结

* 自连接是处理“员工-经理”这种层级关系的核心技巧。
* 显式 `JOIN` 语法清晰易读，更建议使用。
* 隐式连接是早期写法，功能相同但容易混淆。

---

## ✅ 题目：182. 查找重复电子邮件（Find Duplicate Emails）

### 📝 题目描述

给定 Person 表，包含唯一主键 `id` 和不含大写字母的 `email` 字段。
编写 SQL 查询找出所有重复出现的电子邮件（即出现次数超过 1 次的 email）。
结果中只返回重复的 email，顺序不限。

示例 1:

```
输入: 
Person 表:
+----+---------+
| id | email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
输出: 
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

解释: `a@b.com` 出现了两次。

### 💡 解题思路

* 利用 `GROUP BY email` 将相同 email 归为一组。
* 使用 `HAVING COUNT(*) > 1` 筛选出现超过一次的 email。
* 返回这些重复的 email。

### ✅ SQL 解法一（GROUP BY + HAVING）

```sql
SELECT email AS Email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

* `GROUP BY email` 对邮件分组。
* `HAVING COUNT(*) > 1` 过滤出重复的邮件。
* 返回列名 `Email`。

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

* 利用子查询 `EXISTS` 判断是否存在不同 id 的相同 email。
* `EXISTS` 子查询只要能找到符合条件的记录就返回真。
* 使用 `DISTINCT` 去重，确保每个重复 email 只出现一次。

### 🧠 收获总结

* `GROUP BY` 与 `HAVING` 是统计重复值的常用手段。
* `EXISTS` 子查询也可实现重复查找，灵活性高。
* 熟练掌握聚合与子查询，提升查询能力。

---

## ✅ 题目：183. Customers Who Never Order

### 📝 题目描述

Customer 表和 Orders 表，找出所有从未下过订单的客户。结果格式如下所示。

示例 1:

```
输入：
Customers 表:
+----+-------+
| id | name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders 表:
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
```

### 💡 解题思路

* 方法一：使用 `LEFT JOIN` 找出关联 Orders 后为空的客户。
* 方法二：使用 `NOT EXISTS` 子查询排除有订单的客户。

### ✅ SQL 解法一（LEFT JOIN）

```sql
SELECT c.name AS Customers
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.customerId IS NULL;
```

* `LEFT JOIN` 保留所有客户信息。
* `WHERE o.customerId IS NULL` 筛选无关联订单的客户。
* 返回列名 `Customers`。

### ✅ SQL 解法二（NOT EXISTS）

```sql
SELECT name AS Customers
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.customerId = c.id
);
```

* 使用 `NOT EXISTS` 判断该客户是否在 Orders 表中出现过。
* 如果不存在任何满足条件的记录，则该客户从未下单。
* 返回列名 `Customers`。

### 🧠 收获总结

* `LEFT JOIN ... WHERE ... IS NULL` 和 `NOT EXISTS` 是查找“缺失关联”常用技巧。
* `LEFT JOIN` 方法语义直观，`NOT EXISTS` 子查询灵活且效率高。
* 处理外键关系时，掌握这两种方法非常重要。

---

## ✅ 题目：184. Department Highest Salary

### 📝 题目描述

`Employee` 表包含 `id`、`name`、`salary` 和 `departmentId`。
查询每个部门的最高工资员工姓名和工资。

示例 1:

```
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
```

解释：Max 和 Jim 在 IT 部门的工资都是最高，Henry 在 Sales 部门工资最高。

### 💡 解题思路

1. 先将 `Employee` 表与 `Department` 表通过 `departmentId = id` 连接，获得员工-部门的对应关系。
2. 使用子查询找出每个部门的最高工资。
3. 将最高工资与员工表匹配，筛出每个部门中所有达到该工资的员工。

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
* 子查询中对每个 `departmentId` 求最大 `salary`。
* 使用 `(departmentId, salary) IN (...)` 组合键匹配子查询结果，找出部门内最高薪资员工。

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

* 使用 `DENSE_RANK()` 按部门对工资进行排名，`rk = 1` 即为该部门的最高工资。
* 外层查询筛选排名为 1 的记录，即为每个部门的最高薪资员工。

### 🧠 收获总结

* 学会通过 `(col1, col2) IN (子查询)` 实现复合条件匹配。
* 掌握 `DENSE_RANK()` 窗口函数的用法，可方便处理排名相关问题。
* 对分组聚合与多表连接配合使用有了更深入的理解。

---

## ✅ 题目：185. Department Top Three Salaries

### 📝 题目描述

`Employee(Id, Name, Salary, DepartmentId)` 表和 `Department(Id, Name)` 表。查询每个部门前三高的所有员工姓名、薪水和部门名（同薪并列，且并列不跳号）。

示例 1:

```
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
Department 表:
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
```

解释：IT 部门中最高是 Max (90000)，第二高是 Joe 和 Randy (85000)，第三高 Will (70000)；Sales 部门中最高是 Henry (80000)，第二高 Sam (60000)，没有第三高。

### 💡 解题思路

1. 使用窗口函数：对每个部门按薪水降序，使用 `DENSE_RANK()` 给行打标签，取排名 ≤ 3 的记录。
2. 使用子查询计数：对每行在本部门中，比它薪水高的不同值个数小于 3，即排在前三。

### ✅ SQL 解法一（窗口函数）

```sql
SELECT d.Name AS Department,
       e.Name AS Employee,
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

* 对 `Employee` 表使用窗口函数按部门给薪水排名。
* `rk <= 3` 取前三名（包括并列）。
* 外层连接 `Department` 表获取部门名，按部门名和薪水排序。

### ✅ SQL 解法二（子查询计数）

```sql
SELECT d.Name AS Department,
       e.Name AS Employee,
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

* 子查询统计在同部门中高于当前记录的不同薪水数。
* 若该计数小于 3，则当前记录属于前 3 名。
* 这样无需使用窗口函数，同样适用于不支持窗口函数的数据库。

### 🧠 收获总结

* 窗口函数写法简洁灵活，可轻松调整排名阈值。
* 子查询计数法通用，但相对复杂，适用于各种 SQL 环境。
* 掌握并列排名和前 N 名的两种通用思路。

---

## ✅ 题目：196. Delete Duplicate Emails

### 📝 题目描述

Person(Id, Email) 表。删除所有重复的邮箱记录，只保留每个邮箱最小 Id 的那条，删除后输出完整表。

示例 1:

```
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
```

解释: `john@example.com` 重复两次，保留最小的 Id = 1。

### 💡 解题思路

1. **自连接 DELETE**：将表与自身连接，删除较大 Id 的记录。
2. **子查询 DELETE**：保留每组最小 Id，其余删除。

### ✅ SQL 解法一（自连接 DELETE）

```sql
DELETE p1
FROM Person p1
JOIN Person p2 
  ON p1.Email = p2.Email 
  AND p1.Id > p2.Id;
```

* 自连接 Person 表，条件 `p1.Email = p2.Email AND p1.Id > p2.Id`。
* 删除 `p1` 中 Id 较大的记录，保留最小 Id。

### ✅ SQL 解法二（子查询 DELETE）

```sql
DELETE FROM Person
WHERE Id NOT IN (
    SELECT MIN(Id)
    FROM Person
    GROUP BY Email
);
```

* 子查询 `SELECT MIN(Id) FROM Person GROUP BY Email` 找到每个邮箱的最小 Id。
* 外层删除不在该结果集中的所有记录，即删除重复的较大 Id。

### 🧠 收获总结

* 自连接删除法直观高效。
* 子查询删除法通用，但有些数据库（如 MySQL）需要分步执行。
* 理解 DELETE 与 SELECT 聚合结合的使用方式。

---

## ✅ 题目：197. Rising Temperature

### 📝 题目描述

Weather(Id, RecordDate, Temperature) 表。找出温度比前一天更高的所有日期 Id。

示例 1:

```
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
```

解释：2015-01-02 的温度（25）比前一天高（10），2015-01-04 的温度（30）比前一天高（20）。

### 💡 解题思路

* 方法一：**自连接**。将表自身连接，关联日期相差 1 天的两行，再比较温度。
* 方法二：**窗口函数**。使用 `LAG(Temperature)` 获取前一天温度，再比较。

### ✅ SQL 解法一（自连接）

```sql
SELECT w1.Id
FROM Weather w1
JOIN Weather w2 
  ON DATEDIFF(w1.RecordDate, w2.RecordDate) = 1
WHERE w1.Temperature > w2.Temperature;
```

* 对同一表 `Weather` 进行自连接，条件是日期差为 1 天。
* 比较温度，筛选温度更高的日期。

### ✅ SQL 解法二（窗口函数）

```sql
SELECT Id
FROM (
    SELECT Id, Temperature,
           LAG(Temperature) OVER (ORDER BY RecordDate) AS prev_temp
    FROM Weather
) t
WHERE Temperature > prev_temp;
```

* 窗口函数 `LAG(Temperature) OVER (ORDER BY RecordDate)` 得到前一天温度。
* 在外层查询中过滤出当天温度高于 `prev_temp` 的行。

### 🧠 收获总结

* 自连接法简单直观，但依赖日期函数 `DATEDIFF()`，适用于老版本数据库。
* 窗口函数法更简洁，对时间序列数据尤其高效。
* 掌握这两种方法，提高对日期序列问题的处理能力。

---

## ✅ 题目：262. Trips and Users

### 📝 题目描述

取消率的计算方式为：(**被司机或乘客取消的非禁止用户生成的订单数量**) / (**非禁止用户生成的订单总数**)。
编写 SQL 查询，找出 2013-10-01 至 2013-10-03 期间有至少一次行程的非禁止用户（乘客和司机都必须未被禁止）的取消率。非禁止用户指 `Users.banned = 'No'`。取消率保留两位小数。

示例 1:

```
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
```

取消率计算：每一天取消次数 / 总行程数，其中乘客和司机都不能是被禁止的用户，结果保留两位小数。

### 💡 解题思路

* 先筛选出 `banned = 'No'` 的乘客和司机。
* 将 `Trips` 表与 `Users` 表分别连接（分别作为乘客和司机），筛选出双方都是非禁止用户的行程。
* 对日期范围内的有效行程，统计每天的取消数和总数，计算取消率。
* 使用 `CASE WHEN` 对取消状态计数，用 `ROUND(..., 2)` 保留两位小数。

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

* 连接 `Users` 表两次：一次筛选非禁止乘客，一次筛选非禁止司机。
* `status != 'completed'` 表示被取消的行程。
* `ROUND(..., 2)` 保留两位小数。
* 按天分组统计。

### ✅ SQL 解法二（使用 CTE 分层过滤）

```sql
WITH ValidTrips AS (
    SELECT *
    FROM Trips t
    JOIN Users c ON t.client_id = c.users_id AND c.banned = 'No'
    JOIN Users d ON t.driver_id = d.users_id AND d.banned = 'No'
    WHERE t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
)
SELECT request_at AS Day,
       ROUND(
         SUM(CASE WHEN status IN ('cancelled_by_driver','cancelled_by_client') THEN 1 ELSE 0 END)
         / COUNT(*), 2
       ) AS `Cancellation Rate`
FROM ValidTrips
GROUP BY request_at;
```

* 使用 CTE 提前筛选出双方均为非禁止用户的有效行程。
* 在主查询中计算每天的取消率。
* `status IN (...)` 判断是否取消，提高可读性。

### 🧠 收获总结

* 熟练掌握多次连接同一张表（玩家/司机筛选）的写法。
* 使用 `CASE WHEN` 结合 `SUM`/`COUNT` 实现自定义统计逻辑。
* 掌握 `ROUND` 和 `GROUP BY` 在汇总统计中的应用。
* CTE 的使用提高了查询结构的清晰度。

---

## ✅ 题目：511. Game Play Analysis I

### 📝 题目描述

给定活动表 `Activity`，记录玩家在游戏平台上的每日行为：

* `player_id`：玩家 ID
* `device_id`：登录设备
* `event_date`：行为发生日期
* `games_played`：当日打开的游戏数（可能为 0）
  主键是 `(player_id, event_date)`。查询每位玩家第一次登录平台的日期。

示例 1:

```
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
```

查询每位玩家 **第一次登录平台的日期**。

### 💡 解题思路

* 每个玩家可能有多条记录。
* 使用 `GROUP BY player_id` 分组，然后对每组取最小的 `event_date` 即为首次登录日期。
* 结果中只返回 `player_id` 和首次登录日期列，命名为 `first_login`。

### ✅ SQL 解法一（GROUP BY + MIN）

```sql
SELECT player_id,
       MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

* `GROUP BY player_id` 将记录按玩家分组。
* `MIN(event_date)` 提取该玩家的最早登录日期。
* 返回列重命名为 `first_login`。

### ✅ SQL 解法二（窗口函数）

```sql
SELECT DISTINCT player_id,
       FIRST_VALUE(event_date) OVER (
           PARTITION BY player_id ORDER BY event_date
       ) AS first_login
FROM Activity;
```

* `FIRST_VALUE(event_date)` 窗口函数获取每个玩家按时间排序后的第一条日期。
* 使用 `DISTINCT` 去除重复的 `player_id`。
* 这种方法在需要保留其他列或更复杂条件时非常灵活。

### 🧠 收获总结

* 掌握 `GROUP BY + MIN()` 实现分组最小值查询。
* 理解窗口函数 `FIRST_VALUE()` 可以直接取每组第一行信息的优势。
* 熟悉数据去重与聚合的组合方式，适用于分析日志类问题。

---

## ✅ 题目：550. Game Play Analysis IV

### 📝 题目描述

给定玩家行为记录表 `Activity`（字段同 511 题）。请计算 **在首次登录的第二天再次登录的玩家比例**，保留小数点后两位，字段命名为 `fraction`。即：在所有玩家中，有多少人在首次登录后的第二天也进行了登录。

示例 1:

```
输入：
Activity 表:
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
```

解释：只有 ID 为 1 的玩家在第一天登录后次日登录，比例为 1/3 = 0.33。

### 💡 解题思路

* 找出每位玩家的首次登录日期。
* 筛选出所有在“首次登录的第二天”有登录记录的玩家。
* 计算满足条件的玩家数除以总玩家数，结果保留两位小数。

### ✅ SQL 解法（JOIN + 子查询）

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

* 子查询 `b` 得到每位玩家的首次登录时间。
* 将活动表 `a` 与 `b` 按 `player_id` 连接。
* `DATEDIFF(a.event_date, b.first_login) = 1` 精确筛选次日登录的记录。
* 分子使用 `COUNT(DISTINCT a.player_id)` 统计符合条件的玩家数。
* 分母单独统计总玩家数，避免 `JOIN` 影响基数。
* 使用 `ROUND(..., 2)` 保留两位小数。

### 🧠 收获总结

* 使用 `DATEDIFF()` 判断具体日期差是处理次日登录问题的关键。
* 分子和分母统计范围不同，分母不能被 `JOIN` 过滤。
* 掌握 `JOIN + 子查询 + COUNT(DISTINCT)` 的组合使用，适合类似行为分析题目。
* 注意精度处理，`ROUND(..., 2)` 在面试题中常见细节。

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

```
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
```

解释：John 拥有 5 位直接下属（102,103,104,105,106）。

### 💡 解题思路

* 对 `managerId` 分组统计每位经理拥有的下属数量。
* 使用 `HAVING COUNT(*) >= 5` 筛出符合条件的经理 `id`。
* 将这些 `managerId` 与 `Employee` 表自身连接，取出对应的经理姓名。

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

* 内层子查询：对 `managerId` 分组并统计下属数量，使用 `HAVING COUNT(*) >= 5` 筛选符合条件的经理 `id`。
* 外层查询将这些 `managerId` 与员工表连接，取出经理的 `name` 字段。

### 🧠 收获总结

* 熟练掌握“自身连接”技巧：一个表对自己的查询。
* `GROUP BY managerId` 是统计直属下属数量的经典用法。
* `HAVING` 用于对聚合结果筛选条件。
* `IS NOT NULL` 避免统计没有经理的记录。

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

示例 1:

```
输入：
Employee 表:
+-------+--------+------------+--------+
| empId | name   | supervisor | salary |
+-------+--------+------------+--------+
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |
+-------+--------+------------+--------+
Bonus 表:
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
输出:
+------+-------+
| name | bonus |
+------+-------+
| Brad | null  |
| John | null  |
| Dan  | 500   |
+------+-------+
```

解释：Brad、John 没有奖金记录；Dan 的奖金 500；John 有奖金但为 1000（不小于 1000），所以也不满足条件。

### 💡 解题思路

* 使用 `LEFT JOIN` 将 `Employee` 和 `Bonus` 表关联，保证所有员工都显示。
* 过滤条件为：奖金少于 1000 或无奖金记录（`bonus < 1000 OR bonus IS NULL`）。
* 返回员工姓名和奖金。

### ✅ SQL 解法（LEFT JOIN + 条件筛选）

```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

* `LEFT JOIN` 保证将所有员工列出，即使没有奖金信息。
* `WHERE b.bonus < 1000 OR b.bonus IS NULL` 过滤出奖金小于 1000 或没有奖金的情况。
* 输出列 `name` 和 `bonus`。

### 🧠 收获总结

* `LEFT JOIN` 用于确保主表（员工）所有记录都被保留。
* 处理 `NULL` 时，`IS NULL` 用于判断是否缺乏对应记录。
* 注意包含 `OR b.bonus IS NULL`，否则无奖金员工会被排除。
* 掌握表连接与条件筛选的结合使用。

---

## ✅ 题目：584. Find Customers Who Never Order

### 📝 题目描述

给定客户表 `Customer`：

* `id`：客户唯一 ID
* `name`：客户姓名
* `referee_id`：推荐该客户的客户 ID（可能为 NULL）

请找出 **没有被 id = 2 的客户推荐的客户的姓名**。

示例 1:

```
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
输出:
+------+
| name |
+------+
| Will |
| Jane |
| Bill |
| Zack |
+------+
```

解释：ID 为 3 和 6 的客户被客户 2 推荐，应排除；其余客户均满足条件。

### 💡 解题思路

* 筛选出所有客户中 `referee_id <> 2 OR referee_id IS NULL` 的记录。
* 排除被客户 2 推荐的客户，保留其余。

### ✅ SQL 解法（条件过滤）

```sql
SELECT name
FROM Customer
WHERE referee_id <> 2 OR referee_id IS NULL;
```

* `referee_id <> 2` 排除被 2 推荐的客户。
* `referee_id IS NULL` 包括没有推荐人的客户。
* 结果无特定排序要求。

### 🧠 收获总结

* `NULL` 在比较中不等于任何值，因此需要单独判断。
* 使用 `OR` 将多个条件结合保证全面筛选。
* 理解外键 `referee_id` 的含义，有助于类似推荐关系的查询。

---

## ✅ 题目：585. Investment in 2016

### 📝 题目描述

给定保险表 `Insurance`，包含字段：

* `pid`：投保编号（主键）
* `tiv_2015`：2015 年投保总金额
* `tiv_2016`：2016 年投保总金额
* `lat, lon`：投保人所在城市的经纬度（均不为空）

请统计满足以下条件的投保人在 2016 年的投保金额总和（保留两位小数）：

1. 该投保人在 2015 年的投保金额 `tiv_2015` 至少与另一名投保人相同。
2. 该投保人所在城市的 `(lat, lon)` 与其他任何投保人均不同（即该城市唯一）。

示例 1:

```
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
```

解释：记录 1 和 4 满足条件：

* tiv\_2015 = 10 的投保人与记录 3 的 tiv\_2015 相同，且 (10,10) 是唯一城市；
* 记录 4 的 tiv\_2015 = 10 与记录 1、3 相同，且 (40,40) 唯一。
  它们的 tiv\_2016 总和为 5 + 40 = 45。

### 💡 解题思路

* 使用自连接判断每个投保人的 `tiv_2015` 是否有另一个相同的。
* 使用分组统计 `(lat, lon)` 是否唯一。
* 筛选出同时满足两个条件的投保人，求其 `tiv_2016` 之和并保留两位小数。

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
      ON a.tiv_2015 = b.tiv_2015
     AND a.pid <> b.pid
)
SELECT ROUND(SUM(i.tiv_2016), 2) AS tiv_2016
FROM Insurance i
JOIN UniqueCity uc ON i.lat = uc.lat AND i.lon = uc.lon
JOIN SameTiv2015 st ON i.pid = st.pid;
```

* `UniqueCity` 筛选唯一城市 `(lat, lon)`。
* `SameTiv2015` 筛选 `tiv_2015` 与其他投保人相同的记录。
* 将两者结果取交集后计算符合条件的 `tiv_2016` 总和。
* 使用 `ROUND(..., 2)` 保留两位小数。

### 🧠 收获总结

* 利用 CTE (`WITH`) 提高查询可读性并分步筛选。
* 自连接筛选满足“存在另一个相同 `tiv_2015`”的投保人。
* `GROUP BY ... HAVING COUNT(*) = 1` 用于筛选唯一城市。
* 多条件交集的实现方式更加清晰。

---

## ✅ 题目：586. Customer With the Most Orders

### 📝 题目描述

给定订单表 `Orders`：

* `order_number`：订单唯一编号（主键）
* `customer_number`：客户编号

请找出**下订单最多的客户的 `customer_number`**。题目保证只有一个客户的订单数最多。

示例 1:

```
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
```

解释：顾客 3 的订单数最多 (2 个)。

### 💡 解题思路

1. 对 `customer_number` 分组，统计每个客户的订单数量。
2. 找出订单数量的最大值。
3. 返回拥有最大订单数的客户编号。

### ✅ SQL 解法一（分组 + LIMIT）

```sql
SELECT customer_number
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;
```

* 按 `customer_number` 分组并按订单数降序排序，取第一条记录即可。

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

* 子查询 `t` 计算每个客户的订单数，找出最大订单数。
* 外层查询筛选订单数等于最大值的客户。

### 🧠 收获总结

* `ORDER BY ... LIMIT 1` 是直观查找最大值的方法。
* 子查询+`HAVING` 在 SQL 中也可实现寻找最大分组的方法。
* 练习了分组聚合和子查询的结合使用。

---

## ✅ 题目：595. Big Countries

### 📝 题目描述

**World(name, continent, area, population, gdp)** 表。查询“大国家”的名称、人口和面积，定义为：面积超过 300 万或人口超过 2500 万。

示例见题目说明。

### 💡 解题思路

* 在 `WHERE` 中直接用 `OR` 连接面积和人口条件。
* 也可拆成两个子查询用 `UNION` 合并。

### ✅ SQL 解法一（条件过滤）

```sql
SELECT name, population, area
FROM World
WHERE area > 3000000 OR population > 25000000;
```

* 简单的逻辑 `OR` 条件过滤。

### 🧠 收获总结

* 逻辑“或”条件可以直接用 `OR` 拼接。
* 可以用 `UNION` 拆分复杂条件，但语句更长。
* 根据实际需要选择直接条件或拆分两次查询。

---

## ✅ 题目：596. Classes More Than 5 Students

### 📝 题目描述

**Courses(student, class)** 表。查询至少有5名不同学生参加的课程名称。

示例见题目说明。

### 💡 解题思路

* 按课程分组，统计不同学生数。
* 使用 `COUNT(DISTINCT student)` 实现去重计数。

### ✅ SQL 解法一（分组过滤）

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5;
```

* `GROUP BY class` 对课程分组。
* `HAVING COUNT(DISTINCT student) >= 5` 保留不同学生数不少于 5 的课程。

### 🧠 收获总结

* 掌握分组聚合结合 `COUNT(DISTINCT)` 实现去重计数。
* `HAVING` 子句用于筛选聚合后的结果。
* 去重计数是统计不同元素数量的常见需求。

---

## ✅ 题目：601. Stadium Attendance

### 📝 题目描述

**Stadium(id, visit\_date, people)** 表记录每日人流量。找出所有在任意连续三行及以上 `id` 均满足 `people >= 100` 的记录，按 `visit_date` 升序输出。

示例见题目说明。

### 💡 解题思路

* 使用窗口函数的行号差法标记连续区间。
* 如果多个连续满足条件的行属于同一组（`id - ROW_NUMBER()` 相同），则这些行连续。
* 筛选组内行数 ≥ 3 的区间，输出对应记录。

### ✅ SQL 解法一（窗口函数）

```sql
SELECT sta.id, sta.visit_date, sta.people
FROM (
    SELECT MIN(id) AS minid, MAX(id) AS maxid, grp
    FROM (
        SELECT *, id - ROW_NUMBER() OVER (ORDER BY id) AS grp
        FROM Stadium
        WHERE people >= 100
    ) t
    GROUP BY grp
    HAVING COUNT(*) >= 3
) g
JOIN Stadium sta ON sta.id BETWEEN g.minid AND g.maxid
ORDER BY sta.visit_date;
```

* 内层子查询通过 `ROW_NUMBER()` 计算连续区间的标记 `grp`。
* 外层对 `grp` 分组，`HAVING COUNT(*) >= 3` 过滤出长度至少 3 的区间。
* 再连接回原表取出所有满足条件的记录。

### 🧠 收获总结

* 行号差法（`id - ROW_NUMBER()`）巧妙地为连续段打组号。
* 窗口函数在处理时间或序列问题时非常有用。
* 这种方法无需考虑隔号的复杂逻辑，只关注连续性。

---

## ✅ 题目：620. Not Boring Movies

### 📝 题目描述

**Cinema(id, movie, description, rating)** 表。影片描述为 **boring** 的不感兴趣。找出描述不为 “boring” 且 `id` 为奇数的影片，按 `rating` 降序排列。

示例 1:

```
输入:
+----+-----------+------------+-------+
| id | movie     | description| rating|
+----+-----------+------------+-------+
| 1  | War       | great 3D   | 8.9   |
| 2  | Science   | fiction    | 8.5   |
| 3  | Irish     | boring     | 6.2   |
| 4  | Ice Song  | Fantasy    | 8.6   |
| 5  | House    | Interesting| 9.1   |
+----+-----------+------------+-------+
输出:
+----+-----------+------------+-------+
| id | movie     | description| rating|
+----+-----------+------------+-------+
| 5  | House     | Interesting| 9.1   |
| 1  | War       | great 3D   | 8.9   |
+----+-----------+------------+-------+
```

解释：只保留 id 为奇数且 description 不为 boring 的电影，按评分降序。

### 💡 解题思路

* 在 `WHERE` 中同时过滤 `description <> 'boring'` 和 `id % 2 = 1`。
* 最后按 `rating DESC` 排序。

### ✅ SQL 解法一（条件过滤）

```sql
SELECT id, movie, description, rating
FROM Cinema
WHERE description <> 'boring' AND id % 2 = 1
ORDER BY rating DESC;
```

* 直接在 `WHERE` 子句中组合两个条件。
* `ORDER BY rating DESC` 按评分降序排列。

### 🧠 收获总结

* 简单的条件过滤与排序即可满足需求。
* 注意 `%` 运算符处理取模筛选。
* 练习了文本匹配与数值筛选的结合。

---

## ✅ 题目：626. Swap Seats

### 📝 题目描述

**Seat(id, student)** 表记录教室座位（`id` 连续自增）。小美希望交换相邻两位同学的座位，若学生总数为奇数，则最后一人座位不变。输出按座位 `id` 重排后的学生列表。

示例 1:

```
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
```

解释：原本 (1, Abbot) 与 (2, Doris) 交换；(3, Emerson) 与 (4, Green) 交换；(5, Jeames) 保持不动。

### 💡 解题思路

* 使用窗口函数 `LEAD` 和 `LAG` 获取相邻行学生姓名。
* 对奇数 `id` 行，用 `LEAD(student)` 取下一行学生名；对偶数 `id` 行，用 `LAG(student)` 取上一行学生名。
* 对最后一条奇数行（无下一行）使用 `COALESCE` 保持原值。

### ✅ SQL 解法一（窗口函数 + CASE）

```sql
SELECT
    id,
    CASE
        WHEN id % 2 = 1 THEN COALESCE(LEAD(student) OVER (ORDER BY id), student)
        ELSE LAG(student) OVER (ORDER BY id)
    END AS student
FROM Seat;
```

* `LEAD(student) OVER (ORDER BY id)` 得到下一行学生名称。
* `LAG(student) OVER (ORDER BY id)` 得到上一行学生名称。
* 使用 `CASE` 判断 `id` 奇偶并选择相应的窗口函数。
* 对最后一条奇数行使用 `COALESCE` 保证原值不丢失。

### 🧠 收获总结

* 窗口函数 `LEAD` 和 `LAG` 非常适合处理相邻行数据。
* `COALESCE` 确保最后一个奇数行学生姓名不变。
* 将 `CASE` 与窗口函数结合，实现了奇偶行的不同操作。

---

## ✅ 题目：627. Swap Gender

### 📝 题目描述

**Salary(id, name, sex, salary)** 表。将 `sex` 列中 `'m'` 和 `'f'` 互换，所有男生变女生，女生变男生。只允许使用一条 `UPDATE` 语句。

示例 1:

```
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
```

解释：`'m'` 和 `'f'` 互换。

### 💡 解题思路

* 使用 `CASE WHEN` 或 `IF` 来在更新时条件替换。
* 在一条 `UPDATE` 语句中完成所有行的更新。

### ✅ SQL 解法一（条件更新）

```sql
UPDATE Salary
SET sex = CASE sex WHEN 'm' THEN 'f' ELSE 'm' END;
```

* 利用 `CASE` 表达式，根据 `sex` 字段的当前值进行替换。
* 该语句一次性将所有记录更新完成，无需分多次。

### 🧠 收获总结

* 单条 `UPDATE` 语句通过 `CASE WHEN` 实现数据互换。
* 条件表达式灵活替代多语句操作。
* 掌握在更新语句中使用条件逻辑。

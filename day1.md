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

✅ **题目：180. Consecutive Numbers**
📝 **题目描述**
判断表中是否存在三个连续的数字 num，使得这三个数字在表中均存在（num, num+1, num+2）。

返回 1 表示存在，0 表示不存在。

💡 **解题思路**
使用自连接三次分别查找 num、num+1 和 num+2 是否都存在。

若存在则返回 1，否则返回 0。

✅ **SQL 关键点**

* 自连接三次，条件分别是 t1.num = t2.num - 1 = t3.num - 2。
* EXISTS 语句检测是否满足条件。
* CASE WHEN 控制返回结果 1 或 0。

```sql
SELECT CASE WHEN EXISTS (
    SELECT 1 
    FROM Numbers t1
    JOIN Numbers t2 ON t2.num = t1.num + 1
    JOIN Numbers t3 ON t3.num = t1.num + 2
) THEN 1 ELSE 0 END AS Consecutive;
```

🧠 **收获总结**

* 自连接实现复杂的序列匹配。
* 用 EXISTS 判断是否存在符合条件的记录。
* 灵活运用 CASE 语句处理结果输出。
---

✅ **题目：183. Customers Who Never Order**
📝 **题目描述**
Customer 表和 Orders 表，找出所有从未下过订单的客户。

💡 **解题思路**

* 方法一：使用 LEFT JOIN 找出订单为 null 的客户。
* 方法二：使用 NOT EXISTS 子查询排除有订单的客户。

**方法一：LEFT JOIN**

```sql
SELECT c.CustomerId, c.Name
FROM Customer c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE o.OrderId IS NULL;
```

**方法二：NOT EXISTS**

```sql
SELECT CustomerId, Name
FROM Customer c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o WHERE o.CustomerId = c.CustomerId
);
```

🧠 **收获总结**

* LEFT JOIN 结合 NULL 判断实现反向筛选。
* NOT EXISTS 是典型的反向查询写法。
* 理解两种方式效率差异及应用场景。

---

✅ **题目：184. Department Highest Salary**
📝 **题目描述**
Employee 表包含 id、name、salary 和 departmentId。
查询每个部门的最高工资员工姓名和工资。

💡 **解题思路**

* 方法一：使用子查询先找部门最高工资，再JOIN回主表。
* 方法二：使用窗口函数 RANK() 或 ROW\_NUMBER() 找每部门排名第一员工。

**方法一：子查询**

```sql
SELECT e.Name, e.Salary, e.DepartmentId
FROM Employee e
JOIN (
    SELECT DepartmentId, MAX(Salary) AS MaxSalary
    FROM Employee
    GROUP BY DepartmentId
) d ON e.DepartmentId = d.DepartmentId AND e.Salary = d.MaxSalary;
```

**方法二：窗口函数**

```sql
SELECT Name, Salary, DepartmentId
FROM (
    SELECT Name, Salary, DepartmentId,
           RANK() OVER (PARTITION BY DepartmentId ORDER BY Salary DESC) as rnk
    FROM Employee
) t
WHERE rnk = 1;
```

🧠 **收获总结**

* 子查询+JOIN适合基本SQL。
* 窗口函数写法更简洁灵活。
* 学习窗口函数对分组排序的强大能力。

---
下面继续按统一格式完成剩余的 SQL 题目整理，包括 185、196、197、511 四题。如有第 256 题的具体题名，请告知，我将补充进去。

---

✅ **题目：185. Department Top Three Salaries**
📝 **题目描述**
Employee(Id, Name, Salary, DepartmentId) 表和 Department(Id, Name) 表。查询每个部门前三高的所有员工姓名、薪水和部门名（同薪并列，且并列不跳号）。

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

**Trips(Id, Client\_Id, Driver\_Id, City\_Id, Status, Request\_at)** 和 **Users(Users\_Id, Banned, Role)** 表。统计 2013-10-01 至 2013-10-03 期间非禁止用户（乘客和司机均未被禁止）的出行订单的“取消率”。取消率 = 非禁止订单中被取消的比例（乘客或司机取消都算取消）。结果按日期输出，保留两位小数。

### 💡 解题思路

* 筛选日期范围内且乘客、司机都非禁止（`Banned='No'`）的订单。
* 使用两次 `JOIN` 过滤被禁止的用户。
* 用条件聚合统计取消订单数（`Status <> 'completed'`）。
* 计算取消率，保留两位小数。

### ✅ SQL 关键点

* 双 `JOIN` Users 表过滤乘客和司机状态。
* 使用 `SUM(CASE WHEN ...)` 或 `COUNT(CASE WHEN ...)` 实现条件计数。
* 按 `Request_at` 分组输出。

```sql
SELECT t.Request_at AS `Day`,
       ROUND(
         SUM(CASE WHEN t.Status <> 'completed' THEN 1 ELSE 0 END) / COUNT(*), 2
       ) AS Cancellation_Rate
FROM Trips t
JOIN Users u1 ON t.Client_Id = u1.Users_Id AND u1.Banned = 'No'
JOIN Users u2 ON t.Driver_Id = u2.Users_Id AND u2.Banned = 'No'
WHERE t.Request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY t.Request_at;
```

### 🧠 收获总结

* 学会对比例计算使用条件聚合。
* 掌握联合过滤多表条件。
* 先筛选数据再聚合，确保计算准确。

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

### 💡 解题思路

* 对奇偶行学生姓名交换。
* 窗口函数 `LEAD` 和 `LAG` 方便获取相邻行数据。
* 奇数行取下一行学生，偶数行取上一行。

### ✅ SQL 关键点

* 使用 `CASE` 判断奇偶行。
* `LEAD(student) OVER (ORDER BY id)` 和 `LAG(student) OVER (ORDER BY id)`。

```sql
SELECT id,
       CASE WHEN id % 2 = 1 THEN LEAD(student) OVER (ORDER BY id)
            ELSE LAG(student) OVER (ORDER BY id)
       END AS student
FROM Seat;
```

### 🧠 收获总结

* 窗口函数适合行内相邻数据处理。
* 巧用 `CASE` 结合 `LEAD/LAG` 实现行交换。

---

## ✅ 题目：627. Swap Gender

### 📝 题目描述

**Salary(id, name, sex, salary)** 表。将 `sex` 列中 'm' 和 'f' 互换，所有男生变女生，女生变男生。只允许使用一条 `UPDATE` 语句。

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





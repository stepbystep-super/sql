
# SQL 错题笔记：DB2 - 输出职业名称与职业数量统计（The PADS）

## 📘 题目描述

本题要求你基于 OCCUPATIONS 表，输出两类信息：

### 1️⃣ 所有人的名字，后跟职业的首字母（大写），格式如下：
```
AnActorName(A)
```

### 2️⃣ 每种职业出现的次数，输出格式如下（按职业名升序排列）：
```
There are a total of [count] [occupation]s.
```

---

## 🧾 表结构

表名：`OCCUPATIONS`

| 列名       | 类型   |
| ---------- | ------ |
| Name       | String |
| Occupation | String |

职业只有以下几类值：`Doctor`, `Professor`, `Singer`, `Actor`

---

## ❌ 错误代码（初始尝试）

```sql
SELECT Name & "(" & SUBSTR(Name,1,1) & ")"
FROM OCCUPATIONS
ORDER BY Name;
```

```sql
SELECT
    "There are a total of " &
    COUNT(Name) OVER(Occupation) &
    " " &
    DISTINCT LOWER(Occupation) &
    "s"
FROM OCCUPATIONS;
```

### ❗ 错误分析

| 问题点                          | 原因                                        |
| ------------------------------- | ------------------------------------------- |
| 使用 `&` 拼接字符串             | ❌ DB2 中拼接字符串应该使用 `||` 而不是 `&` |
| `COUNT(...) OVER(...)` 错误用法 | 不适合当前格式需求，应使用 `GROUP BY`       |
| `DISTINCT` 放在表达式中         | 语法错误，不能直接加在字符串拼接中          |

---

## ✅ 正确写法（DB2 Accepted 版本）

```sql
-- Part 1: 输出名称 + 职业首字母
SELECT Name || '(' || SUBSTR(Occupation,1,1) || ')'
FROM OCCUPATIONS
ORDER BY Name;

-- Part 2: 输出职业统计信息
SELECT 'There are a total of ' || COUNT(*) || ' ' || LOWER(Occupation) || 's.'
FROM OCCUPATIONS
GROUP BY Occupation
ORDER BY COUNT(*), Occupation;
```

---

## 🧠 总结：DB2 字符串拼接与聚合技巧

| 需求             | DB2 写法                       |
| ---------------- | ------------------------------ |
| 字符串拼接       | `字段1 || 字段2`               |
| 截取字符串首字母 | `SUBSTR(字段, 1, 1)`           |
| 小写转换         | `LOWER(字段)`                  |
| 分组计数         | `GROUP BY ...` + `COUNT(*)`    |
| 拼接字符串和数值 | `'文本' || COUNT(*) || '文本'` |

---

## ✅ 教训点

- ❌ 不要在 DB2 中用 `&` 拼接字符串
- ✅ 聚合时用 `GROUP BY` 而不是 `OVER(...)` 进行分组计数
- ✅ 所有拼接都应使用 `||`
- ✅ 字段拼接最好显式使用 `SUBSTR()`、`LOWER()` 提高可读性

---

## 🔚 最终效果示例输出

```
Aakash(D)
Ashley(P)
...
There are a total of 2 doctors.
There are a total of 3 professors.
There are a total of 4 singers.
There are a total of 4 actors.
```

---

## 🗂️ 标签

`#DB2` `#字符串拼接` `#聚合函数` `#GROUPBY` `#错误拼接符号` `#HackerRank`

---
👇 使用 COUNT() OVER(PARTITION BY Occupation)：
```sql
Copy
Edit
SELECT Name, Occupation,
       COUNT(*) OVER(PARTITION BY Occupation) AS Occ_Count
FROM OCCUPATIONS;
```
📌 输出：

| Name  | Occupation | Occ_Count |
| :---- | :--------- | :-------- |
| Anna  | Doctor     | 2         |
| Bob   | Doctor     | 2         |
| Carol | Singer     | 1         |

✔️ 每一行都保留了，适合展示“每个人 + 该职业人数”

👇 使用 GROUP BY：
```sql

SELECT Occupation, COUNT(*) AS Occ_Count
FROM OCCUPATIONS
GROUP BY Occupation;
```
📌 输出：

| Occupation | Occ_Count |
| :--------- | :-------- |
| Doctor     | 2         |
| Singer     | 1         |

✔️ 只返回聚合后的结果，适合展示“每个职业有多少人”


这个问题非常好！你提到的 SQL：

```sql

SELECT 'There are a total of ' || COUNT(*) AS occurencies ||
       ' ' || LOWER(Occupation) || 's.'
FROM OCCUPATIONS
GROUP BY Occupation
ORDER BY occurencies;
```
看起来像只有一点小问题，但其实是两个 经典语法陷阱叠加，我们来逐个拆解：

❌ 错误 1:

SQL 不允许你在 || 表达式中间插入 AS。别名 AS occurencies 只能放在 整条表达式的末尾：

❌ 错误 2：

```sql

SELECT 'There are a total of ' || COUNT(*) || ' ' || LOWER(Occupation) || 's.' AS occurencies
FROM OCCUPATIONS
GROUP BY Occupation
ORDER BY occurencies;
```
仍然可能报错（尤其在 DB2）！

在 DB2（以及部分 SQL 实现）中，在 ORDER BY 中不能直接引用别名，尤其当这个别名本身是由表达式构成的。

✅ 推荐的做法是：
直接用表达式排序：

```sql
ORDER BY COUNT(*), Occupation;

或者用子查询包装：


SELECT * FROM (
    SELECT 'There are a total of ' || COUNT(*) || ' ' || LOWER(Occupation) || 's.' AS occurencies,
           COUNT(*) AS count_val, Occupation
    FROM OCCUPATIONS
    GROUP BY Occupation
) AS t
ORDER BY count_val, Occupation;


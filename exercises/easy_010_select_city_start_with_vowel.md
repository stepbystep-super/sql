## 🧠 题目名称：
Weather Observation Station 5（筛选首字母为元音的城市）

---

## 📌 题目要求：

从 `STATION` 表中：

- 选出所有以元音字母（A, E, I, O, U）开头的城市名
- 不区分大小写
- 输出字段：`CITY`，去重

---

## 🧪 数据表结构：

| 字段名 | 类型 |
|--------|------|
| CITY   | VARCHAR2(21) |

---

## ❗ 我的卡点：


- ❌ 错误地写成了 `STRING(CITY, 1, 1)`（应为 `SUBSTR()` 或 `LEFT()` SUBSTR()! 不是SUBSTRING()！
- ❌ 用了 Python 的 `['a','e']` 语法而非 SQL 的 `('a','e')`
- ❌ 没意识到可以直接用 `LIKE 'A%'` 实现

---

## ✅ 我的原始写法（错误）：

```sql
SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, 1, 1)) IN ('a', 'e', 'i', 'o', 'u');


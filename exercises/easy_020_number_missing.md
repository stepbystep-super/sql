
# SQL 错题笔记：DB2 平均工资误差（去 0 问题）

## 📘 题目描述

Samantha 想计算所有员工的平均工资，但由于她的键盘上的 "0" 键坏了，她在计算平均工资时，错误地将所有工资中的 `0` 字符删除再求平均。

你的任务是写出 SQL，计算：

```
正确的平均工资 - 去掉 0 后的错误平均工资
```

并将差值向上取整为最接近的整数。

---

## ❌ 错误示例（你之前写的代码）

```sql
SELECT CAST(ROUND(AVG(Salary)-AVG(CAST(REPLACE(CAST(Salary AS CHARACTER),0,'')AS NUMERIC)),0) AS INTEGER)
FROM EMPLOYEES;
```

### ❗错误点分析

| 问题点 | 原因 |
|--------|------|
| `CHARACTER` 不合法 | DB2 中应使用 `CHAR()` 函数或 `VARCHAR` |
| 缺少 `TRIM` | 若结果含空格，DB2 会报 truncation 警告（SQL0445W） |
| `ROUND(..., 0)` 不符合题目要求 | 应使用 `CEIL()` 向上取整 |
| 类型转换不清晰 | 应选用 DB2 推荐的类型如 `DOUBLE`, `DECIMAL` 等 |

---

## ✅ 正确写法（DB2 最佳答案）

```sql
SELECT CAST(
    CEIL(
        AVG(CAST(Salary AS DOUBLE)) - 
        AVG(CAST(REPLACE(Salary,'0','') AS DOUBLE))
    ) AS INT
)
FROM EMPLOYEES;
```

### ✅ 解释：

- `REPLACE(Salary, '0', '')`：自动将 `Salary` 转为字符串，去除 0
- `CAST(... AS DOUBLE)`：将字符串结果转为可参与平均的数值
- `AVG(...)`：分别求平均
- `CEIL(...)`：向上取整
- `CAST(... AS INT)`：转换为整数输出

---

## 🧠 DB2 中常用类型转换总结

| 操作意图             | 推荐用法                        | 说明 |
|----------------------|----------------------------------|------|
| 数值转字符串         | `CHAR(Salary)`                  | 会保留空格（要用 TRIM） |
| 字符串去字符         | `REPLACE(CHAR(Salary), '0', '')`| 自动转字符串 |
| 去空格               | `TRIM(...)`                     | 去掉 REPLACE 后可能的空格 |
| 字符串转整数         | `CAST(... AS INT)`              | 标准写法 |
| 字符串转浮点         | `CAST(... AS DOUBLE)`           | 推荐用于 `AVG()` 运算 |
| 字符串转高精度数     | `CAST(... AS DECIMAL(10,2))`    | 如果需要更精确控制位数 |
| 求平均               | `AVG(...)`                      | 适用于 INT/DOUBLE 等 |
| 向上取整             | `CEIL(...)`                     | DB2 中可用 |

---

## 💡 Tips：DB2 转换行为

- `REPLACE(1230, '0', '')` 会自动隐式转为 `VARCHAR`
- 不加 `TRIM()`，可能出现 `SQL0445W` 警告：字符串含空格无法正确转换
- 推荐统一用 `CAST(... AS DOUBLE)` 简化处理（不指定精度时更宽容）

---

## ✅ 总结

使用 DB2 做 SQL 转换题目时：

- 优先用 `DOUBLE` 参与数学运算
- 所有字符串处理后的值记得 `TRIM()`
- 保持类型转换的清晰性，避免隐式错误

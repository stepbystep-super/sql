## 🧠 题目名称：

Weather Observation Station 5

---

## 📌 题目要求：

* 从 `STATION` 表中，选出 **CITY 名字结尾是元音字母 (a, e, i, o, u)** 的城市
* 不要重复值（`DISTINCT`）

---

## 📊 数据表结构：

| 字段名     | 类型           |
| ------- | ------------ |
| ID      | NUMBER       |
| CITY    | VARCHAR2(21) |
| STATE   | VARCHAR2(2)  |
| LAT\_N  | NUMBER       |
| LONG\_W | NUMBER       |

---

## ⚠ 我的错误&考点：

SUBSTR() !!! NOT SUBSTRING()!!!

SUBSTR(CITY,-1,1) NOT GONA WORK!! 不能用负数，这不是python! 用SUBSTR(CITY,LENGTH(CITY),1) 或者像below 用RIGHT()
---

## ✅ 正确答案（适用于 DB2）：

```sql
SELECT DISTINCT CITY
FROM STATION
WHERE RIGHT(LOWER(CITY), 1) IN ('a', 'e', 'i', 'o', 'u');
```

---

## 🔧 知识点小结：

| 函数               | 说明                     |
| ---------------- | ---------------------- |
| `RIGHT(CITY, 1)` | 截取 CITY 最后一个字符（DB2 支持） |
| `LOWER()`        | 转成小写，断言时不分大小写          |
| `DISTINCT`       | 去重，防止城市重复              |

---

## 📝 备注：

* 本题考察 SQL 中字符串操作与过滤逻辑，尤其是末尾字符筛选
* `RIGHT()` , `SUBSTR(..., LENGTH(...))`
* 提交前注意去重和大小写统一处理，避免漏选

---

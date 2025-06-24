## 🧠 题目名称：
Weather Observation Station 4（城市名长度 - 最短 & 最长）

---

## 📌 题目要求：

从 `STATION` 表中：

- 查询名字最短的城市 和 名字最长的城市（按 `CITY` 字段）
- 如果有多个相同长度的城市，取**按字母排序靠前的**
- 输出这两个城市名，以及它们名字的长度

---

## 🧪 数据表结构：

| 字段名 | 类型 |
|--------|------|
| ID     | NUMBER |
| CITY   | VARCHAR2(21) |
| STATE  | VARCHAR2(2) |
| LAT_N  | NUMBER |
| LONG_W | NUMBER |

---

## ❗ 我的卡点：

- ❌ 忘记 SQL 中如何一次性取第一行 & 最后一行 ，不同于Python R, sql 要通过跑俩次来获得。
- ❌有点不太确定length()可以用在这里，还是获得整个数据的长度（行数），整个数据的行数其实是通过COUNT(*) OVER()来获得的。
- ❌ ASC 是default 的， DESC需要特别注明

## 答案
```sql 
(
  SELECT 
    CITY,
    LENGTH(CITY) AS len_name
  FROM STATION
  ORDER BY len_name, CITY
  FETCH FIRST 1 ROWS ONLY
)
UNION
(
  SELECT
    CITY,
    LENGTH(CITY) AS len_name
  FROM STATION
  ORDER BY len_name DESC, CITY
  FETCH FIRST 1 ROWS ONLY
);

```
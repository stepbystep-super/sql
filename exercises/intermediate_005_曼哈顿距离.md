# ❌ SQL 错题笔记：曼哈顿距离 Manhattan Distance（全表极值 + 绝对值）

## 📘 题目描述

给定一张表 `STATION`，其中包含二维平面坐标字段：

- `LAT_N`：纬度
- `LONG_W`：经度

请计算曼哈顿距离（Manhattan Distance），即：

ABS(MAX(LAT_N) - MIN(LAT_N)) + ABS(MAX(LONG_W) - MIN(LONG_W))




要求结果保留 **4 位小数**。

---

## ❌ 我的错误写法（示例）

```sql
SELECT 
  CAST(
    (ABS(MAX(LAT_N) - MIN(LAT_N)) + ABS(MAX(LONG_W) - MIN(LONG_W)))
    AS DECIMAL(16,4)
  )
FROM STATION
GROUP BY LAT_N, LONG_W;  -- ❌ 不该分组！
❗ 错误原因
不该 GROUP BY，因为我们想要的是 全表 极值

GROUP BY LAT_N, LONG_W 会对每一行单独分组，完全不符合题意
```
## ✅ 正确解法
```sql

WITH tb AS (
    SELECT 
        MAX(LAT_N) AS c,
        MIN(LAT_N) AS a,
        MAX(LONG_W) AS d,
        MIN(LONG_W) AS b
    FROM STATION 
)
SELECT 
    CAST(ROUND(ABS(a - c) + ABS(b - d), 4) AS DECIMAL(10, 4)) AS Manhattan_Distance
FROM tb;
```
也可以更加简洁
```sql
SELECT 
  CAST(
    (ABS(MAX(LAT_N) - MIN(LAT_N)) + ABS(MAX(LONG_W) - MIN(LONG_W)))
    AS DECIMAL(16,4)
  )
FROM STATION;
```


## 🧠 知识点总结
技巧	|说明|

ABS()	|计算绝对值|
MAX(), MIN()	|求全表极值|
WITH 子句（CTE）	|提前计算出 max/min，便于主查询引用|
ROUND(..., 4)|	控制保留小数位|     
CAST(... AS DECIMAL(...))|明确指定输出类型及精度|


❌ 不要加 GROUP BY 全表统计时不需要分组


🔚 总结
这题虽然简单，但极易犯下：

GROUP BY 不当使用

ABS 顺序混淆

小数精度未控制好

💡 适合作为SQL 聚合函数 + 格式化输出的入门练习题，建议收藏！
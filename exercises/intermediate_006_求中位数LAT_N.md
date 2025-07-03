# ❌ SQL 错题总结：求中位数 LAT_N（全面复盘）

## 📘 题目描述

从 `STATION` 表中选出字段 `LAT_N` 的**中位数**，并保留 4 位小数。

---

## ❗ 我踩过的坑和错误写法

### ❌ 1. 使用 `IF THEN ELSE` 控制逻辑（非 SQL 标准）

```sql
WHERE IF MOD(...) THEN ... ELSE ...
SQL 中不支持这种类似 C 或 Python 的结构

正确写法应使用 CASE WHEN 或数学表达式控制条件
```
❌ 2. ROW_NUMBER(LAT_N) 语法错误
```sql

ROW_NUMBER(LAT_N) OVER (ORDER BY LAT_N)
ROW_NUMBER() 是不接受参数的窗口函数
```
正确写法应为：

```sql

ROW_NUMBER() OVER (ORDER BY LAT_N)
```
❌ 3. MAX(rk) 用在 WHERE 子句中
```sql

WHERE rk IN (MAX(rk)/2, ...)
```
聚合函数（如 MAX()）不能直接出现在 WHERE 子句中

解决方式：在子查询中提前计算总行数 COUNT(*) OVER ()

❌ 4. 使用 [ ... ] 表达 IN 条件
```sql

WHERE rk IN [1, 2]  
```
-- ❌ Python 写法
SQL 中应使用括号 (...) 而非方括号

```sql

WHERE rk IN (1, 2)  -- ✅
```
❌ 5. COUNT(*) / 2 是整除不是小数！


-- 误以为 499/2 = 249.5，其实是 249！
SQL 中整数除法默认向下取整（floor）

499 / 2 → 249

正确方式是使用 (total_num + 1)/2 和 (total_num + 2)/2 获取中间值

❌ 6. 多选了一行（奇数情况）
```sql

rk IN (total_num / 2, (total_num + 1)/2, (total_num + 2)/2)
```
在奇数情况（如总行数 499）中，total_num / 2 = 249 是多余的

会导致取出第 249 和 250 行，而中位数应只取第 250 行

✅ 最终正确解法（适配奇偶数情况）
```sql

WITH TB AS (
    SELECT 
        LAT_N,
        ROW_NUMBER() OVER (ORDER BY LAT_N) AS rk,
        COUNT(*) OVER () AS total_num
    FROM STATION
)
SELECT 
    CAST(AVG(LAT_N) AS DECIMAL(10, 4)) AS median_lat
FROM TB
WHERE rk IN (
    (total_num + 1)/2,
    (total_num + 2)/2
);
```
🧠 关键知识点总结
知识点	说明
ROW_NUMBER()	编号每一行，方便定位中间行
COUNT(*) OVER ()	获取总行数，不需要 GROUP BY
/ 是整除	整数之间除法自动取整，如 499/2 = 249
CAST(... AS DECIMAL)	控制保留小数位数
中位数通用公式	(n+1)/2, (n+2)/2 可适配奇偶两种情况

✅ 调试小技巧
你曾用以下语句查看中间行：

```sql

SELECT 
  CAST(LAT_N AS DECIMAL(10,4)), 
  total_num/2, (total_num+1)/2, (total_num+2)/2
FROM (
  SELECT LAT_N,
         ROW_NUMBER() OVER(ORDER BY LAT_N) AS rk,
         COUNT(*) OVER() AS total_num
  FROM STATION
) AS TB
WHERE rk IN (total_num/2, (total_num+1)/2, (total_num+2)/2);
```
这是个很好的调试方式 ✅，但在正式答案中应删去 total_num/2 条件。

✅ 口诀记忆
SQL 整除别大意，除法加点 .0；
中位取法加 1 和加 2，全表配合窗口计！

🔚 总结
这是中位数计算在 SQL 中的经典陷阱题。这道题让我清楚理解了：

SQL 与编程语言的差异（如语法结构、除法）

ROW_NUMBER() + COUNT(*) OVER() 的组合应用

奇偶通吃的中位数处理方法

语法细节容错性差，调试必须严谨

💡 这道题值得多次复盘，建议收藏为经典题模板。
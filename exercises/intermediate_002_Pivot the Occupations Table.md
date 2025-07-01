### 题目： 选择
# ✅ SQL 错题笔记：Pivot the Occupations Table（最终理解版）

## 📘 题目描述

将 OCCUPATIONS 表中的数据进行 **转置（Pivot）**，将每个人按职业分类后，**将人名作为值，职业作为列**，每列包含该职业下按字母排序的所有人名。

最终输出格式如下：

| Doctor | Professor | Singer | Actor |
|--------|-----------|--------|-------|
| Alice  | Bob       | Carol  | Diana |
| Jenny  | Maria     | Kristeen | Ketty |
| ...    | ...       | ...    | ...   |

- 如果某一行中某职业没有人，则填 `'NULL'`
- 输出列顺序固定：Doctor, Professor, Singer, Actor
- 每列中人名按字母升序排列

---

## ❌ 我的最初错误写法

```sql
SELECT
  CASE(WHEN Occupation ='Doctor': Name, ELSE: 'NULL') AS Doctor,

FROM OCCUPATIONS
ORDER BY Name;
```
❗ 错误点分析：
错误点	原因
CASE(WHEN ... : ...)	❌ 非标准 SQL 语法，正确写法应为 CASE WHEN ... THEN ... END
没有 ROW_NUMBER()	❌ 无法把不同职业中同一排序位置的人对齐为一行
没有分组	❌ 没有 GROUP BY，也就无法进行列转置
没有聚合函数	❌ 在 GROUP BY 场景中未使用 MAX()、SUM() 等聚合函数

✅ 正确解法（标准版）

```sql

SELECT 
  COALESCE(MAX(CASE WHEN Occupation = 'Doctor' THEN Name END), 'NULL') AS Doctor,
  COALESCE(MAX(CASE WHEN Occupation = 'Professor' THEN Name END), 'NULL') AS Professor,
  COALESCE(MAX(CASE WHEN Occupation = 'Singer' THEN Name END), 'NULL') AS Singer,
  COALESCE(MAX(CASE WHEN Occupation = 'Actor' THEN Name END), 'NULL') AS Actor
FROM (
  SELECT 
    Name,
    Occupation,
    ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) AS rn
  FROM OCCUPATIONS
) AS ranked
GROUP BY rn
ORDER BY rn;
```
✅ 逐步逻辑说明：
```sql
ROW_NUMBER()：

为每个职业内部的人名排序编号（如：医生们按名字编号）

GROUP BY rn：

将各职业中第1个、第2个、第3个名字“拼接在同一行”

CASE WHEN ... THEN Name END：

在该职业的列上返回人名，其它列返回 NULL

MAX(...)：

从每一组中提取该列非 NULL 的人名

COALESCE(..., 'NULL')：

如果该列没有人，则显示为 'NULL'

🧠 重要 SQL 知识点总结
知识点	说明
ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)	给每个职业按人名排序编号
CASE WHEN ... THEN ... END	实现条件列选择
MAX()	聚合每个分组中命中的值，排除 NULL
COALESCE(..., 'NULL')	避免显示 NULL，替换为 'NULL' 字符串
GROUP BY rn	每一组为一行，完成列转置效果
```
❓ 常见疑问解答
Q: 如果一个职业有多个同名的人怎么办？

A: ROW_NUMBER() 会强制编号，即使名字相同，每一位都会在不同的行中显示出来（比如两个 Jenny 会在不同行）。

Q: 为什么一定要用 MAX()？

A: 因为在 GROUP BY rn 中，如果你不聚合非分组列，SQL 会报错。而 MAX() 可以从每组中提取你想要的那一列的值（只会有一个非 NULL 值）。

Q: 为什么不能写 MAX(COALESCE(...))？

A: 因为会把 'NULL' 字符串参与比较，字典序上可能会覆盖合法人名。应该先聚合再补空值 → COALESCE(MAX(...), 'NULL')

✅ 最终记忆模板

```sql

SELECT
  COALESCE(MAX(CASE WHEN Occupation = 'X' THEN Name END), 'NULL') AS X,
  ...
FROM (
  SELECT Name, Occupation,
         ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) AS rn
  FROM OCCUPATIONS
) AS tmp
GROUP BY rn
ORDER BY rn;
```
🔚 心得
这道题学到了 SQL 中：

多行转列的技巧（Pivot 思维）

窗口函数和分组结合的强大

聚合函数和 CASE 的配合使用

NULL 字符串的隐藏陷阱

虽然花了很多时间，但现在已经彻底掌握 ✅
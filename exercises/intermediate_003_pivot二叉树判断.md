# ❌ SQL 错题笔记：判断二叉树节点类型（Node Type in BST）

## 📘 题目描述

给定一张二叉树表 `BST`，包含以下字段：

- `N`: 节点值
- `P`: 节点的父节点值（NULL 表示根节点）

我们需要判断每个节点的类型，输出如下三种：

- `'Root'`: 如果该节点是根节点（P IS NULL）
- `'Leaf'`: 如果该节点不是任何节点的父节点（即它不出现在 P 中）
- `'Inner'`: 其它情况（即它既不是根节点，也有子节点）

要求结果按 `N` 升序排序。

---

## ❌ 我的错误写法

```sql
SELECT DISTINCT
    N,
    CASE 
        WHEN P IS NULL THEN 'Root'
        WHEN N NOT IN (
            SELECT DISTINCT P
            FROM BST
        ) THEN 'Leaf'
        ELSE 'Inner' 
    END
FROM BST
ORDER BY N;
❗ 问题分析
错误点	原因
N NOT IN (SELECT P FROM BST)	❌ P 中含有 NULL，导致 NOT IN (...) 判断失效
所有节点都变成 'Inner'	因为 N NOT IN (...) 条件永远为 UNKNOWN，永远不成立

✅ 正确写法（修复 NOT IN）
sql
Copy
Edit
SELECT
    N,
    CASE 
        WHEN P IS NULL THEN 'Root'
        WHEN N NOT IN (
            SELECT DISTINCT P FROM BST WHERE P IS NOT NULL
        ) THEN 'Leaf'
        ELSE 'Inner'
    END AS NodeType
FROM BST
ORDER BY N;
🔍 关键点讲解
✅ 为什么需要排除 NULL？
SQL 中：NOT IN (x, y, NULL) → 永远返回 UNKNOWN

因此：即使 N 确实不在其中，也不会返回 TRUE

✅ 正确写法：... NOT IN (SELECT P FROM BST WHERE P IS NOT NULL)

🧠 学到的知识点
技术点	说明
P IS NULL	判断是否为根节点
N NOT IN (...)	判断是否为叶子节点，但需要排除 NULL 才生效
三值逻辑	SQL 中 NULL 会导致逻辑表达式变成 UNKNOWN，不是 TRUE 或 FALSE
CASE WHEN 语句	多条件判断输出不同结果
子查询的使用	判断某值是否出现在另一列中

🧪 示例输入：
text
Copy
Edit
N   P
------
1   2
2   3
3   NULL
4   5
5   3
6   5
✅ 正确输出：
text
Copy
Edit
1 Leaf
2 Inner
3 Root
4 Leaf
5 Inner
6 Leaf
✅ 总结口诀
SQL 中 NOT IN (...) 最怕 NULL
遇到判断 “不在某列中”，先 WHERE ... IS NOT NULL

🔚 结语
这是典型的“语法正确，逻辑错误”类型 SQL 题，通过这题我学会了：

如何判断节点在树中的角色（根 / 叶 / 内部）

NOT IN 的坑以及如何修复

CASE + 子查询的组合应用

下次遇到类似结构判断问题时，可以直接套这个逻辑 ✅

yaml
Copy
Edit

---

如你希望将所有错题整理成一个合集导航页（如目录页、索引页等），我也可以帮你生成那部分内容！需要的话告诉我 👍
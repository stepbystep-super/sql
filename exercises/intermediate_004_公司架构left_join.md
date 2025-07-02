# ❌ SQL 错题笔记：统计公司各层级员工数量（多表连接 + 分组聚合）

## 📘 题目描述

给定多个表：`Company`、`Lead_Manager`、`Senior_Manager`、`Manager` 和 `Employee`，要求输出每个公司：

- `company_code`
- `founder`
- 各层级员工（lead / senior / manager / employee）数量（去重计数）

结果要求：
- 每个公司占一行
- 按照 `company_code` 的**字典序**排序（注意：是字符串）

---

## 🧾 表结构说明

```sql
Company(company_code STRING, founder STRING)
Lead_Manager(company_code STRING, lead_manager_code STRING)
Senior_Manager(company_code, lead_manager_code, senior_manager_code)
Manager(company_code, lead_manager_code, senior_manager_code, manager_code)
Employee(company_code, lead_manager_code, senior_manager_code, manager_code, employee_code)
❌ 我的错误写法
sql
Copy
Edit
SELECT
    company_code,
    founder,
    COUNT(DISTINCT lead_manager_code) OVER(PARTITION BY company_code, founder),
    ...
FROM Company
LEFT JOIN ...
❗ 错误原因
使用了 COUNT(...) OVER(PARTITION BY ...) 窗口函数

问题：不会聚合为一行每公司

会保留所有连接组合结果行 → 导致重复计数 + 多行同公司

❗ 还犯了一个拼写错误
sql
Copy
Edit
c.comppany_code  -- ❌ 多打了一个 `p`
✅ 正确写法
使用传统 GROUP BY + COUNT(DISTINCT ...) 统计每家公司各层级人数：

sql
Copy
Edit
SELECT
    c.company_code,
    c.founder,
    COUNT(DISTINCT l.lead_manager_code) AS lead_count,
    COUNT(DISTINCT sm.senior_manager_code) AS senior_count,
    COUNT(DISTINCT m.manager_code) AS manager_count,
    COUNT(DISTINCT em.employee_code) AS employee_count
FROM Company AS c
LEFT JOIN Lead_Manager AS l
    ON c.company_code = l.company_code
LEFT JOIN Senior_Manager AS sm
    ON l.company_code = sm.company_code
    AND l.lead_manager_code = sm.lead_manager_code
LEFT JOIN Manager AS m
    ON sm.company_code = m.company_code
    AND sm.lead_manager_code = m.lead_manager_code
    AND sm.senior_manager_code = m.senior_manager_code
LEFT JOIN Employee AS em
    ON m.company_code = em.company_code
    AND m.lead_manager_code = em.lead_manager_code
    AND m.senior_manager_code = em.senior_manager_code
    AND m.manager_code = em.manager_code
GROUP BY c.company_code, c.founder
ORDER BY c.company_code;
🧠 学到的知识点
技术点	说明
多表 LEFT JOIN	保留公司即使下层没有员工
聚合计数	COUNT(DISTINCT code) 避免重复路径
GROUP BY 聚合	每个 company_code 聚合成一行输出
排序	按字符串 ORDER BY c.company_code 排序（非数字序）
拼写小心	company_code 拼写错误也会导致 SQL 报错

✅ 记忆口诀
“统计用 GROUP BY，不要 OVER；重复用 DISTINCT，层级要连全。”

🧪 示例输出格式（字段）：
text
Copy
Edit
company_code | founder | lead_count | senior_count | manager_count | employee_count
🔚 总结
本题让我真正搞清楚了：

OVER() 与 GROUP BY 的区别

多层级 LEFT JOIN 的使用

COUNT(DISTINCT ...) 的重要性

以及一个字符打错可能导致整条语句失败 😭

这类题本质是结构型 JOIN + 分组统计，适合刷题训练「SQL 建模 + 分组计数能力」。

yaml
Copy
Edit

---

如你后续希望把所有错题汇总成一个导航页（比如：`index.md`），我也可以帮你自动生成结构！继续加油 💪！








Ask ChatGPT

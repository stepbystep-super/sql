🧠 题目名称：
Weather Observation Station 3

📌 题目要求：
选出所有偶数 ID 的记录。

❗ 我的卡点：
忘记了 SQL 中表达“偶数”的写法

不确定 MOD() 函数的语法和参数顺序

✅ 正确写法：
```sql
SELECT *
FROM STATION
WHERE MOD(ID, 2) = 0;
```

💡 知识点小结：
函数	用法	说明
MOD(a, b)	取余数（a 除以 b 的余数）	MOD(ID, 2) = 0 表示 ID 是偶数
MOD(ID, 2) = 1	选出奇数 ID
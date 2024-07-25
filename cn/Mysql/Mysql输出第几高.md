# Mysql输出第几高

## limit实现

~~~java
SELECT DISTINCT salary
FROM employee
ORDER BY salary DESC
LIMIT 1 OFFSET 3;
~~~

## Mysql8 ROW_NUMBER()

~~~java
WITH ranked_salaries AS (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
    FROM (SELECT DISTINCT salary FROM employee) AS unique_salaries
)
SELECT salary
FROM ranked_salaries
WHERE rn = 4;
~~~


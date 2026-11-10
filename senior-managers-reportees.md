# [Задача от Google ро менеджеров](https://datalemur.com/questions/senior-managers-reportees)

Предположим, у нас есть таблица сотрудников Google с указанием их руководителей.

Руководитель (manager) — это сотрудник, у которого есть хотя бы один прямой подчиненный. Старший руководитель (senior manager) — это сотрудник, который руководит хотя бы одним руководителем, но ни один из его прямых подчиненных сам не является старшим руководителем. Напишите SQL-запрос, чтобы найти старших руководителей и их прямых подчиненных.

Выведите имя старшего руководителя и количество его прямых подчиненных. Старший руководитель с наибольшим количеством прямых подчиненных должен идти первым.

Иметь в виду, что у сотрудника может быть 2 старших менеджера.

Вобщем то ли условия задачи плохо сформулированы, то ли решение, которое дано в источнике имеет ошибку. Можно почитать в дискуссии.

Решение, которое мне по крайней мере понятно. Главное на что тут нужно обратить внимание, что сказано, что один сотрудник может отвечать 2м менеджерам. Поэтому на каждом соединении нужно следить, чтобы не проникали дубли.

```sql
WITH managers AS (
    -- 1. Находим всех обычных менеджеров (у кого есть хотя бы один подчинённый)
    SELECT DISTINCT manager_id AS emp_id
    FROM employees
    WHERE manager_id IS NOT NULL
),
senior_candidates AS (
    -- 2. Находим кандидатов в Senior Managers:
    -- руководители, у которых ХОТЯ БЫ ОДИН подчинённый сам является менеджером
    SELECT DISTINCT e.manager_id AS emp_id
    FROM employees e
    JOIN managers m ON e.emp_id = m.emp_id
    WHERE e.manager_id IS NOT NULL
),
valid_senior_managers AS (
    -- 3. Исключаем тех кандидатов, у которых в подчинённых есть ДРУГОЙ кандидат в Senior Manager
    SELECT sc.emp_id
    FROM senior_candidates sc
    WHERE NOT EXISTS (
        SELECT 1
        FROM employees e
        JOIN senior_candidates sub_sc ON e.emp_id = sub_sc.emp_id
        WHERE e.manager_id = sc.emp_id
    )
)
-- 4. Считаем количество прямых подчинённых для каждого честного Senior Manager
SELECT 
    e.manager_name AS senior_manager_name,
    COUNT(DISTINCT e.emp_id) AS direct_reports_count
FROM employees e
JOIN valid_senior_managers vsm ON e.manager_id = vsm.emp_id
GROUP BY e.manager_id, e.manager_name
ORDER BY direct_reports_count DESC;
```

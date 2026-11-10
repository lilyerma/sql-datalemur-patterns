# [Задача про твиты со скользящим средним](https://datalemur.com/questions/rolling-average-tweets)

Учитывая таблицу с данными о твитах за определенный период времени, рассчитайте 3-дневное скользящее среднее количество твитов для каждого пользователя. Выведите ID пользователя, дату твита и скользящее среднее, округленное до 2 знаков после запятой.

В оринальных даных нет пропусков, но чтобы сделать задачу боле интересной, предположим, что они могут появиться.

1) Выясним минимальную и максимальную даты.
2) Сгенерируем все даты между максимумом и минимумом.
3) Сделаем список уникальных пользователей.
4) CROSS JOIN дат и пользовталей, чтобы получить шаблон для заполнения.
5) Делаем LEFT JOIN с исходной таблицей по дате и пользователю, NULL которые появятся в tweet_count обрабатываем COALESCE.
6) Делаем оконную функцию AVG по tweet_count, партиция по пользователю, сортировка по дате, фрейм по 2 ряда до и текущий ряд (теперь нам не страшны пропуски дат).
7) Заворачиваем в ROUND(,2) по условиям задачи.

```sql
WITH limit_dates AS (
    SELECT 
        MIN(tweet_date)::date AS min_date,
        MAX(tweet_date)::date AS max_date
    FROM tweets
),
calendar AS (
    SELECT 
        generate_series(min_date, max_date, INTERVAL '1 day')::date AS tweet_date
    FROM limit_dates
),
users AS (
    SELECT DISTINCT user_id 
    FROM tweets
    WHERE user_id IS NOT NULL
),
user_calendar AS (
    SELECT 
        u.user_id,
        c.tweet_date
    FROM users u
    CROSS JOIN calendar c
),
user_filled_tweets AS (
    SELECT 
        uc.user_id,
        uc.tweet_date,
        COALESCE(t.tweet_count, 0) AS tweet_count
    FROM user_calendar uc
    LEFT JOIN tweets t 
        ON uc.user_id = t.user_id 
       AND uc.tweet_date = t.tweet_date::date
)
SELECT 
    user_id,
    tweet_date,
    ROUND(
        AVG(tweet_count) OVER (
            PARTITION BY user_id 
            ORDER BY tweet_date 
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        )::numeric, 
        2
    ) AS rolling_avg_3d
FROM user_filled_tweets
ORDER BY user_id, tweet_date;

```

---


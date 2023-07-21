Задания + код решения

Первая часть.

* Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».
``` sql
SELECT COUNT(p.id)
FROM stackoverflow.posts p
JOIN stackoverflow.post_types pt ON p.post_type_id = pt.id
WHERE (p.favorites_count >= 100
      OR
      p.score > 300)
      AND
      pt.type = 'Question'
```

* Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлите до целого числа.
``` sql
WITH total_q AS
(
    SELECT COUNT(p.id) count_questions,
       EXTRACT(DAY FROM p.creation_date::date) date_of_question
    FROM stackoverflow.posts p
    JOIN stackoverflow.post_types pt ON p.post_type_id = pt.id
    WHERE pt.type = 'Question'
          AND
          p.creation_date::date BETWEEN '2008-11-01' AND '2008-11-18'
    GROUP BY 2
)

SELECT ROUND(AVG(count_questions))
FROM total_q
```

* Сколько пользователей получили значки сразу в день регистрации? Выведите количество уникальных пользователей.
``` sql
SELECT COUNT(DISTINCT u.id)
FROM stackoverflow.users u
JOIN stackoverflow.badges b ON u.id = b.user_id
WHERE u.creation_date::date = b.creation_date::date
```

* Сколько уникальных постов пользователя с именем `Joel Coehoorn` получили хотя бы один голос?
``` sql
SELECT COUNT(DISTINCT p.id)
FROM stackoverflow.users u
JOIN stackoverflow.posts p ON u.id = p.user_id
JOIN stackoverflow.votes v ON p.id = v.post_id
WHERE u.display_name = 'Joel Coehoorn'
HAVING COUNT(v.id) >= 1
```

* Выгрузите все поля таблицы `vote_types`. Добавьте к таблице поле `rank`, в которое войдут номера записей в обратном порядке. Таблица должна быть отсортирована по полю `id`.
``` sql
SELECT *,
       ROW_NUMBER() OVER(ORDER BY id DESC) rank
FROM stackoverflow.vote_types 
ORDER BY id
```

* Отберите 10 пользователей, которые поставили больше всего голосов типа `Close`. Отобразите таблицу из двух полей: идентификатором пользователя и количеством голосов. Отсортируйте данные сначала по убыванию количества голосов, потом по убыванию значения идентификатора пользователя.
``` sql
SELECT DISTINCT u.id,
       COUNT(v.id) OVER (PARTITION BY u.id) close_count
FROM stackoverflow.users u
JOIN stackoverflow.votes v ON u.id = v.user_id
JOIN stackoverflow.vote_types vt ON v.vote_type_id = vt.id
WHERE vt.name = 'Close'
ORDER BY close_count DESC, u.id DESC
LIMIT 10
```

* Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.
  Отобразите несколько полей:
  * идентификатор пользователя;
  * число значков;
  * место в рейтинге — чем больше значков, тем выше рейтинг.

Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.
Отсортируйте записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.
``` sql
SELECT DISTINCT u.id,
       COUNT(b.id) total_badges, 
       DENSE_RANK() OVER(ORDER BY COUNT(b.id) DESC) rating
FROM stackoverflow.users u
JOIN stackoverflow.badges b ON u.id = b.user_id
WHERE b.creation_date::date BETWEEN '2008-11-15' AND '2008-12-15'
GROUP BY u.id
ORDER BY total_badges DESC, u.id
LIMIT 10
```

* Сколько в среднем очков получает пост каждого пользователя?
  
  Сформируйте таблицу из следующих полей:
  * заголовок поста;
  * идентификатор пользователя;
  * число очков поста;
  * среднее число очков пользователя за пост, округлённое до целого числа.
  
 Не учитывайте посты без заголовка, а также те, что набрали ноль очков.
``` sql
SELECT title,
       user_id,
       score,
       ROUND(AVG(score) OVER (PARTITION BY user_id))
FROM stackoverflow.posts
WHERE title IS NOT NULL
      and
      score <> 0
```

* Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. Посты без заголовков не должны попасть в список.
``` sql
SELECT p.title
FROM stackoverflow.posts p
JOIN stackoverflow.users u ON p.user_id = u.id
JOIN stackoverflow.badges b ON u.id = b.user_id
WHERE p.title IS NOT NULL
GROUP BY p.title
HAVING COUNT(b.id) > 1000
```

* Напишите запрос, который выгрузит данные о пользователях из Канады.
  
  Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
  * пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
  * пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
  * пользователям с числом просмотров меньше 100 — группу 3.
    
  Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу.
``` sql
SELECT u.id,
       u.views,
       CASE
           WHEN u.views >= 350 THEN 1
           WHEN u.views >= 100 AND u.views < 350 THEN 2
           WHEN u.views < 100 THEN 3
       END group_user
FROM stackoverflow.users u
WHERE u.location LIKE '%Canada%'
      AND
      u.views <> 0 
```

* Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров. Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.
``` sql
WITH grouped AS
(
    SELECT u.id,
       u.views,
       CASE
           WHEN u.views >= 350 THEN 1
           WHEN u.views >= 100 AND u.views < 350 THEN 2
           WHEN u.views < 100 THEN 3
       END group_user
    FROM stackoverflow.users u
    WHERE u.location LIKE '%Canada%'
          AND
          u.views <> 0 
),
sorted AS
(
    SELECT *,
           MAX(views) OVER (PARTITION BY group_user) max_views
    FROM grouped
)

SELECT id,
       group_user,
       max_views
FROM sorted
WHERE views = max_views
ORDER BY views DESC, id 
```

* Посчитайте ежедневный прирост новых пользователей в ноябре 2008 года.
  
  Сформируйте таблицу с полями:
  * номер дня;
  * число пользователей, зарегистрированных в этот день;
  * сумму пользователей с накоплением.
``` sql
SELECT DISTINCT EXTRACT(DAY FROM creation_date::date) day_number,
       COUNT(id) OVER (ORDER BY EXTRACT(DAY FROM creation_date)),
       COUNT(id) OVER (PARTITION BY EXTRACT(DAY FROM creation_date))
FROM stackoverflow.users
WHERE creation_date::Date BETWEEN '2008-11-01' AND '2008-11-30'
```

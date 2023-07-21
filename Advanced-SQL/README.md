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

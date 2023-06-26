Задания + код решения.

* Посчитайте, сколько компаний закрылось.
  ```
  SELECT COUNT(id)
  FROM company
  WHERE status = 'closed'
  ```
* Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы `company`. Отсортируйте таблицу по убыванию значений в поле `funding_total`.
```
SELECT funding_total
FROM company
WHERE category_code = 'news'
      AND
      country_code = 'USA'
ORDER BY funding_total DESC
```

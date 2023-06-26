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
* Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.
  ```
  SELECT SUM(price_amount)
  FROM acquisition
  WHERE term_code = 'cash'
        AND
        EXTRACT(YEAR FROM acquired_at) BETWEEN 2011 AND 2013
  ```

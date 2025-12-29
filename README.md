# TOP N 问题
![Alt Text](https://github.com/Aeon0630/sql_-/blob/main/TOP%20N.png)
```ruby
WITH province_user_stats AS(
  SELECT province, user_id, SUM(order_amount) AS total_amount, COUNT(order_id) AS order_count,
         ROW_NUMBER() OVER(PARTITON BY province ORDER BY SUM(order_amount) DESC, COUNT(order_id) DESC) AS rank_in_province
  FROM orders
  WHERE order_date BETWEEN '2025-06-01' AND '2025-06-30'
  GROUP BY province, user_id
)
SELECT province, user_id, rank_in_province, total_amount, order_count
FROM province_user_stats
WHERE rank_in_province <= 10
ORDER BY province, rank_in_province ASC
```


# 行列转换问题
![Alt Text](https://github.com/Aeon0630/sql_-/blob/main/%E8%A1%8C%E5%88%97%E8%BD%AC%E6%8D%A2.png)
```ruby
SELECT
  user_id,
  SUM(CASE WHEN shop_id = 'A' THEN order_amount ELSE 0 END) AS A,
  SUM(CASE WHEN shop_id = 'B' THEN order_amount ELSE 0 END) AS B,
  SUM(CASE WHEN shop_id = 'C' THEN order_amount ELSE 0 END) AS C
FROM orders
WHERE order_date BETWEEN '2025-06-01' AND '2025-06-30' AND shop_id IN ('A','B','C')
GROUP BY user_id
OEDER BY user_id ASC
```


# 存留率问题

user_login_info一张表：user_id  login_time  
```ruby
(SELECT user_id,MIN(date(login_time)) AS first_login_date FROM user_login_info GROUP BY user_id)

```

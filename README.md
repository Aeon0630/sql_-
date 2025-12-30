# TOP N 问题
![Alt Text](https://github.com/Aeon0630/sql_-/blob/main/TOP%20N.png)
```ruby
WITH province_user_stats AS(
  SELECT
    province, user_id, SUM(order_amount) AS total_amount, COUNT(order_id) AS order_count,
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
![Alt Text](https://github.com/Aeon0630/sql_-/blob/main/%E5%AD%98%E7%95%99%E7%8E%87.png)
将题目中的6月注册改为6月首次登录
### 1.整个月的存留率
```ruby
WITH
user_register AS(
  SELECT
    user_id, MIN(login_date) AS register_day
  FROM user_login_events
  WHERE login_date BETWEEN '2025-06-01' AND '2025-06-30'
  GROUP BY user_id
)
retained_users AS(
  SELECT
    ur.user_id, ur.register_day,
    CASE WHEN ule.login_date = DATE_ADD(ur.register_day, INTERVAL 7 DAY) THEN 1 ELSE 0 END AS is_retained
  FROM user_register ur
  LEFT JOIN user_login_events ule
    ON ur.user_id = ule.user_id AND ule.login_date = DATE_ADD(ur.register_day, INTERVAL 7 DAY)
)
SELECT
  SUM(is_retained)/COUNT(DISTINCT user_id) AS retain_rate_7d
FROM retained_users
```
### 2.本月中每一天的存留率
```ruby
WITH
user_register AS(
  SELECT
    user_id, MIN(login_date) AS register_day
  FROM user_login_events
  WHERE login_date BETWEEN '2025-06-01' AND '2025-06-30'
  GROUP BY user_id
)
retained_users AS(
  SELECT
    ur.register_day,
    COUNT(DISTINCT ur.user_id) AS new_users,
    COUNT(DISTINCT ule.user_id) AS retained_users
  FROM user_register ur
  LEFT JOIN user_login_events ule
    ON ur.user_id = ule.user_id AND ule.login_date = DATE_ADD(ur.register_day, INTERVAL 7 DAY)
  GROUP BY ur.register_day
)
SELECT
  register_day, new_users, retained_users,
  ROUND(retained_users/new_users*100, 2) AS retain_rate_7d
FROM retained_users
```


# 连续登录问题
![Alt Text](https://github.com/Aeon0630/sql_-/blob/main/%E8%BF%9E%E7%BB%AD%E7%99%BB%E5%BD%95.png)
```ruby
WITH
user_login_ranked AS(
  SELECT
    user_id, login_date, ROW_NUMBER() OVER(PARTITON user_id OERDER BY login_date) AS rk
  FROM user_login_events
  WHERE login_date BETWEEN '2025-01-01' AND '2025-12-31'
)
user_login_groups AS(
  SELECT
    user_id, login_date, DATE_SUB(login_date, INTERVAL rk DAY) AS group_id
  FROM user_login_ranked
)
continuous_days AS(
  SELECT
    user_id, group_id, MIN(login_date) AS first_date, MAX(login_date) AS last_date, COUNT(*) AS continuous_days
  FROM user_login_groups
  GROUP BY user_id, group_id
)
user_max_days AS(
  SELECT
    user_id, MAX(continuous_days) AS max_days
  FROM continuous_days
  GROUP BY user_id
)
SELECT user_id, max_days
FROM user_max_days
ORDER BY max_days DESC
LIMIT 10
```

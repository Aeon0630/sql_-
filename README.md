# 存留率问题

user_login_info一张表：user_id  login_time  
```ruby
(SELECT user_id,MIN(date(login_time) AS first_login_date FROM user_login_info GROUP BY user_id)

```

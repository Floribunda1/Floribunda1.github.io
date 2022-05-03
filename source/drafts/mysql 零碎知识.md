---
title: MySQL 碎片知识
tags:
  - mysql
categories:
  - 零零碎碎 
---

### 外键

先看看官方 wiki 是怎么解释外键的

{% note primary %}
A **foreign key** is a set of attributes in a table that refers to the [primary key](https://en.wikipedia.org/wiki/Primary_key) of another table. The foreign key links these two tables.
{% endnote %}

假设有客户表

| id   | user_name | user_email | age  |
| ---- | --------- | ---------- | ---- |
| 1    | 甲        | foo@xx.com | 21   |
| 2    | 乙        | bar@xx.com | 37   |
| 3    | 丙        | baz@xx.com | 32   |

一个订单表

| order_id | order_customer | create_time |
| -------- | -------------- | ----------- |
| 100      | 1              | xx          |
| 200      | 1              | xx          |
| 300      | 2              | xx          |


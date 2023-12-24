---
layout: post
title:  "Sql Having & GroupBy!"
date:   2018-08-29 09:23:58 +0545
categories: [SQL, Having - Group by]
tags: [sql, having_group_by]
---

#### Having & Group Clause
* Having is used to restrict the rows affected by the Group By clause as it iis similar to Where clause.
* Having applies to summarized group records, whereas Where applies to individual records.
* Only the groups that meets Having criteria will be returned.
* Having requires that the GROUP BY clause is present so both are in the same query.
* Group By in sql is used to return distinct rows based on table column supplied on group by or group() method.

We have following records in our database, now we want to group record based on i) title ii) id, title combination, and filter further to get count result for articles id greater than 2.
```
<Article id: 1, title: "", created_at: "2019-04-23 05:44:22", updated_at: "2019-04-23 05:44:23">
<Article id: 2, title: "WND", created_at: "2019-04-23 05:46:20", updated_at: "2019-04-23 05:46:20">
<Article id: 3, title: "a", created_at: "2019-04-25 07:35:49", updated_at: "2019-04-25 07:35:50">
<Article id: 4, title: "a", created_at: "2019-04-25 07:35:52", updated_at: "2019-04-25 07:35:52">]
```

```
Article.group(:title).count
SELECT COUNT(*) AS count_all, "articles"."title" AS articles_title FROM "articles" GROUP BY "articles"."title"
=> {""=>1, "WND"=>1, "a"=>2}
```

```
Article.group(:id, :title).count
SELECT COUNT(*) AS count_all, "articles"."id" AS articles_id, "articles"."title" AS articles_title FROM "articles" GROUP BY "articles"."id", "articles"."title"
=> {[1, ""]=>1, [2, "WND"]=>1, [3, "a"]=>1, [4, "a"]=>1}
```

```
Article.group(:title).having("articles.id>2").count
SELECT COUNT(*) AS count_all, "articles"."title" AS articles_title FROM "articles" GROUP BY "articles"."title" HAVING (articles.id>2)
=> {"a"=>2}
```

#### #group_by is used to group the record and #transform_values can be used to count each grouped records. 

```
Institution.first.items.includes(:vendor).group_by{|item| item.vendor.name}.transform_values {|values| values.count}

=> {"Vendor 1"=>1,
 "Vendor 2"=>3,
 "Vendor 3"=>3,
 "Vendor 4"=>1,
 "Vendor 5"=>10
 }
```

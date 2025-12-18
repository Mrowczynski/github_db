# Pull Requests

A high level note : pull requests are issues, but not all issues are pull requests.  When you're looking for PR data, most people are going to assume it's here but I'm just going to warn you...

You will want to hit the issues table as well.

### Bunch of crap

Yeah, there's a bunch of crap here.  Let's filter for what we really care about.  There are two avenues - we are actually figuring out the source/dest (because we're replaying PRs across multiple systems) or we just want metrics.

Let's start with the "metrics"

### Metrics

Most of the time, you're in here because you need metrics on PRs.  Which repo, which user, how long to close, etc.  

```
mysql> select id,repository_id,user_id,created_at,updated_at,merged_at,state from pull_requests limit 3;
+----+---------------+---------+---------------------+---------------------+---------------------+-------+
| id | repository_id | user_id | created_at          | updated_at          | merged_at           | state |
+----+---------------+---------+---------------------+---------------------+---------------------+-------+
|  1 |           352 |      83 | 2011-09-27 13:57:17 | 2011-09-27 14:00:41 | 2011-09-27 14:00:41 | NULL  |
|  2 |           352 |      83 | 2011-09-28 15:41:50 | 2011-09-28 15:48:15 | 2011-09-28 15:48:15 | NULL  |
|  3 |           352 |      83 | 2011-09-28 17:04:09 | 2011-09-30 12:59:00 | NULL                | NULL  |
+----+---------------+---------+---------------------+---------------------+---------------------+-------+
```

The repository_id and user_id correspond to "the other tables" and a PR which is merged has ... merged_at != NULL

So let's make this more human readable.  First we're going to start with the user_id and find my PRs.

```
mysql> select pr.id,pr.repository_id,pr.user_id,u.login,pr.created_at,pr.updated_at,pr.merged_at,pr.state from pull_requests pr, users u where pr.user_id=u.id and u.login="pmrowczynski" limit 3;
+-------+---------------+---------+--------------+---------------------+---------------------+-----------+-------+
| id    | repository_id | user_id | login        | created_at          | updated_at          | merged_at | state |
+-------+---------------+---------+--------------+---------------------+---------------------+-----------+-------+
|  2250 |          1532 |       4 | pmrowczynski | 2012-05-09 21:32:09 | 2012-05-09 21:32:09 | NULL      | NULL  |
|  4426 |          6010 |       4 | pmrowczynski | 2012-08-06 14:12:46 | 2012-08-06 14:26:51 | NULL      | NULL  |
| 35203 |          6028 |       4 | pmrowczynski | 2013-11-13 16:13:21 | 2013-11-13 16:50:58 | NULL      | NULL  |
+-------+---------------+---------+--------------+---------------------+---------------------+-----------+-------+
```

As we can see, these are old PRs and never merged.  So let's figure out which repo

```
mysql> select pr.id,pr.repository_id,org.login,r.name,u.login,pr.created_at,pr.updated_at,pr.merged_at,pr.state from pull_requests pr, users u, users org, repositories r where pr.user_id=u.id and pr.repository_id=r.id and r.owner_id=org.i
d and u.login="pmrowczynski" limit 3;
+-------+---------------+-------------------+-----------------+--------------+---------------------+---------------------+-----------+-------+
| id    | repository_id | login             | name            | login        | created_at          | updated_at          | merged_at | state |
+-------+---------------+-------------------+-----------------+--------------+---------------------+---------------------+-----------+-------+
|  2250 |          1532 | o1                | r1              | pmrowczynski | 2012-05-09 21:32:09 | 2012-05-09 21:32:09 | NULL      | NULL  |
|  4426 |          6010 | o2                | r2              | pmrowczynski | 2012-08-06 14:12:46 | 2012-08-06 14:26:51 | NULL      | NULL  |
| 35203 |          6028 | o3                | r3              | pmrowczynski | 2013-11-13 16:13:21 | 2013-11-13 16:50:58 | NULL      | NULL  |
+-------+---------------+-------------------+-----------------+--------------+---------------------+---------------------+-----------+-------+
```

In the avove, we can see the multiple column names start to bleed together.  But we have data.  We can then clean it up once we are happy with the basic output to make it more presentable.

```
mysql> select pr.id,CONCAT(org.login,"/",r.name) as repo,u.login as user_login ,pr.created_at,pr.updated_at,pr.merged_at,pr.state from pull_requests pr, users u, users org, repositories r where pr.user_id=u.id and pr.repository_id=r.id and r.owner_id=org.id and u.login="pmrowczynski" and merged_at is not NULL limit 3;
+--------+-----------------------+--------------+---------------------+---------------------+---------------------+-------+
| id     | repo                  | user_login   | created_at          | updated_at          | merged_at           | state |
+--------+-----------------------+--------------+---------------------+---------------------+---------------------+-------+
|  35208 | o1/r1                 | pmrowczynski | 2013-11-13 17:00:13 | 2013-11-15 10:04:54 | 2013-11-15 10:04:54 | NULL  |
| 534835 | o2/r2                 | pmrowczynski | 2017-09-11 15:21:39 | 2018-08-21 14:38:14 | 2018-08-21 14:38:14 | NULL  |
| 745546 | o3/r3                 | pmrowczynski | 2018-06-14 15:04:02 | 2018-08-16 14:08:03 | 2018-08-16 14:08:02 | NULL  |
+--------+-----------------------+--------------+---------------------+---------------------+---------------------+-------+
```

And as you can see, state is kind of useless.  How fast is this?  For the entire DB to see how many PRs I created ...

```
| 3352470 | DevGenAI/copilot-doc                   | pmrowczynski | 2024-03-12 19:00:32 | 2024-03-12 19:24:33 | 2024-03-12 19:24:33 | NULL  |
| 4323478 | SCM/sync.iso                           | pmrowczynski | 2025-07-30 03:18:59 | 2025-07-30 05:06:47 | 2025-07-30 05:06:47 | NULL  |
| 4566499 | DevGenAI/copilot-doc                   | pmrowczynski | 2025-12-05 01:19:00 | 2025-12-05 01:19:09 | 2025-12-05 01:19:09 | NULL  |
+---------+----------------------------------------+--------------+---------------------+---------------------+---------------------+-------+
27 rows in set (0.00 sec)
```

Yeah, I suck at times.  But here you can get an idea of how many PRs you have (column 1) or how many PRs are created by individuals such as "pmrowczynski" ... or output it all | cut -f3 | sort -nr kind of thing

### More PR fun

Okay, head over to the issues portion - remember that all PRs are issues, but not all issues are PRs!



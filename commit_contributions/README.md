# Commit Contributions

End-of-Year?  Get tasked with trying to find out which repos are beat on or who makes the most contributions?  This is the table for you.

```
mysql> select * from commit_contributions limit 1;
+----+---------+---------------+--------------+----------------+---------------------+---------------------+
| id | user_id | repository_id | commit_count | committed_date | created_at          | updated_at          |
+----+---------+---------------+--------------+----------------+---------------------+---------------------+
|  1 |    2304 |           466 |            1 | 2012-12-12     | 2013-02-14 10:32:14 | 2013-02-14 10:32:14 |
+----+---------+---------------+--------------+----------------+---------------------+---------------------+
```

As you can see, this is a very short table in terms of columns, but it has legs into two other tables : repositories and users.  If you're not familiar with users nor repositories, go and look in the other locations in this repository

To make use of it (I am so old school I predate join) we use a combination of where & and statements in mysql.  So to make the above make sense, we construct a complex select statement

### Getting the User into the output
Apologies in advance, I'm walking through this for people who may not know about SQL

```
mysql> select c.id,u.login,c.repository_id,c.commit_count,c.committed_date,c.created_at,c.updated_at from commit_contributions c, users u where c.user_id=u.id and u.login="pmrowczynski" limit 1;
+----------+--------------+---------------+--------------+----------------+---------------------+---------------------+
| id       | login        | repository_id | commit_count | committed_date | created_at          | updated_at          |
+----------+--------------+---------------+--------------+----------------+---------------------+---------------------+
| 10828364 | pmrowczynski |         18146 |            1 | 2011-04-28     | 2022-08-24 23:46:56 | 2022-08-24 23:46:56 |
+----------+--------------+---------------+--------------+----------------+---------------------+---------------------+

mysql> select c.id as commit_id,u.id as user_id,u.login as user_login,c.repository_id,c.commit_count,c.committed_date,c.created_at,c.updated_at from commit_contributions c, users u where c.user_id=u.id and u.login="pmrowczynski" limit 1;
+-----------+---------+--------------+---------------+--------------+----------------+---------------------+---------------------+
| commit_id | user_id | user_login   | repository_id | commit_count | committed_date | created_at          | updated_at          |
+-----------+---------+--------------+---------------+--------------+----------------+---------------------+---------------------+
|  10828364 |       4 | pmrowczynski |         18146 |            1 | 2011-04-28     | 2022-08-24 23:46:56 | 2022-08-24 23:46:56 |
+-----------+---------+--------------+---------------+--------------+----------------+---------------------+---------------------+
```

As we can see in the above, the user_id is now changed to something we would recognize.

### Getting the Repository information

Here we need to figure out what is the repository name.  Complex SQL but you can simply keep adding compound AND statements until it looks kind of right

```
mysql> select c.id as commit_id,u.id as user_id,u.login as user_login,c.repository_id,r.name,org.login,c.commit_count,c.committed_date,c.created_at,c.updated_at from commit_contributions c, users u, repositories r, users org where c.user_id=u.id and u.login="pmrowczynski" and c.repository_id=r.id and r.owner_id=org.id limit 3;
+-----------+---------+--------------+---------------+----------------+---------------+--------------+----------------+---------------------+---------------------+
| commit_id | user_id | user_login   | repository_id | name           | login         | commit_count | committed_date | created_at          | updated_at          |
+-----------+---------+--------------+---------------+----------------+---------------+--------------+----------------+---------------------+---------------------+
|  10828364 |       4 | pmrowczynski |         18146 | org_1          | repo_1        |            1 | 2011-04-28     | 2022-08-24 23:46:56 | 2022-08-24 23:46:56 |
|  14386482 |       4 | pmrowczynski |         30623 | org_1          | repo_2        |            1 | 2011-04-28     | 2024-03-14 03:26:51 | 2024-03-14 03:26:51 |
|   6865257 |       4 | pmrowczynski |         36737 | org_2          | repo_3        |            1 | 2011-04-28     | 2019-11-25 22:38:15 | 2019-11-25 22:38:15 |
+-----------+---------+--------------+---------------+----------------+---------------+--------------+----------------+---------------------+---------------------+```
```

Then clean up the formatting

```
mysql> select c.id as commit_id,u.id as user_id,u.login as user_login,CONCAT(org.login,"/",r.name) as repo,c.commit_count,c.committed_date,c.created_at,c.updated_at from commit_contributions c, users u, repositories r, users org where c.user_id=u.id and u.login="pmrowczynski" and c.repository_id=r.id and r.owner_id=org.id and c.created_at > "2025-12-05" limit 3;
+-----------+---------+--------------+--------------------------+--------------+----------------+---------------------+---------------------+
| commit_id | user_id | user_login   | repo                     | commit_count | committed_date | created_at          | updated_at          |
+-----------+---------+--------------+--------------------------+--------------+----------------+---------------------+---------------------+
|  19423472 |       4 | pmrowczynski | DevGenAI/copilot-doc     |            5 | 2025-12-04     | 2025-12-05 00:46:48 | 2025-12-05 01:27:15 |
|  19426789 |       4 | pmrowczynski | SCM/github.users         |            1 | 2025-12-05     | 2025-12-05 10:00:24 | 2025-12-05 10:00:24 |
|  19495314 |       4 | pmrowczynski | pmrowczynski/commit.data |            1 | 2025-12-05     | 2025-12-17 04:11:36 | 2025-12-17 04:11:36 |
+-----------+---------+--------------+--------------------------+--------------+----------------+---------------------+---------------------+
```

#### Who gets credit?

Code commit contributions is based upon the email address the user has saved in the settings - if it's not in there, they're not getting credit!  You can see this in the green timeline graph, or when you look at any code commit that is linked to their account.  

If you can not click on to the code commit and see the link to their account - they're not getting credit.  This is why there are a few fields

#### What is the difference between committed_date, created_at, etc?

In the DB, the committed_date corresponds to the code commit.  The created_at is when GitHub was able to properly index the code commit and associate it with the proper user_id via email address. 

#### Why do it this way?

If you're gathering metrics and have this ... far easier than trying to hit an API for multiple users plus you won't necessarily see data for the internal/private repos.

```
|  19504000 |       4 | pmrowczynski | SCM/github.users                          |            2 | 2025-12-18     | 2025-12-18 10:01:50 | 2025-12-18 10:01:52 |
|  19504249 |       4 | pmrowczynski | pmrowczynski/commit.data                  |            1 | 2025-12-18     | 2025-12-18 12:00:34 | 2025-12-18 12:00:34 |
|  19504246 |       4 | pmrowczynski | pmrowczynski/gpg.data                     |            2 | 2025-12-18     | 2025-12-18 12:00:19 | 2025-12-18 12:00:25 |
+-----------+---------+--------------+-------------------------------------------+--------------+----------------+---------------------+---------------------+
3106 rows in set (0.02 sec)
```



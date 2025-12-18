# Users Table

There are really too many columns here to describe them all in detail, but there are a few we focus on for metrics, user data and usage in other tables.  

| id | login | type | created_at | suspended_at |
| -- | ----- | ---- | ---------- | ------------ |
| 4 | pmrowczynski | User | 2011-09-01 | NULL |
| 5 | SCM | Organization | 2011-09-01 | NULL

In the above, we can see that the DB corresponds here and everywhere else as the match.  The login is a plaintext name of the user, so no emails.  The type is either going to be User or Organization to help differentiate between /pmrowczynski and /SCM and then we have the created_at which is when the user or org was created, finally suspended_at is only for users who have been disabled

If you're generating a report of organizations

```select id,login from users where type = "Organization;```

If you're generating a report of users, most of the time you don't care how long ago the user was disabled out of your system so...

```select id,login from users where type = "User" and suspended_at is NULL;```

Another example

```
mysql> select id,login,type,created_at,updated_at,suspended_at from users where login="pmrowczynski";
+----+--------------+------+---------------------+---------------------+--------------+
| id | login        | type | created_at          | updated_at          | suspended_at |
+----+--------------+------+---------------------+---------------------+--------------+
|  4 | pmrowczynski | User | 2011-09-17 10:51:35 | 2025-11-21 01:16:20 | NULL         |
+----+--------------+------+---------------------+---------------------+--------------+

mysql> select id,login,type,created_at,updated_at,suspended_at from users where login="SCM";
+----+-------+--------------+---------------------+---------------------+--------------+
| id | login | type         | created_at          | updated_at          | suspended_at |
+----+-------+--------------+---------------------+---------------------+--------------+
| 62 | SCM   | Organization | 2011-09-20 18:57:52 | 2021-09-21 23:23:25 | NULL         |
+----+-------+--------------+---------------------+---------------------+--------------+
```

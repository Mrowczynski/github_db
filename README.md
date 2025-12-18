# GitHub DB

A repository to hold some information pertaining to the on-premise instance of GitHub and the DB implementation.  

### Why?

I have forgotten far too much about GitHub, deployments and DB ever since deploying it back in 2011 (GitHub Firewall)

### Access

```$ mysql -u root github_enterprise```

Newer versions of GitHub may require running this as sudo

```sudo mysql -u root github_enterprise```

And if you just want to get some information real quick

```
$ mysql -u root github_enterprise -e 'select id,login,type,created_at,updated_at,suspended_at from users where login="pmrowczynski"'
+----+--------------+------+---------------------+---------------------+--------------+
| id | login        | type | created_at          | updated_at          | suspended_at |
+----+--------------+------+---------------------+---------------------+--------------+
|  4 | pmrowczynski | User | 2011-09-17 10:51:35 | 2025-11-21 01:16:20 | NULL         |
+----+--------------+------+---------------------+---------------------+--------------+
```

And if you do not care for (because ... whatevs) the header or deliminators, just use -sN

```
$ mysql -sN -u root github_enterprise -e 'select id,login,type,created_at,updated_at,suspended_at from users where login="pmrowczynski"'
4	pmrowczynski	User	2011-09-17 10:51:35	2025-11-21 01:16:20	NULL
```

# GitHub DB

A repository to hold some information pertaining to the on-premise instance of GitHub and the DB implementation.  

This is 100% not supported by GitHub, you're not supposed to be in the DB, you're not supposed to use mysql but instead of the logged ghe-console kind of command.  And even then, only with the express approval of a Microsoft GitHub support engineer to give you the commands and make sure you don't don't break things.

Do NOT run an UPDATE command.  Seriously ... just don't do it.  You're probably in a ghe-repl-status environment and though we'd love to say all commands are replicated ... we know that things happen. 

Not sure, want help?  Drop me a line first.  I'd rather not a late night call when something bad happens.  

-- P

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

### Items to cover

These are the areas I want to cover - based upon how frequently we pull, utilize and consider the importance of data

* Users
* Repositories
* Code Commits
* Pull Requests
* Issues

Pausing here... back to the audit stuff...

* OAUTH tokens
* Abilities in orgs
* GPG data

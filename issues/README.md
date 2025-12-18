# Issues

Most of the time you're only going to be here because you want metrics on PRs but that information isn't good enough in the pull_requests table.  

Here are the key columns you're going to want to start with:

```
mysql> select id,repository_id,user_id,issue_comments_count,number,title,state,created_at,closed_at,pull_request_id,assignee_id,has_pull_request from issues limit 1;
+----+---------------+---------+----------------------+--------+----------------+-------+---------------------+-----------+-----------------+-------------+------------------+
| id | repository_id | user_id | issue_comments_count | number | title          | state | created_at          | closed_at | pull_request_id | assignee_id | has_pull_request |
+----+---------------+---------+----------------------+--------+----------------+-------+---------------------+-----------+-----------------+-------------+------------------+
|  1 |           241 |       4 |                    3 |      1 | 0x1234566789AB | open  | 2011-09-19 15:11:57 | NULL      |            NULL |        NULL |                0 |
+----+---------------+---------+----------------------+--------+----------------+-------+---------------------+-----------+-----------------+-------------+------------------+
```

A few notes on the above.  
* repository_id corresponds to the id of the repositories table (the repository)
* user_id corresponds to the id of the user table, the person who made the issue (or more likely, the PR)
* issue_comments_count is what you are looking for if you do NOT want to see rubber-stamped PRs and a back-and-forth
* created_at ... closed_at actually helps you figure out the duration of a PR
* closed_at if the PR, err, issue is closed
* pull_request_id ... this is the link to the PR table.
* title : this is a killer ... non-UTF characters exist!

With that in mind, we may want to find more information such as who does the most PRs, assigned (NOT approves!) PRs, and a if there is a title, etc.  Let's get one-half real:

```
mysql> select i.id,i.repository_id,u.login as creator_login,i.issue_comments_count,i.number as issue_number,i.title,i.state,i.created_at,i.closed_at,pr.id as pr_id,a.login as assignee_login,i.has_pull_request from issues i, users u, users
 a, pull_
+------+---------------+---------------+----------------------+--------------+----------------------------------------------------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| id   | repository_id | creator_login | issue_comments_count | issue_number | title                                                                      | state  | created_at          | closed_at           | pr_id | assignee_login | has_pull_request |
+------+---------------+---------------+----------------------+--------------+----------------------------------------------------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| 1637 |          2089 | u1            |                    0 |           34 | 0x506C65617365206D6572676520746869732070756C6C2072657175657374                                                                      | closed | 2012-03-26 21:21:34 | 2012-03-28 03:40:17 |  1311 | u4             |                1 |
| 1829 |          1275 | u2            |                    0 |            1 | 0xHex                                                                      | closed | 2012-04-05 04:35:25 | 2012-04-28 01:05:08 |  1482 | u5             |                1 |
| 1906 |          2089 | u3            |                    0 |           58 | 0xHex                                                                      | closed | 2012-04-10 23:31:05 | 2012-04-10 23:33:34 |  1555 | u6             |                1 |
+------+---------------+---------------+----------------------+--------------+----------------------------------------------------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
```

So with this we can see how many PRs there may be created by a person, maybe the state and repository information.  Let's pull a little bit more to generate a better report.

```
mysql> select i.id, CONCAT(org.login,"/",r.name) as repo,u.login as creator_login,i.issue_comments_count,i.number as issue_number,i.title,i.state,i.created_at,i.closed_at,pr.id as pr_id,a.login as assignee_login,i.has_pull_request from issues i, users u, users a, pull_requests pr, repositories r, users org where i.user_id=u.id and i.assignee_id = a.id and i.pull_request_id=pr.id and i.has_pull_request=1 and i.repository_id=r.id and r.owner_id=org.id limit 3;
+------+--------------------+---------------+----------------------+--------------+----------------------------------------------------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| id   | repo               | creator_login | issue_comments_count | issue_number | title                                                                      | state  | created_at          | closed_at           | pr_id | assignee_login | has_pull_request |
+------+--------------------+---------------+----------------------+--------------+----------------------------------------------------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| 1637 | o1/r1              | u1            |                    0 |           34 | 0x506C65617365206D6572676520746869732070756C6C2072657175657374             | closed | 2012-03-26 21:21:34 | 2012-03-28 03:40:17 |  1311 | u4             |                1 |
| 1829 | o2/r2              | u2            |                    0 |            1 | 0xHex                                                                      | closed | 2012-04-05 04:35:25 | 2012-04-28 01:05:08 |  1482 | u5             |                1 |
| 1906 | o3/r3              | u3            |                    0 |           58 | 0xHex                                                                      | closed | 2012-04-10 23:31:05 | 2012-04-10 23:33:34 |  1555 | u6             |                1 |
+------+--------------------+---------------+----------------------+--------------+----------------------------------------------------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
```

Finally, we have to CONVERT and CAST the title to make it human readable : CONVERT (i.title, char) with the cleaned up column header for parsing or human readability

```
mysql> select i.id, CONCAT(org.login,"/",r.name) as repo,u.login as creator_login,i.issue_comments_count,i.number as issue_number,CONVERT(i.title,char),i.state,i.created_at,i.closed_at,pr.id as pr_id,a.login as assignee_login,i.has_pull_request from issues i, users u, users a, pull_requests pr, repositories r, users org where i.user_id=u.id and i.assignee_id = a.id and i.pull_request_id=pr.id and i.has_pull_request=1 and i.repository_id=r.id and r.owner_id=org.id limit 1;
+------+--------------+---------------+----------------------+--------------+--------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| id   | repo         | creator_login | issue_comments_count | issue_number | CONVERT(i.title,char)          | state  | created_at          | closed_at           | pr_id | assignee_login | has_pull_request |
+------+--------------+---------------+----------------------+--------------+--------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| 1637 | o1/r1        | u1            |                    0 |           34 | Please merge this pull request | closed | 2012-03-26 21:21:34 | 2012-03-28 03:40:17 |  1311 | u4             |                1 |
+------+--------------+---------------+----------------------+--------------+--------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+

mysql> select i.id, CONCAT(org.login,"/",r.name) as repo,u.login as creator_login,i.issue_comments_count,i.number as issue_number,CONVERT(i.title,char) as issue_title,i.state,i.created_at,i.closed_at,pr.id as pr_id,a.login as assignee_login,i.has_pull_
org.id limit 1;
+------+--------------+---------------+----------------------+--------------+--------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| id   | repo         | creator_login | issue_comments_count | issue_number | issue_title                    | state  | created_at          | closed_at           | pr_id | assignee_login | has_pull_request |
+------+--------------+---------------+----------------------+--------------+--------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
| 1637 | o1/r1        | u1            |                    0 |           34 | Please merge this pull request | closed | 2012-03-26 21:21:34 | 2012-03-28 03:40:17 |  1311 | u4             |                1 |
+------+--------------+---------------+----------------------+--------------+--------------------------------+--------+---------------------+---------------------+-------+----------------+------------------+
```

### NOTE : PR is an Issue!

If you were to browse to the https://{github}/o1/r1/issues/34 in the above, you'd automatically be redirected to https://{github}/o1/r1/pull/34 - note that this pull request URL isn't the same as the API, but it let's you know that issue => PR is indeed a fact even though in the above 1311 is an entirely different table (by reference) in the database.

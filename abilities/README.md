# Abitilies

This table controls who has access to what.  Think Teams, Direct Collaborators for repos and even Organization Owners.


| id | actor_id | actor_type | action | subject_id | subject_type | priority | created_at          | updated_at          | parent_id |
|----|----------|------------|--------|------------|--------------|----------|---------------------|---------------------|-----------|
|  1 |      421 | User       |      1 |      36303 | Repository   |        1 | 2014-08-09 12:56:52 | 2014-08-09 12:56:52 |         0 |
|  2 |     1378 | Team       |      2 |      51603 | Repository   |        1 | 2014-08-09 16:15:30 | 2014-08-09 16:15:30 |         0 |

Of importance here is the subject_type and subject_id.  

### subject_type

The subject_type is going to be one of a couple of options: Repository, Organizations or Team


* if it is an Organization, then the subject_id is going to be the id from the users table
* if it is a Repository, then the subject_id is going to be the ID from the repositories table

### Action

This pertains to the permissions level.  
For example, at the repository level, 0 may be read, 1 maybe write and 2 may be admin.  

### Actor_Type

Most of the time I spent using "User" as the key to figure out who had access to individual organizations, or just being a Team member.  

### Why would I use this?

If you have several (XXX ... let's say thousands) of organizations, you may not want to wait for something bad to happen with a webhook ("Oh look, the admins here just removed everyone") and even if you're using the audit log, there is actually a 30-90 second delay before things do show up.  

The DB is faster, and you can query a LOT of changes faster - notice the "updated_at" column yet?

Today I have to "make sure that if this user account gets removed from the organization PUT IT BACK" and then the "if the webhook changes, PUT IT BACK"  I have a good rate limiting but the problem is that we may not be able to rely upon the API call to detect the change.  Yaya, I am going to be checking the github auth logs too - but sometimes we gotta be faster than that ("We have 500 organizations, you better be sure that ABC account is still owner when I get back!")

```
mysql> select a.id,a.actor_id,a.actor_type,a.action,a.subject_id,a.subject_type, a.created_at, a.updated_at from abilities a where a.actor_id=19402 and a.actor_type="User" limit 3;
+---------+----------+------------+--------+------------+--------------+---------------------+---------------------+
| id      | actor_id | actor_type | action | subject_id | subject_type | created_at          | updated_at          |
+---------+----------+------------+--------+------------+--------------+---------------------+---------------------+
| 2742018 |    19402 | User       |      0 |         39 | Organization | 2023-03-24 23:33:30 | 2024-01-09 19:45:11 |
| 2741830 |    19402 | User       |      2 |         96 | Organization | 2023-03-24 20:47:48 | 2023-03-24 20:47:48 |
| 3412903 |    19402 | User       |      2 |        221 | Organization | 2025-08-02 09:24:15 | 2025-08-02 09:24:15 |
+---------+----------+------------+--------+------------+--------------+---------------------+---------------------+
<SNIP>
| 3544514 |    19402 | User       |      0 |      10528 | Team            | 2025-12-08 15:24:57 | 2025-12-08 15:24:57 |
| 3546189 |    19402 | User       |      0 |      10532 | Team            | 2025-12-09 16:36:23 | 2025-12-09 16:36:23 |
+---------+----------+------------+--------+------------+-----------------+---------------------+---------------------+
20464 rows in set (0.10 sec)
```

As obvious in the above, there are quite a few references with that little SQL command and I can focus on the areas (e.g. Organization) where it matters.

In this case, I may zero in on one organization (42846) and user (19402) to track if/when things changea

```
mysql> select a.id,a.actor_id,a.actor_type,a.action,a.subject_id,a.subject_type, a.created_at, a.updated_at from abilities a where a.actor_id=19402 and subject_id=42846 and subject_type="Organization" and a.actor_type="User" limit 3;
+---------+----------+------------+--------+------------+--------------+---------------------+---------------------+
| id      | actor_id | actor_type | action | subject_id | subject_type | created_at          | updated_at          |
+---------+----------+------------+--------+------------+--------------+---------------------+---------------------+
| 3452120 |    19402 | User       |      2 |      42846 | Organization | 2025-09-12 00:25:08 | 2025-09-12 00:25:08 |
+---------+----------+------------+--------+------------+--------------+---------------------+---------------------+
```

### Readability : Organizations

For more in-depth readability such as "what organizations is this account an 'owner' of (action == 2) I'd construct a more complext SQL command.  

```
mysql> select a.id,u.login,a.action,org.login, a.created_at, a.updated_at from abilities a, users u, users org where subject_type="Organization" and a.actor_type="User" and a.actor_id=u.id and subject_id=org.id and u.login="pmrowczynski"
limit 3;
+---------+--------------+--------+----------------+---------------------+---------------------+
| id      | login        | action | login          | created_at          | updated_at          |
+---------+--------------+--------+----------------+---------------------+---------------------+
| 3358317 | pmrowczynski |      2 | o1             | 2025-05-13 21:07:21 | 2025-05-13 21:07:21 |
| 3321330 | pmrowczynski |      2 | SCM            | 2025-03-10 17:25:42 | 2025-03-10 17:25:42 |
| 3490605 | pmrowczynski |      2 | o2             | 2025-10-25 18:53:06 | 2025-10-25 18:53:06 |
+---------+--------------+--------+----------------+---------------------+---------------------+
```

In a large organization, you can generate a report with "Who has access to what?  When did it change?" as well.  
Tis means I'm not using my rate-limits and can structure my SQL query as well to avoid hundreds of thousands of prior results by using the API or GraphQL.  So if I wanted to see changes in the past month for my account

```
mysql> select a.id,u.login as user, a.action, org.login as org, a.created_at, a.updated_at from abilities a, users u, users org where subject_type="Organization" and a.actor_type="User" and a.actor_id=u.id and subject_id=org.id and u.login="pmrowczynski" and a.updated_at > "2025-12-01";
+---------+--------------+--------+---------------+---------------------+---------------------+
| id      | user         | action | org           | created_at          | updated_at          |
+---------+--------------+--------+---------------+---------------------+---------------------+
| 3489998 | pmrowczynski |      0 | o3            | 2025-10-25 13:45:27 | 2025-12-17 16:16:53 |
+---------+--------------+--------+---------------+---------------------+---------------------+
```

Obviously eliminating the u.login="pmrowczynski" gives you ALL of the changes that have been added.  Now this does NOT show you which ones have been removed and you can't determine the prior state ("was an owner, but is not any more") but you can hit the audit-logs for that once you figure out which organization.

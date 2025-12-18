# Repositories table

Usually the next thing people want is repository information.  Either count, things to delete or what changes a lot.

As always, there are key columns you care about, not everything.  

In no real particular order except for how frequently I use each column

| id | name | owner_id | parent_id | created_at | updated_at | deleted_at | disabled_at | archived_at | public | 
| -- | ---- | -------- | --------- | ---------- | ---------- | ---------- | ----------- | ----------- | ------ | 
|  21 | scm       |        6 |      NULL | 2011-09-17 11:14:14 | 2014-05-22 19:27:04 | NULL       | 2022-06-21 23:33:25 | NULL        |      0 |

The select statement for the above would look like this:

```
select id,name,owner_id,parent_id,created_at,updated_at,deleted_at,disabled_at,archived_at,public from repositories limit 1;
+----+------+----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
| id | name | owner_id | parent_id | created_at          | updated_at          | deleted_at | disabled_at         | archived_at | public |
+----+------+----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
| 21 | scm  |        6 |      NULL | 2011-09-17 11:14:14 | 2014-05-22 19:27:04 | NULL       | 2022-06-21 23:33:25 | NULL        |      0 |
+----+------+----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
```

Now people call them joins, I just use multiple where & and statements.  In the above, we know the repo name, but we also need to know the organization name.  This owner_id comes from the users table.  

```
mysql> select r.id,r.name,r.owner_id,r.parent_id,r.created_at,r.updated_at,r.deleted_at,r.disabled_at,r.archived_at,public from repositories r, users org where r.owner_id = org.id and r.name="SCM" and r.owner_id=6;
+----+------+----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
| id | name | owner_id | parent_id | created_at          | updated_at          | deleted_at | disabled_at         | archived_at | public |
+----+------+----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
| 21 | scm  |        6 |      NULL | 2011-09-17 11:14:14 | 2014-05-22 19:27:04 | NULL       | 2022-06-21 23:33:25 | NULL        |      0 |
+----+------+----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
```

IMO, TMI and more difficult to understand.  In my real world, I start concat things and want to eliminate duplicates***

```
mysql> select r.id,CONCAT (org.login,"/",r.name) as repo ,r.parent_id,r.created_at,r.updated_at,r.deleted_at,r.disabled_at,r.archived_at,public from repositories r, users org where r.owner_id = org.id and r.name="SCM" and r.owner_id=4;
+-------+------------------+-----------+---------------------+---------------------+------------+-------------+-------------+--------+
| id    | repo             | parent_id | created_at          | updated_at          | deleted_at | disabled_at | archived_at | public |
+-------+------------------+-----------+---------------------+---------------------+------------+-------------+-------------+--------+
| 42126 | pmrowczynski/scm |      NULL | 2014-05-08 16:02:39 | 2020-04-16 17:30:14 | NULL       | NULL        | NULL        |      1 |
+-------+------------------+-----------+---------------------+---------------------+------------+-------------+-------------+--------+
```

#### Duplicates
Duplicates can occur.  This is why we may want to include the deleted_at reference or exclude because of it.  For example, if you delete a repository with one name, you are free to re-fork and create another repository with the exact same name.  
This matter as you get more in-depth for reporting.  Otherwise, there's a good chance you can wind up with multiple row results when trying to parse things out - then wondering why the ID doesn't give you correct results by the API (because that name is deleted, renamed, etc.)

### Is it a fork?
Over the years, GitHub changed a few things.  But basically 
* Source ID is the root node of the network and this Source ID either corresponds to that node, is zero meaning this is the root node or is equal to the ID of this node (again, it's a root node)
* Parent ID is the "forked from" ID in the repositories table

- If there is no parent_id, this is a root node
- If there is no source_id, this is a root node
- If there is a source_id AND the id is the source_id, this is a root node

```
mysql> select r.id,CONCAT (org.login,"/",r.name) as repo, org.type, r.source_id,r.parent_id,r.created_at,r.updated_at,r.deleted_at,r.disabled_at,r.archived_at,public from repositories r, users org where r.owner_id = org.id and r.deleted_a
t is NULL
+-----+-----------------------------------+------+-----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
| id  | repo                              | type | source_id | parent_id | created_at          | updated_at          | deleted_at | disabled_at         | archived_at | public |
+-----+-----------------------------------+------+-----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
|  21 | is_user/scm                       | User |        21 |      NULL | 2011-09-17 11:14:14 | 2014-05-22 19:27:04 | NULL       | 2022-06-21 23:33:25 | NULL        |      0 |
| 190 | is_user/lbms-rest                 | User |       190 |      NULL | 2011-09-17 11:35:50 | 2014-06-11 16:23:01 | NULL       | 2022-06-21 23:33:25 | NULL        |      0 |
| 231 | is_user/ken_repo                  | User |       231 |      NULL | 2011-09-18 18:21:56 | 2011-09-18 18:27:55 | NULL       | 2022-05-09 21:54:34 | NULL        |      1 |
+-----+-----------------------------------+------+-----------+-----------+---------------------+---------------------+------------+---------------------+-------------+--------+
```

#### Visibility
column name "public" controls it.  There are pretty much only three fields: public (1), internal and private (0).  

TBD:  Where is 'internal' referenced I totally forgot where "internal" is flagged, hence why I am writing this.



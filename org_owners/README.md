# Organization Owners

You may want to review the abilities portion because we'll be using this table, but on the off chance you just want an answer

TL:DR : You want to find out who the organization owners are in a large GHES instance and don't want to sit the 15-20 minutes for the export.  

And if you have thousands of organizations, you just want something to auto-gen or to fix right away.  In addition, you may have people who constantly ask "Can you add me to the organization, I'm new here" and no one is making their information public.

### Background

The abilities table is where it is at.  You can make one query, save or parse the output.  In our example, we to determine which organizations a service account belongs to, and if they're not a member of a specific organization, make that change

* Abilities is very light weight and only has a few corresponding (aka relevant) pieces of information.
* Columns from abilities are going to be the subject_type, actor and action

```
# Assuming you know the dbid of your account from the users table
select * from abilities where actor_id={dbid} and subject_id={org_dbid} and subject_type="Organization" and action=2
```

The above is the core of the operation using one table.  But if you want to extend it a bit, maybe you want to find the list of the organizations for {user}

```
# Please note that I'm using limit here ...
mysql> select u.login,org.login,a.action from abilities a, users u, users org where a.subject_type="Organization" and u.login="{user}" and a.actor_type="User" and a.actor_id=u.id and a.subject_id=org.id and a.action=2 limit 5;
+---------+------------+--------+
| login   | login      | action |
+---------+------------+--------+
| user    | org-1      |      2 |
| user    | org-2      |      2 |
| user    | org-3      |      2 |
| user    | org-4      |      2 |
| user    | org-5      |      2 |
+---------+------------+--------+
5 rows in set (0.00 sec)
```

So if you have a few organizations, this method is much faster than anything out there!  0.01 seconds for 2900+ organizations - no pagination, no parsing, no graphql

Then you can simple output it to a file, "cat | grep" and look for the entries you want (e.g. the organizations to find the owners, or the user to see what they own if appropriate)

```
mysql> select u.login,org.login,a.action from abilities a, users u, users org where a.subject_type="Organization" and u.login="pmrowczynski" and a.actor_type="User" and a.actor_id=u.id and a.subject_id=org.id and a.action=2;
...
| pmrowczynski | Bling-1                                  |      2 |
| pmrowczynski | bling-2                                  |      2 |
+--------------+------------------------------------------+--------+
2931 rows in set (0.01 sec)
```

### Adding owner
Then if you're lazy and find a miss, you can use the ghe utility.  Mind you, this is VERY slow but doesn't require anything but GHES access (and you're hopefully not using it to add/remove/add/remove ... use the API for that)

```
ghe-org-admin-promote -y -u {user} -o {org}
```

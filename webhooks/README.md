# WebHooks

This is a bit more convoluted.  For example in an on-premise instance you want to find out which repositories have hooks and maybe specific URLs.  In some cases, it's to get a quick dashboard for targetted end-points (compliance) or if you want to find out which repositories need to have their webhooks updated due to an end-point EOL

### Key Tables

There are two key tables in the DB : hook_config_attributes and hook_event_subscriptions.  These two tables control the deployment of the hooks, the endpoint and the envents;
* Organization & Repository : this is the human readable format which you can adjust for your API needs
* URL : You will have a targetted URL for your hooks to submit the payload data against, this is it
* Event : The event is items which you'd select when creation of the webhook like push, pull_request, branch_protection_rule and issue_comment

### Background

"Hey, we need to know ASAP when our webhooks are modified - and put the hook back.  The problem we also see is 'they' may create a new repository and we need to make sure that they also get a new webhook.  When you're done, give us a dashboard too.  And if our service account doesn't have permissions, grant them."

Yes, lots of way to address this situation but if you want the fastest way without DDoS'ng yourself, this is an option.  And who doesn't like reviewing the DB.

### Implementation : DB Call

This is a simple compound SQL statement (old school, I predate join so YMMV)

```
  # mysql> select CONCAT(org.login,"/",r.name) as repo, CONVERT (hca.value,char) as url, hes.name as event, h.active, h.updated_at  from hooks h, repositories r, users org, hook_config_attributes hca, hook_event_subscriptions hes where installation_target_type="Repository" and h.installation_target_id=r.id and r.owner_id=org.id and org.login="org_name" and h.id=hca.hook_id and hca.key="url" and h.id=hes.subscriber_id and CONVERT(hca.value,char) like "%https://some-company-url/v1/github/webhook%";
  # +---------------+--------------------------------------------+------------------------+--------+---------------------+
  # | repo          | url                                        | event                  | active | updated_at          |
  # +---------------+--------------------------------------------+------------------------+--------+---------------------+
  # | org_name/repo | https://some-company-url/v1/github/webhook | branch_protection_rule |      1 | 2025-12-19 19:11:07 |
  # | org_name/repo | https://some-company-url/v1/github/webhook | pull_request           |      1 | 2025-12-19 19:11:07 |
  # | org_name/repo | https://some-company-url/v1/github/webhook | issue_comment          |      1 | 2025-12-19 19:11:07 |
  # +---------------+--------------------------------------------+------------------------+--------+---------------------+
```

Some key areas to be aware of in the above:
* Use a "like" statement to narrow down the URLs, if you exclude it ... you get the full list
* Use the 'org.login="org_name"' to also limit it for testing or to get the full list

This way, you can create a hash of the items (org_name/repo + url + event) so that if you do NOT see the match ("active = 1") in your result, you can use the API

### Scripting

Let's say you use PERL, you're going to want to break down the above into chunks for usage into a hash.  Personally I just read the entire thing into memory (plenty of memory on my systems) and parse it as an array.  If you're memory constrained in this day and of of DDR5 2x64G costing $1500 then ps.open it as a stream but I digress

#### Reading the output
```
    # Take each column as a variable
    my ($hook_repo, $hook_url, $hook_event, $hook_active, $hook_updated_at) = split (/\t/, $ARRAY_OUTPUT[$j]);

    # Sanity check if you get a NULL 
    for ($hook_repo, $hook_url, $hook_event, $hook_active, $hook_updated_at) {
        $_ = undef if defined($_) && $_ eq 'NULL';
    }
```

#### Structured Data

Here we'll just create a simple row of structured data
```
    # Creating a row of structured data
    my $row = {
          repo        => $hook_repo,
          url         => $hook_url,
          event       => $hook_event,
          active      => $hook_active,
          updated_at  => $hook_updated_at
    };
```

Create the key
```
    push (@ARRAY_WEBHOOKS, $row);
    # Creating the key
    my $key = join '|', $hook_repo // '', $hook_url // '', $hook_event // '';
    $webhook_lookup{$key} = $row;
```

#### Checking for validity

Later on, you can check for the existence of "active = 1" and if not, do the needful (in my case, apply the webhook correctly) using something like this
* $REPO is the org_name/repo
* $webhook_url is the URL we want to ensure is on the hook (because you may have a dozen of them
* Active is the go/no-go for the hook return as well.

```
      my $key = join '|', $REPO, $webhook_url, $event;

      if (($webhook_lookup{$key} && ($webhook_lookup{$key}->{active} == 1)))
      {
        # Hook is good ... we find it ... and it's active

      } else {
        # Otherwise the hook doesn't exist (for the params we look for) or is not active
        # So go and create the hook
      }
```

In addition to the above, you can create the array for event (aka list) and run through it because each $row in the hash of structured data can correspond to an event (or not) that you want to ensure

### Applying & updating the hook

It's going to be the same command via API using the -X POST (don't worry about the documentation claiming to use PATCH, the POST works to update the URL as well) 

```
  my $cmd_exec = "curl --silent -X POST -H 'Authorization: Bearer $GH_TOKEN' -H 'Content-Type: application/json' -d '{
    \"name\": \"web\",
    \"active\": true,
    \"events\": [\"$event\"],
    \"config\": {
      \"content_type\": \"json\",
      \"insecure_ssl\": \"0\",
      \"url\": \"https://some-company-url/v1/github/webhook\"
    }
  }' https://ghes-company-url/api/v3/repos/$ORG/$repo_name/hooks";
```

Obviously in the above, if we were using python, we'd fight a whole lot less with the "escape the quotes", eh?  But sometimes, we do want to obfuscate this from the next gen, low key fam?  Fax no printer.

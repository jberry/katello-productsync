# productsync
A CLI tool to help manage product synchronization for Katello

##Why does this exist?
I built this because I did not like how Sync Plans worked, allowing multiple to run at the same time.  This always led to sync issues.

Also, since I sync CentOS, there is a very fun order things have to happen in, and this makes it easy to do. The CentOS sync requires the following to happen:
- Sync all CentOS repositories
- Use a script to download/parse the CentOS errata from their mailing list
- Run an errata import script against the above script output, which requires CentOS already sync'd into Katello so it can find the packages to apply errata to
- Sync all the CentOS repositores again (and at this stage, I sync ALL products)

Also as mentioned, the built in Sync Plans setup allows multiple syncs to be running at the same time, with no health watching/watchdog for run times.  This is also addressed by this tool.

##Ok, makes sense.  How do I use it?
To get going, we do the following:

1. Build the YAML with our config header. An example file is included in this repo
2. You need to build your list of products to sync. The `listasyaml` action will output a list of all products known to Katello, for you to append to your config file
3. Edit the YAML file you appended to, and delete any Products you do not wish to sync
4. Start a screen session to run the `productsync sync` under, as it can take a long time to run
5. Run with the `sync` action (and any necessary flag options), and wait for it to be gone

##Available actions
* `sync`: This will sync any products listed in your config yaml file
* `list`: This will list out all known products to your Katello instance in standard text format
* `listasyaml`: Does the same as list,except outputs in YAML format for `>>` appending to your config file, for easy setup
* `health`: Pretty much the same thing as `hammer product list`, however it tells you if the product was found in your YAML or not. Nice for comparison of products vs. what you're syncing via your YAML

##Example configuration file and run output:
```
---
:settings:
    :user: admin
    :pass: changeme
    :uri: https://localhost
    :timeout: 300
    :org: 1
    :lockfile: /tmp/katello-productsync.lock
    :email: root@localhost
---
:products:
    - Red Hat CloudForms
    - Red Hat Satellite
    - Red Hat Satellite Capsule
```
* `user`: username of a Katello user to execute the actions with
* `pass`: password of the same user
* `uri`: URI of the Katello, `https://localhost` will work when executed directly on the Katello machine
* `timeout`: Timeout, in seconds, for any API calls made
* `org`: Organization ID (not name) for managing content in
* `lockfile`: The lockfile to use when running a sync, to ensure multiple arn't running at once
* `email`: Who to email results, or long-running warnings to

##Example run output:

Below is an example run output with 1 product in an error state (I force cancelled an earlier sync to put it into a bad state for example)
```
[root@mysat6 sat6-productsync]# ./productsync sync -c ../../productsync-example.yaml 
Syncing Red Hat CloudForms
   Time: 00:00:30 < Task ID 1c5e085e-ec1c-46e5-8d58-efb183c9d20f >  
   Syncing of Red Hat CloudForms completed. Result was success
Product Red Hat Satellite is not in a stopped state. Current state: error
Syncing Red Hat Satellite Capsule
   Time: 00:00:30 < Task ID 3b65e2d8-60ef-49db-825d-c6531d5a48e4 >  
   Syncing of Red Hat Satellite Capsule completed. Result was success
[root@mysat6 sat6-productsync]#
```
Below is the example email that was sent:

```
From: katello-productsync@mysat6.mycompany.com
To: me@mycompany.com
Subject: Sync results: 1 error
Body:
Syncing Red Hat CloudForms
Syncing of Red Hat CloudForms completed. Result was success
Product Red Hat Satellite is not in a stopped state. Current state: error
Syncing Red Hat Satellite Capsule
Syncing of Red Hat Satellite Capsule completed. Result was success
```


##Can I contribute?
Absolutely! The license is GPL3, so feel free to fork and play with it.  And if you make a beneficial change/bugfix, feel free to submit a pull request and I'll review!

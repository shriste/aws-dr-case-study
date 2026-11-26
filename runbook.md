# Recovery runbook

How to recover the WordPress site in this project. It follows the recovery steps I tested in my dissertation. Times are what I observed in my test runs, not guarantees.


## Before you start

- Region: eu-west-2 (London)
- Recovery strategy: Backup and Restore (manual RDS snapshots, S3 file backups)
- The recovery is finished only when the WordPress site loads and the content is correct. An AWS status of Running or Available is not enough.
- Write down the time the failure happened and the time you start recovery. Waiting before you start also counts as downtime.

## Scenario A: Database problem (restore from RDS snapshot)

1. Confirm the problem: the WordPress site shows a database error, or the RDS instance is not available.
2. In the RDS console, open Snapshots and choose the most recent manual snapshot. Note its creation time. Data created after that time will not be in the restored database.
3. Restore the snapshot. This creates a new RDS instance with a new endpoint.
4. Wait until the new instance shows Available (about 6 minutes in my tests).
5. Check the security group: the database must allow MySQL (port 3306) from the web server security group.
6. Update the WordPress database configuration so it points to the new RDS endpoint.
7. Restart the web service if the site still does not connect.
8. Test: open the site and check that older content, such as a known test post, is there. Total time in my test: about 11 minutes.

## Scenario B: Web server problem (EC2 stopped or unhealthy)

1. In the EC2 console, check the instance state and status checks.
2. Start the instance (or recover it if it is unhealthy).
3. Wait until the state is Running and the status checks show 2/2 passed. The Elastic IP keeps the public address the same.
4. Give the web service time to start, then open the site.
5. Test: the site loads and the content is correct. Active recovery time in my test: about 10 minutes.

## Scenario C: Full disaster (database and web server)

1. Do Scenario B first, so the web server is running.
2. Do Scenario A to restore the database from the latest snapshot.
3. Reconnect WordPress to the restored database endpoint and restart the web service.
4. Test the site and the content. Total time in my test: about 21 minutes, which was about 6 minutes over my 15-minute target.

## After recovery

- Record the times and what was slow. Manual steps and waiting for services to start were the main delays.
- Stop or delete anything you no longer need, to keep AWS costs down.
- Take a new manual snapshot so the next recovery has a fresh recovery point.

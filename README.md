# github-restic-backup 

The GitHub action can back up all repositories in a namespace (organization's or user's).
It encrypts them using `restic` before uploading them to your target storage - eg. AWS.

Restic also provides:
- deduplication
- cleaning up any backups past a retention period you specify


For how to pass in your S3 credentials, restic password and set the rotation period, see inputs in action.yml
For how restic handles backups, rotation, etc. see the excellent upstream docs: https://restic.net/


## Note on security

As with any 3rd party action, we recommend pinning this action to a commit, to ensure integrity, eg:

```
juno-fx/github-restic-backup@<commit sha>
```

The above is more secure than:
```
juno-fx/github-restic-backup@main
```


You can get the latest commit sha by clicking on "History" in the GitHub web ui - or running `git rev-parse HEAD` on a local checkout.




## Usage example

A simple example of how you might use this action:

```
name: Daily Github repo backup to S3
on:
  push:
  workflow_dispatch:
  schedule:
    - cron: "0 4 * * *"

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - name: Back up the Github organization
        uses: juno-fx/github-restic-backup@<commit sha>
        with:
          github_namespace: juno-fx
          github_access_token: ${{ secrets.GIT_PASS }}
          restic_password: ${{ secrets.GH_BACKUP_RESTIC_PASSWORD }}
          restic_repository: s3:https://s3.us-east-1.amazonaws.com/<your backup bucket>
          restic_image_tag: latest
          s3_access_key_id: ${{ secrets.GH_BACKUP_AWS_ACCESS_KEY_ID }}
          s3_secret_access_key: ${{ secrets.GH_BACKUP_AWS_SECRET_ACCESS_KEY }}
          validate_private_repos_presence: "true"
          restic_keep_last: 2
          restic_keep_daily: 7

```

### Github Actions caveats


Note that the approach above is an example - it does have one pitfall.
As with all scheduled jobs, if you offboard the user running it, it will become suspended. Keep that in mind and monitor it appropriately if you choose to reuse this example.

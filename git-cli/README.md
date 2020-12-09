# GIT commands on the command line

## git cherry-pick

In this example the hotfix-flow of the now-app is used to demonstrate the capabilities of the git cherry-pick command using the CLI.

### TLDR - just give me the commands already:

```
# on prod-now branch, get commit hash of desired commit
$ git log

# create tmp branch from master to merge changes
$ git checkout master
$ git checkout -b tmp/merge-prod-now-cherry-pick
$ git push --set-upstream origin tmp/merge-prod-now-cherry-pick

# cherry-pick the commit
$ git cherry-pick -x [YOUR_COMMIT_HASH]

# if conflicts, resolve, add the resolution & continue cherry-pick
$ git add .
$ git cherry-pick --continue

# check history (cherry-pick should be there)
$ git log

# push the changes
$ git push

# done! => create PR
```

### Step-by-Step Guide:

Branches:
- `master`
- `prod-now`

A hotfix change has been committed to the `prod-now` branch. The hotfix is deployed and it is time to merge the changes to the master branch. We can achieve this by cherry-picking our hotfix-commit from the `prod-now` branch and applying that commit on the `master` branch. In the end our commit message will even be appended by a cherry-pick message which tells us exactly where this commit originally came from.

Let's get started!

### Step 1 - get the commit hash of the commit you want to cherry-pick

Make sure you are on the `prod-now` branch and it is up to date:

```
$ git status
On branch prod-now
Your branch is up to date with 'origin/prod-now'.

nothing to commit, working tree clean
```

Display the log, and get the commit hash of the commit to pick

```
$ git log

###
# The terminal will show the history
# You can exit by simply pressing the letter 'q'
###

commit 3a0a23908173d9b960b865ed756ab6a6b116b5c5 (HEAD -> prod-qa-now, origin/prod-qa-now)
Author: Sandro Camastral <sandro.camastral@sap.com>
Date:   Wed Dec 9 14:35:53 2020 +0100

    some hotfix changes

commit 0e408f1718804174efd9c01720a246b53ef4d2f8
Author: Sandro Camastral <sandro.camastral@sap.com>
Date:   Wed Dec 9 14:12:14 2020 +0100

    added some new stuff

...
```

The commit hash of the desired commit in this case is:

`3a0a23908173d9b960b865ed756ab6a6b116b5c5`

You can also shorten that and just use the first 7 letters/digits:

`3a0a239`

Copy this, you will need it later.

### Step 2 - create a temporary branch to merge the changes

Switch back to the `master` branch and make sure it is up to date:

```
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

Create a temporary branch (used for the Pull Request on Github)

```
$ git checkout -b tmp/merge-prod-now-cherry-pick
Switched to a new branch 'tmp/merge-prod-now-cherry-pick'
```

Push this branch into the repository (and set the upstream branch)

```
$ git push --set-upstream origin tmp/merge-prod-now-cherry-pick
```

### Step 3 - git cherry-pick the commit (with merge of eventual conflicts)

Using the commit hash obtained above, we can now cherry pick the desired commit from the `prod-now` branch into our branch. The `-x` flag used here will append a line that says "(cherry picked from commit …​)" to the original commit message in order to indicate which commit this change was cherry-picked from.

The following steps depend on whether or not we will have any merge conflicts.

#### No conflicts:

```
$ git cherry-pick -x 3a0a239
[tmp/merge-prod-now-cherry-pick 3a0a239] some hotfix changes
 Date: Wed Dec 9 14:35:53 2020 +0100
 1 file changed, 1 insertion(+)
 create mode 100644 some-new-file.md
```

We are good to go, our commit has been cherry picked into our branch and we can proceed with Step 4.

#### Conflicts:

```
$ git cherry-pick -x 3a0a239
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
error: could not apply 3a0a239... some hotfix changes
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

If we have conflicts, we do have to merge them and add the resolution. After that it's IMPORTANT NOT TO USE THE `git commit` COMMAND to commit our merge, instead we will use `git cherry-pick --continue` to complete the cherry pick. The GIT CLI will also give you a hint regarding this:

```
# Check the status of the current situation
$ git status
On branch tmp/merge-prod-now-cherry-pick
You are currently cherry-picking commit 3a0a239.
  (fix conflicts and run "git cherry-pick --continue")
  (use "git cherry-pick --skip" to skip this patch)
  (use "git cherry-pick --abort" to cancel the cherry-pick operation)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   appconfig.json

no changes added to commit (use "git add" and/or "git commit -a")
```

After resolving the conflicts, add the resolution:

```
# A wildcard can be used to add all changes at once
# (you could also add each filepath speparately)
$ git add .
$ git status
On branch tmp/merge-prod-now-cherry-pick
You are currently cherry-picking commit 3a0a239.
  (all conflicts fixed: run "git cherry-pick --continue")
  (use "git cherry-pick --skip" to skip this patch)
  (use "git cherry-pick --abort" to cancel the cherry-pick operation)

Changes to be committed:
	modified:   appconfig.json
```

Now it's time to continue and finish the cherry-pick with our merged changes. Depending on your terminal setting, the editor that is openend to write the resulting commit-message will be different. For most, the default editor that will open is vim:

```
$ git cherry-pick --continue

###
# VIM EDITOR WILL OPEN AND SHOW THE FOLLOWING:
###

some hotfix changes

(cherry picked from commit 3a0a23908173d9b960b865ed756ab6a6b116b5c5)

# Conflicts:
#       appconfig.json
#
# It looks like you may be committing a cherry-pick.
# If this is not correct, please remove the file
#       .git/CHERRY_PICK_HEAD
# and try again.


# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Dec 9 14:35:53 2020 +0100
#
# On branch tmp/merge-prod-now-cherry-pick
# You are currently cherry-picking commit 3a0a239.
#
```

To save this message, just type `:wq` and press enter. After successfully merging and commiting our cherry-pick, the terminal will show:

```
$ git cherry-pick --continue
[tmp/merge-prod-now-cherry-pick f055d74] some hotfix changes
 Date: Wed Dec 9 14:35:53 2020 +0100
 1 file changed, 1 insertion(+)
```

You can now proceed with Step 4

### Step 4 - Check the git log

In order to verify that everything worked, we will check the git log to see if our cherry picked commit (including cherry-pick message) is in our history.

```
$ git log

###
# The terminal will show the history
# You can exit by simply pressing the letter 'q'
###

commit f055d746a55784caf8fa1231517f02cf00392f97 (HEAD -> tmp/merge-prod-now-cherry-pick)
Author: Sandro Camastral <sandro.camastral@sap.com>
Date:   Wed Dec 9 14:35:53 2020 +0100

    some hotfix changes

    (cherry picked from commit 3a0a23908173d9b960b865ed756ab6a6b116b5c5)

...
```

### Step 5 - Push changes & create a pull request

Finally we can push the changes:

```
$ git push
```

Now we successfully cherry-picked the commit from the `prod-now` branch into our branch. We can now create a pull request from this branch into the `master` branch to merge the hotfix changes back into it.

By using this method, we will have the appended cherry-pick message in the commit, allowing us to see where this change originally came from. Github even links the commit hash in this message to the original commit, so it's really easy to check where it originated from.

That's all, now have fun using it! :)

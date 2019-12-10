This pull request has conflicts.

*I. official instructions from Bitbucket: Option A*

You must resolve the conflicts by manually merging this branch into master. This will merge the pull request remotely.

Step 1: Checkout the target branch and merge in the changes from the source branch. Resolve conflicts.

{code:java}
git checkout master
git pull origin local_squid
{code}

Step 2: After the merge conflicts are resolved, stage the changes accordingly, commit the changes and push.

{code:java}
git commit
git push origin HEAD
{code}

Step 3: The pull request will be updated and marked as merged.

*II. Option B:*
https://stackoverflow.com/questions/39758967/resolving-pull-requests-with-merge-conflicts-when-using-branch-permissions-in-bi

On BitBucket server, when we get any conflict while merging any pull request, we can use git bash tool to resolve it on our local system and then we can commit and push our changes to remote feature branch and merge it to main branch.

Following steps need to be followed in order in our local system’s git bash tool.

(1) Open git bash tool and checkout or switch to your local feature branch.

(2) Pull the latest changes from the main branch (say 'master') into feature branch.


{code:java}
git pull origin master
{code}

(3) If above command fails due to some local changes then use below command to stash them otherwise move to next step.


{code:java}
git stash
{code}

followed by -


{code:java}
git pull origin master
{code}

(4) In case of conflict, automatic merge will fail so we need to merge it manually. Use below command to resolve conflicts.


{code:java}
git mergetool
{code}

By default, it will display all the available merge tools and one of them will be picked automatically. If we feel we are much comfortable with any other tool then we can also configure that and git will open that tool for us for conflict resolution.

(5) Once the conflicts are resolved then commit the changes into feature branch.


{code:java}
git commit
{code}

(6) Push the changes to remote feature branch.


{code:java}
git push
{code}

Verify on BitBucket server, now pull request should get updated automatically.

Again try to merge it; in case of no conflict it will get merged successfully.

If it has merge conflict again (if someone has committed new changes in main branch during we were resolving conflict on our local system) then follow the above steps again to resolve them.

We should be able to successfully resolve any conflicts if we have followed the above steps in order.

Thanks, hope it helps.
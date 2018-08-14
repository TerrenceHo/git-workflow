# A Successful Git Workflow for Teams

Here at UCLA DevX, individual developers work on small teams, pushing commits to shared projects. Here we describe a workflow that has proven to work well for all our teams, and we heavily encourage you and your teams to adopt it. This workflow details how to manage a code repository so that development is quick, production releases are simple, and fewer code conflicts are possible (less chance of leaving the git repository in a hard-to-salvage-state-that-requires-git-magic.)

_Note that this tutorial assumes you have a general understanding of version control and basic knowledge of using git_.

### Github/Central Repository
Although Git was developed to be a decentralized version control system, common usage has devolved to being a centralized version control system. For the rest of the article, we refer to _origin_ as the central repository where a member can clone the repository, commit changes, and push code.  For many, GitHub is this central repository. 

We may also refer to a remote called _upstream_, where you did not clone your code initially, but is still where you want your commits to be merged into. This only has relevance in forking.

When working in a small team, when you have created new commits and are ready to let others use your new code, or just want to have another copy somewhere else, you push your code to _origin_. Once push to _origin_, others can pull/clone your code easily and run it at their leisure.  This also allows two or more develops to work on the same branch/feature together, where they push and pull from the same branch on _origin_.

![central_repository](http://puredev.eu/wp-old-content/2011/08/image2.png)

#### Person-to-Person Fetches
It is possible that two developers may not want to share their branch to the central repository, for fear that a third person may work off their code, and introduce complicated, migraine-inducing merge conflicts.  In this case, git supports developer-to-developer code pulls, but that is beyond the scope of this article.

### Master and Develop Branches

To develop a feature, make a branch. Branch naming semantics can be left up to the team, aside from these protected names: _master_, _develop_, and _patch_. (Patches will be discussed later)

The _master_ branch should be stable. No one should be able to push directly to master. Any new member should be able to clone master, and run off the latest __stable__ version of your software, with no potential bugs.  Why? The reason being, we require a branch to mark releases and tag versions with, a branch that is not constantly changed all the time.  That way, no broken features are left on the _master_ branch. 

Instead, code is pushed to a _develop_ branch. The tip of _develop_ is always at least even with the _master_ branch, and almost always ahead in commits.  When code is in _develop_ is ready for production, it is merged into _master_, tagged with a version, and marked as a release.

This should almost always be a fast-forward merge, rather than a recursive merge. The illustration below shows a develop and master branch in action.

![master_develop](https://nvie.com/img/main-branches@2x.png)

This has many benefits. If you were to make nightly builds (latest builds with new features) for testers, you would build off the tip of _develop_.  Only when you are certain the tip of _develop_ is stable and ready for release, is _develop_ merged into _master_.

Additioanlly, since _master_ is strictly related to production code, a script/webhook can be triggered when a commit is pushed to _master_ and automatically roll out to production.

(We highly recommend that on Github, pushing code directly to master is not allowed. Instead, pull requests must be made to propose and merge code.)

### Feature Branches

Even with _master_ locked down, we do not want developers pushing nilly wily to _develop_ either. _Develop_ represents the working tip, not broken tip. We want code in _develop_ to work, albeit not production ready. _Develop_ should be left in a state where anyone can pull the branch and work off it.

Instead, create a new branch, that branches off from _develop_. Feature branches branch off from develop, and __must__ merge back into develop, if it is merged at all. To ensure that you branch off from _develop_, use the command: 
```
git checkout -b my_new_feature develop
```
When you are done creating the feature, merge the feature branch back into the _develop_ branch. Now people wanting to work off your code can do so.

![feature_branches](https://nvie.com/img/fb@2x.png)

Note that almost no one should develop off another feature branch to make a feature, unless you coordinate closely.  Always branch off _develop_.  Treat other branches as though you did not own them. 

Consider branch _featureA_ that was branched off _develop_. A new function _coolFunc_ was made. _featureB_ branches off _featureA_, and uses _coolFunc_ to implement their feature. However, _featureA_ continues to commit off his branch, and realizes that a better implementation of _coolFunc_ was possible. Now you have diverging paths of code, that rely on the same namespace.  Eventually, _coolFunc_ will cause a merge conflict, if both branches are merged back in, and this must be resolved. If _coolFunc_ diverged enough, one of the branches may be invalidated and require extensive rework. (This disregards the order of merging, another complication)

_featureB_ should have 1) merged off of _develop_, to work off clean code, and 2) either waited for _featureA_'s function to become stable, or implemented their own version.  By always branching off of _develop_, situations like these are easily avoided.


### Patches
All code has bugs. It just depends on how many. And sometimes you can't wait for the next production release to fix the bug, it must be fixed ASAP! Then we use patch branches. 

Patch branches introduce code to fix a big.  They are branched from _master_, (where the bug exists), and merged back into _master_ and _develop_.  It must be merged back into both, so that _master_ does not have functionality that does not exist in _develop_. We do not branch off _develop_, because that branch may have moved beyond _master_, and merging the hotfix into _master_ will also introduce newer, less tested code.

![patches](https://nvie.com/img/hotfix-branches@2x.png)

### Overall Workflow
![workflow](https://nvie.com/img/git-model@2x.png)

Here we bring together the overall ideal workflow. We have a _master_ branch feeding into production releases. A _develop_ branch is branched off from to create features, and then merged back in. When _develop_ has reached maturity/enough new features, it is merged back into _master_, version tagged, and released for production. Occasionally, a hotfix is created and push to both _develop_ and _master_, to keep feature parity.

You can even throw in other branches between _develop_ and _master_, such as a _qa_ branch for testing. But this describes a basic workflow that should work for most teams.

### Advanced

#### Fast forward or not?
When merging should you use the `--no-ff` option?

When merging with `--no-ff`, you exclude a merge commit, an extra commit denoting that a code merge was made at this point in the history.  If you do not make a merge commit, a feature branch fits directly into the history, with no indication it was created in a branch. The difference can be seen when you use `git log --graph`. 

Some people like having the merge commit, because it keeps it more true to the history. On the other hand, many like not having the merge commit, because it keeps the history cleaner, more linear, and easier to read.  This should be decided in teams.

![fast_forward](https://nvie.com/img/merge-without-ff@2x.png)

#### Rebasing

Rebasing is the act of taking a branch, and moving it to the tip of another branch. Here, we move a feature branch on top of a _master_ branch. Afterwords, you can fast-forward your branch so that master sits at the tip. You are basically rewriting history, in an effort to keep your history linear. No "merge commit" is created, and the history remains linear.

![rebase](https://wac-cdn.atlassian.com/dam/jcr:5b153a22-38be-40d0-aec8-5f2fffc771e5/03.svg?cdnVersion=jo)

If you rebased master on top of your feature branch, the history would look like this. 

![rebase_wrong](https://wac-cdn.atlassian.com/dam/jcr:1d22f018-b2c7-4096-9db1-c54940cf4f4e/05.svg?cdnVersion=jo)

The issue with this is that _master_ has diverged from other people's _master_ branch, and so conflicts will arise. 

The golden rule of rebasing is to never do it on public branches. Rebasing is dangerous precisely because it rewrites history, but it can be very useful (for those anal about git histories). With great power comes great responsibility. Ask yourself, is anyone else using this branch?


#### Squashing
When you develop locally, you often make spurious commits, with terrible commit messages. These commits don't tell you what was changed, and make it harder to read the history. Thus, it is recommended that before merging a feature branch into _develop_ that you combine all your commits into one or two meaningful commits that have detailed commit messages.

To do this:
```
git rebase -i HEAD~4
```
Here you replace 4 with the number of commits that you want to squash. In the rebase text editor, simply replace the words "pick" with "squash", and git will combine together those messages. You can then change your commit message. This makes use of the interactive rebase, where you can also do your normal rebase actions.

NEVER squash commits that other people have worked off of (i.e. _master_ and _develop_ branches). You can destroy other people's contributions and changes, and will cause conflicts when other people try to push into that branch, since the history has diverged. Only squash your own branch. If you have pushed your feature branch to _origin_, then you will have to force push after you squash. But again, never push to a main branch.

#### Branch Pruning
`git branch -d branchName` to delete branches. Always do this after merging in a feature, otherwise you end up with 1000 branches. Additionally, it removes the temptation to develop off a stale branch.

#### Forks
Github has the ability to create a fork of a repository, that you own. Afterwards, you can commit to your fork, push it back to Github, and then submit a pull request to get your code merged into mainstream. 

To keep your fork up to date set the original repository as _upstream_ and your forked repository _origin_. 
```
git remote add upstream git://github.com/ORIGINAL-DEV-USERNAME/REPO-YOU-FORKED-FROM.git
git remote add origin git://github.com/YOUR-USERNAME/FORKED-REPO
```

Make commits to _origin_, and when you are ready to merge your code, make a pull request to _upstream_. 

When upstream moves ahead, you'll want to grab those commits too. Simply pull from _upstream_ to keep your fork up to date, and push to origin.
```
git pull upstream master
git push origin master
```

### Last Words

Obviously, teams may adapt this strategy to their needs, but the basic workflow should be similar. 

### Author
Written by the illustrious and great Terrence Ho.

Other contributors include: ...

### Sources
<https://nvie.com/posts/a-successful-git-branching-model/>

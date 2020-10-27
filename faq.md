# Git FAQ

This document provides a number of targetted answers to questions that
are likely to come up often for users, developer and integrators.

## Users

### How do I track -current and -stable with only one copy of the repo?

**Q:** Although disk space isn't a huge issue, it's more efficient to use
only one copy of the repository. With svn mirroring, I could checkout
multiple trees from the same repo. How do I do this with git?

**A:** You can use git worktrees. There's a number of ways to do this,
but the simplest way is to do a clone to track -current, and a
worktree to track stable releases. While using a 'naked repository'
has been put forward as a way to cope, it's more complicated and won't
be documented here.

First, you need to clone the FreeBSD repository, shown here cloning into
`freebsd-current` to reduce confusion. $URL is whatever mirror works
best for you:
```
% git clone $URL freebsd-current
```
then once that's cloned, you can simply create a worktree from it:
```
% cd freebsd-current
% git worktree create ../freebsd-stable-12 stable/12
```
this will checkout `stable/12` into a directory named `freebsd-stable-12`
that's a peer to the `freebsd-current` directory. Once created, it's updated
very similar to how you might expect:
```
% cd freebsd-current
% git checkout main
% git pull --ff-only
# changes from upstream now local and current tree updated
% cd ../freebsd-stable-12
% git merge --ff-only origin/stable/12
# now your stable/12 is up to date too
```
I recommend using `--ff-only` because it's safer and you won't accidentally
get into a 'merge nightmare' where you have an extra change in your tree,
forcing a complicated merge rather than a simple one.

## Developers

### Ooops! I committed to `main` instead of a branch.

**Q:** From time to time, I goof up and commit to main instead of to a
branch. What do I do?

**A:** First, don't panit.

Second, don't push.

The following answer assumes you committed to `main` and want to
create a branch called `issue`:
```
% git checkout -b issue     # Create the issue branch
% git checkout master       # Go back to main branch
% git reset --hard HEAD^    # Reset what master references
% git checkout issue        # Back to where you were
```
The above assumes one commit, hence the `HEAD^`. You can put anything
there, but `HEAD^` or `HEAD^^` are the most typical.

### Ooops! I committed something to the wrong branch!

**Q:** I was working on feature on the `wilma` branch, but
accidentally committed a change relevant to the `fred` branch
to that branch. What do I do?

**A:** The answer is similar to the previous one, but with
cherry picking. This assumes there's only one commit on wilma,
but will generalize to more complicated situations. It also
assumes that it's the last commit on wilma (hence using wilma
in the `git cherry-pick` command), but that too can be generalized.

```
# We're on branch wilma
% git checkout fred		# move to fred branch
% git cherry-pick wilma		# pull in wrong commit
% git checkout wilma		# go back to wilma branch
% git reset --hard HEAD^	# move what wilma refers to
```

**Q:** I want split off some chagnes from branch `wilma` into branch `fred`

**A:** The more general answer would be the same as the
previous. You'd checkout/create the `fred` branch, cherry pick the
changes you want from `wilma` one at a time, then rebase `wilma` to
remove those changes you cherry picked. `git rebase -i main wilma`
will toss you into an editor, and remove the `pick` lines that
correspond to the hashes you committed to `fred`. If all goes well,
and there are no conflicts, you're done. If not, you'll need to
resolve the conflicts as you go.

The other way to do this would be to checkout `wilma` and then create
the branch `fred` to point to the same point in the tree. You can then
`git rebase -i` both these branches, selecting the changes you want in
`fred` or `wilma` by retaining the pick likes, and deleting the rest
from the editor. Some people would create a tag/branch called
`pre-split` before starting in case something goes wrong in the split,
you can undo it with the following sequence:
```
% git checkout pre-split	# Go back
% git branch -D fred		# delete the fred branch
% git checkout -B wilma		# reset the wilma branch
% git branch -d pre-split	# Pretend it didn't happen
```
the last step is optional. If you are going to try again to
split, you'd omit it.

**Q:** But I did things as I read a long and didn't see your advice at
the end to create a branch, and now `fred` and `wilma` are all
screwed up. How do I find what the `wilma` has was before I started. I don't
know how many times I moved things around.

## Integrators

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

## Integrators

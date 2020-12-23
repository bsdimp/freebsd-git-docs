# Github mirroring plan

# How to migrate from old hashes to new

Once we start mirroring the new git repo to github, old users of
https://github.com/freebsd/freebsd will need to update.

## Simple case: just tracking

If you are just tracking, without any changes, then there are only two cases. If you are tracking 'master' then you'll need to checkout main now:
```
% git pull
% git checkout main
```
If you are tracking one of the stable/XX branches, then a simple `git pull` will do the trick.

## I have work off the `master` branch

If you have branches that you need to migrate, there are two ways: in
place or separate tree. Most people will want to use the in-place
migration technique if they have only a few changes. For people with a
lot of changes, it may make sense to create a new tree and pull those
work branches in one at a time.

### In Place

Here the instructions are simple. Pull the new repo. For each branch you have,
rebase:
```
% git rebase master my-branch --onto main
```
this will rebase things forward and then you can continue as you have.

### Separate tree

If you have a lot of changes, it may make sense to use a fresh
tree. Clone the github repo. I'll assume that the two trees are
/tree/old and /tree/new for simplicity.

```
% cd /tree
% git clone https://github.com/freebsd/freebsd-src new
% cd new
% git remote add old /tree/old
```
This creates the new tree and creates a remote called 'old'. Now, you can
pull in the branches you want to migrate, as you want to migrate them:
```
% git fetch old my-branch
% git rebase master my-branch --onto main
```

This works becuase 'master' is the same in both repos. If you are
reading this after the planned removal of 'master' from the github
mirror, please see the section on migrating stable/X branches.

### Using patches

There's a third way if the repos are completely disconnected for
some reason. It's rather long, but see
http://bsdimp.blogspot.com/2020/08/how-to-transport-branch-from-one-git.html

## Migrating a stable/XX branch

Since we replaced the stable/XX definitions on github, if you have
branches from stable/XX you may need the old references.

```
% git remote add legacy https://github.com/freebsd/freebsd-legacy
% git fetch legacy
```
This looks like it will fetch a buch of stuff, but really it just fetches the
legacy branch names so you can refer to them in the rebase:
```
% git rebase legacy/stable/12 my-branch --onto stable/12
```
then is how to convert (once you've updated the stable/12
branch from the old refs to the new).

# Current Plans

## FreeBSD History

The FreeBSD project is currently mirroring to github an experimental
tree. Due to the large number of bugs that adversely affects the
quality of the tree, and its day to day use for anything more
complicated that merging and commits and simple history browsing, the
move to git project opted to redo the hashes and fix many historical
mistakes as part of its effort. As such, a migration plan is needed,
so this describes it.

## The Plan for src

1. Push a copy of the long-runing 'beta hashes' repo to github as freebsd-legacy
3. rename github/freebsd -> github/freebsd-src. Old users that have freebsd/freebsd
   in their .git/config will get the new stream.
4. Add the merge commit to freebsd-legacy that we need for subtree merges (for each of
   the branches that are active)
5. At some point, delete 'master' from the new freebsd-src.

With this plan, all the upward migration paths are preserved.

For folks tracking master, they will see no updates until they cut
over. This is -current, and people are expected to pay attention, so
this is OK.

Folks with work off stable/* will see forced push and will have to do
something special. My notion is the special thing will be
```
% git remote add legacy https://github.com/freebsd/freebsd-legacy
% git fetch legacy
% foreach branch B:
  git rebase legacy/stable/X B --onto stable/X
```
the fetch legacy will largely be a nop since they have all the
objects, just not the refs. Once they do the conversion, they can get
rid of the double history with git gc.

We put this in the readme.md (and delete readme, but that's a separate
issue) and a pointer to the doc.

## Plan for doc

1. fork freebsd-doc-legacy
2. delete the delgacy stuff except master.
3. keep master around for N months (6?) and then delete it.
4. Add note to readme about how to jump.
5. NOTE: historical branches will be deleted so only 'main' reamins on github.



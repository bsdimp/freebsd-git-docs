# FreeBSD Doc Committer Tranistion Guide

This document is designed to walk people through the conversion process from subversion to git, written from the doc committer's point of view. This document is a living document, so please don't hesistate to send improvements, or even ask for areas to be explained more / better / at all.

## Git Basics

There are many primers on how to use git on the web. There's a lot of them (google "git primer"). This one comes up first, and is generally good. https://danielmiessler.com/study/git/ and https://gist.github.com/williewillus/068e9a8543de3a7ef80adb2938657b6b are good overviews. The git book is also complete, but much longer https://git-scm.com/book/en/v2 .

This document will assume that you've read through it and will try not to belabor the basics (though it will cover them briefly).

## Migrating from a subversion tree

This section will cover a couple of common scenarios for migrating from using the FreeBSD subversion repo to the FreeBSD docs repo. The freebsd git conversion is still be beta status, so some minor things may change between this and going into production.

Before you git started, you'll need a copy of git. Any git will do, though the latest ones are always recommeneded. Either build it from ports, or install it using pkg (though some folks might use `su` or `doas` instead of `sudo`):
```
% sudo pkg install git
```

### No staged changes migration

If you have no changes pending, the migration is straight forward. In this, you abandon the subversion tree and clone the git repo. It's likely best to retain your subversion tree, in case there's something you've forgotten about there.  First, let's clone a repo:
```
% mkdir git-docs
% cd git-docs
% git clone https://cgit-beta.freebsd.org/doc freebsd-doc
```
will create a clone of the FreeBSD doc repo into a subdirectory called `freebsd-doc`. I selected that name because there's an excellent chance that the repo will change from `doc` to `freebsd-doc` before we publish the final repo. The current plan for github mirroring is to mirror to https://github/freebsd/freebsd-doc as well, but more on that later.

Now, it's useful to have the old subversion revisions, so let's fetch that information. This data is stored using git notes, but git doesn't fetch those by default. It's best to add them now, especially if you are a translator. Sadly, there's no way to add this to the `git clone` command.
```
% git config --add remote.origin.fetch "+refs/notes/*:refs/notes/*"
% git fetch
```

At this point you have the docs checked out into a git tree, ready to do other things.

### But I have changes that I've not committed

If you are migrating from a tree that has changes you've not yet committed to FreeBSD, you'll need to follow the steps from the previous section first, and then follow these.

```
% cd path-to-svn-checkout-tree
% svn diff > /tmp/docs.diff
% cd git-docs/freebsd-doc
% git checkout -b working
```
This will create a diff of your current changes. The last command creates a branch called `working` though you can call it whatever you  want.
```
% git apply /tmp/docs.diff
```
this will apply all your pending changes to the working tree. This doesn't commit the change, so you'll need to make this permanant:
```
% git commit
```
The last command will commit these changes to the branch. The editor will prompt you for a commit message. Enter one as if you were committing to FreeBSD.

At this point, your work is preserved, and in the git repo.

### Keeping current

So, time passes. It's time now to update the tree for the latest changes upstream. When you checkout `main` make sure that you have no diffs. It's a lot easier to commit those to a branch (or use `git stash`) before doing the following. I recommend the longer form here since the combination command `git pull` can be harder to recover from if something goes wrong.
```
% cd git-docs/freebsd-doc
% git checkout main
% git fetch origin
% git merge --ff-only origin/main
```
These commands reset your tree to the main branch, and then update it from where you pulled the tree from originally. It's important to switch to `main` before doing this so it moves forward. Now, it's time to move the changes forward:
```
% git rebase -i main working
```
This will bring up an interactive screen to change the defaults. For now, just exit the editor. Everything should just apply. If not, then you'll need to resolve the diffs. https://docs.github.com/en/free-pro-team@latest/github/using-git/resolving-merge-conflicts-after-a-git-rebase can help you navigate this process.

### Time to push changes upstream

Note: git is still in beta, so your changes won't go into the subversion repo and will be lost if you do this before the cutover.

```
% git checkout main
% git merge --ff-only working
% git push
```
The above commands merge the 'working' tree into main line and push them upstream. It's important that you curate your changes to be just like you want them in the FreeBSD doc repo before doing this.

## Migrating from github fork

This section needs to be completed, but only if people need to do this.


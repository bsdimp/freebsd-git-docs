# FreeBSD Doc Committer Tranistion Guide

This document is designed to walk people through the conversion
process from Subversion to Git, written from the doc committer's point
of view. This document is a living document, so please don't hesitate
to send improvements, or even ask for areas to be explained more /
better / at all. Please note that the URL for the repo is currently
https://cgit-beta.freebsd.org/doc, but that will change when this is
put into production.


## Old vs New URL translation table

Before we get started, here's a handy cheat sheet for old to new URLs.

SVN infra -> Git infra map

| Item                                     | SVN                             | Git                                 |
| ---------------------------------------- | ------------------------------- | ----------------------------------- |
| Web-based repository browser             | https://svnweb.freebsd.org      | https://cgit.freebsd.org            |
| Distributed mirrors for anonymous readonly checkout/clone | https://svn.freebsd.org svn://svn.freebsd.org | https://git.freebsd.org git+ssh://anongit@git.freebsd.org |
| Read/write Repository for committers (*) | svn+ssh://(svn)repo.freebsd.org | git+ssh://git@(git)repo.freebsd.org |

(*) Before all repositories in SVN have been migrated, the repo.freebsd.org will be pointing to one of:
    - svnrepo.freebsd.org
    - gitrepo.freebsd.org

please use the hostname that explicitly includes the VCS name to
access the right repositories during the migration. `repo.freebsd.org`
will be the canonical FreeBSD Git repository for the committers after
all the repositories migrated to Git.

## Git basics

There are many primers on how to use Git on the web. There's a lot of
them (google "Git primer"). This one comes up first, and is generally
good. https://danielmiessler.com/study/git/ and
https://gist.github.com/williewillus/068e9a8543de3a7ef80adb2938657b6b
are good overviews. The Git book is also complete, but much longer
https://git-scm.com/book/en/v2 .

This document will assume that you've read through it and will try not
to belabor the basics (though it will cover them briefly).

## Migrating from a Subversion tree

This section will cover a couple of common scenarios for migrating
from using the FreeBSD Subversion repo to the FreeBSD docs repo. The
FreeBSD Git conversion is still be beta status, so some minor things
may change between this and going into production.

Before you git started, you'll need a copy of Git. Any Git will do,
though the latest ones are always recommemded. Either build it from
ports, or install it using pkg (though some folks might use `su` or
`doas` instead of `sudo`):
```
% sudo pkg install git
```

### No staged changes migration

If you have no changes pending, the migration is straight forward. In
this, you abandon the Subversion tree and clone the Git repo. It's
likely best to retain your subversion tree, in case there's something
you've forgotten about there.  First, let's clone a repo:
```
% mkdir git-docs
% cd git-docs
% git clone --config remote.origin.fetch='+refs/notes/*:refs/notes/*' https://cgit-beta.freebsd.org/doc.git freebsd-doc
```
will create a clone of the FreeBSD doc repo into a subdirectory called
`freebsd-doc` and include the 'notes' about the revisions. I selected
that name because there's an excellent chance that the repo will
change from `doc` to `freebsd-doc` before we publish the final
repo. The current plan for GitHub mirroring is to mirror to
https://github.com/freebsd/freebsd-doc.git as well, but more on that
later.

It's useful to have the old Subversion revisions. This data is stored
using Git notes, but Git doesn't fetch those by default. The --config
and is argument above changed the default to fetch the notes. If
you've cloned the repo without this, or wish to add notes to an
previsouly clone repository, use the following commands:
```
% git config --add remote.origin.fetch "+refs/notes/*:refs/notes/*"
% git fetch
```
At this point you have the docs checked out into a Git tree, ready to
do other things.

### But I have changes that I've not committed

If you are migrating from a tree that has changes you've not yet
committed to FreeBSD, you'll need to follow the steps from the
previous section first, and then follow these.
```
% cd path-to-svn-checkout-tree
% svn diff > /tmp/docs.diff
% cd git-docs/freebsd-doc
% git checkout -b working
```
This will create a diff of your current changes. The last command
creates a branch called `working` though you can call it whatever you
want.
```
% git apply /tmp/docs.diff
```
this will apply all your pending changes to the working tree. This
doesn't commit the change, so you'll need to make this permanent:
```
% git commit
```
The last command will commit these changes to the branch. The editor
will prompt you for a commit message. Enter one as if you were
committing to FreeBSD.

At this point, your work is preserved, and in the Git repo.

### Keeping current

So, time passes. It's time now to update the tree for the latest
changes upstream. When you checkout `main` make sure that you have no
diffs. It's a lot easier to commit those to a branch (or use `git
stash`) before doing the following. I recommend the longer form here
since the combination command `git pull` can be harder to recover from
if something goes wrong.
```
% cd git-docs/freebsd-doc
% git checkout main
% git fetch origin
% git merge --ff-only origin/main
```
These commands reset your tree to the main branch, and then update it
from where you pulled the tree from originally. It's important to
switch to `main` before doing this so it moves forward. Now, it's time
to move the changes forward:
```
% git rebase -i main working
```
This will bring up an interactive screen to change the defaults. For
now, just exit the editor. Everything should just apply. If not, then
you'll need to resolve the
diffs. https://docs.github.com/en/free-pro-team@latest/github/using-git/resolving-merge-conflicts-after-a-git-rebase
can help you navigate this process.

### Time to push changes upstream

Note: Git transition is still in beta, so your changes won't go into
the Subversion repo and will be lost if you do this before the
cutover.

```
% git checkout main
% git merge --ff-only working
% git push
```
The above commands merge the 'working' tree into main line and push
them upstream. It's important that you curate your changes to be just
like you want them in the FreeBSD doc repo before doing this.

### Finding the Subversion Revision

You'll need to make sure that you've fetched the notes (see the `No
staged changes migration` section above for details. Once you have
these, notes will show up in the git log command like so:
```
commit 77788b7b18c1569e368e845cfbd4739329a2b76f
Author: phk <phk@FreeBSD.org>
Date:   Mon Nov 23 09:00:37 2020 +0000

    Update my GPG key

Notes:
    svn path=/head/; revision=54705
```

If you have a specific version in mind, you can use this construct:
```
% git log --grep revision=54700
commit 9a9960bfb7e8ff9fa7fe04eb488eabb1fd06814b
Author: dbaio <dbaio@FreeBSD.org>
Date:   Sun Nov 22 20:57:38 2020 +0000

    pt_BR/articles/freebsd-releng: Sync with en_US r54689

    Approved by:    ebrandi (doc)
    Obtained from:  https://translate-dev.freebsd.org
    Differential Revision:  https://reviews.freebsd.org/D27315

Notes:
    svn path=/head/; revision=54700
%
```
to find the specific revision. The hex number after 'commit' is the
hash you can use to refer to this commit.

## Migrating from GitHub fork

This section needs to be completed, but only if people need to do this.

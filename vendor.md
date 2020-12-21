# How to do vendor imports

**THIS IS A DRAFT DOCUMENT AND CONTAINS SOME WORKING NOTES**

With the FreeBSD Subversion to Git conversion complete,	it's time to
look at how to do a vendor import.

There are hundreds, if not thousands of	tags related to	vendor imports
that the FreeBSD project has created over the years. To	keep all this
clutter from getting in the way, vendor	branches were imported in
their own name space, in much the same way notes are.

To import notes (helpful when you want to look up svn revisions), for example, one would clone the repository and then do:
```
git config --add remote.freebsd.fetch '+refs/notes/*:refs/notes/*' && git fetch
```

Replace `freebsd` with the actual remote name in your clone.

To retrieve the	vendor branches and tags, do the following:
```
git config --add remote.freebsd.fetch '+refs/vendor/*:refs/vendor/*' && git fetch
```
This will pull in all the vendor relevant stuff. You should expect to
need a fair amount of disk space for this operation. In	the future,
this guide will no doubt be updated to optimize	this for the specific
software you're looking to use.

The rest of this document assumes that you've done the above.

## First step: Creating	a vendor branch

When we	imported the CVS vendor	branches into Subversion, the project
required a 'flattening' step before you	could update the vendor branch.
The same is true for for the Subversion to Git conversion. The
automation could only do so much, and some manual adjustment is needed
for some things.

We'll use `mtree` as an example for this process. It's a small program
that's easy to understand the changes from the last import. And it also
tickles at least one of the special cases that needs some help.

The path to the vendor branch in subversion was [base/vendor/NetBSD/mtree](https://svnweb.freebsd.org/base/vendor/NetBSD/mtree/) making it a nested vendor
branch in some ways. The conversion process didn't actually make it a 'nested branch'
but preserved the namespace.

So, the first step is to create a vendor branch for mtree. Step 1 is to verify that the subversion
conventions were followed by looking at ref/vendor/NetBSD/mtree/dist
```
% git log refs/vendor/NetBSD/mtree/dist
commit 65547801490dc5cc1554d222b55706c5709de9cc (refs/vendor/NetBSD/mtree/dist, tag: refs/vendor/NetBSD/mtree/20141028)
Author: Brooks Davis <brooks@FreeBSD.org>
Date:   Tue Oct 28 16:24:07 2014 +0000

    Vendor import of NetBSD's mtree(8) at 2014-10-28

commit a90dc36c0d24a9956fd254d8781c18e03bf83a48 (tag: refs/vendor/NetBSD/mtree/20131121)
Author: Brooks Davis <brooks@FreeBSD.org>
Date:   Thu Nov 21 19:16:52 2013 +0000

    Vendor import of NetBSD's mtree at 2013-11-21

commit 675e2c61fa88e791c757f4402381c0884c5c0187 (tag: refs/vendor/NetBSD/mtree/20131016)
Author: Brooks Davis <brooks@FreeBSD.org>
Date:   Wed Oct 16 22:28:08 2013 +0000

    Vendor import of NetBSD's mtree at 2013-10-16

commit 55d28badaa21a43a3f3ca71c369113c6179b6ff9 (tag: refs/vendor/NetBSD/mtree/20130408)
Author: Ed Schouten <ed@FreeBSD.org>
Date:   Mon Apr 8 19:44:30 2013 +0000

    Vendor import of NetBSD's mtree at 2013-04-08.

commit 89a652b23536a594be53247b21c1c305749fccfb (tag: refs/vendor/NetBSD/mtree/20122112)
Author: Brooks Davis <brooks@FreeBSD.org>
Date:   Fri Dec 21 16:54:00 2012 +0000

    Vendor import of NetBSD's mtree at 2012-12-21
```
They were followed. Good. And there's only a few versions, so I'm able to list them all here. And it appears they followed the naming scheme of YYYYMMDD for the imports (NetBSD doesn't have frequent enough releases to just pull from a release).

Since the trees are usually a tiny subset of the FreeBSD, it's best to do them with work trees since the process is quite fast. Make sure that whatever directory you choose (the `../mtree`) argument is empty and doesn't conflict.
```
% cd ...../src
% git worktree add -b vendor/NetBSD/mtree ../mtree refs/vendor/NetBSD/mtree/dist
```

If you'd already bootstrapped the above, then you'd be able to just do:
```
% git worktree add ../mtree vendor/NetBSD/mtree
```
and you'd have the same thing.
### Update the Sources in the Vendor Branch

I have my copy of NetBSD checked out from their github mirror in `~/git/NetBSD`, so I'll update from there:
```
% cd ../mtree
% cp ~/git/NetBSD/usr.sbin/mtree/* .
% git diff
...
% git status
...
% git commit -a -m"Vendor import of NetBSD's mtree at 2020-12-11"
[vendor/NetBSD/mtree 8e7aa25fcf1] Vendor import of NetBSD's mtree at 2020-12-11
 7 files changed, 114 insertions(+), 82 deletions(-)
% git tag -a vendor/NetBSD/mtree/20201211
```

Note: I used git commit -a here because I was sure I knew what I was doing and I'd run the `git diff` and `git status` commands to make sure nothing weird was present. Normally I only recommend using that when you're an expert and have checked what's going on to be sure.

It's also important to create an annotated tag, otherwise the push will be rejected.

### Updating the FreeBSD Copy
Now you need to update the mtree in FreeBSD. The sources live in `contrib/mtree` since it's upstream software.

At this point you can push it upstream
```
% git push --follow-tags freebsd vendor/NetBSD/mtree
```

`--follow-tags` tells `git push` to also push tags associated with the locally committed revision.

### Updating the FreeBSD source tree

```
% cd ../src
% git subtree merge  -P contrib/mtree vendor/NetBSD/mtree
```

This would generate a subtree merge commit of `contrib/mtree` against the local `vendor/NetBSD/mtree` branch.
If there were conflicts, you would need to fix them before committing.

### Rebasing your change against latest FreeBSD source tree

Because the current policy recommends against using merges, if the upstream FreeBSD `main` moved forward
before you get a chance to push, you would have to redo the merge.

Regular `git rebase` or `git pull --rebase` doesn't know how to rebase a merge commit **as a merge commit**,
so instead of that you would have to recreate the commit.

The easiest way to do this would be to create a side branch with the **contents** of the merged tree:

```
% cd ../src
% git fetch freebsd
% git checkout -b merge_result
% git merge freebsd/main
```

Typically, there would be no merge conflicts here (because developers tends to work on different components).
In the worst case scenario, you would still have to resolve merge conflicts, if there was any, but this
should be really rare.

Now, checkout `freebsd/main` again as `new_merge`, and redo the merge:

```
% git checkout -b new_merge freebsd/main
% git subtree merge  -P contrib/mtree vendor/NetBSD/mtree
```

Instead of resolving the conflicts, perform this instead:

```
% git checkout merge_result .
```

Which will overwrite the files with conflicts with the version found in `merge_result`.

Examine the tree against `merge_result` to make sure that you haven't missed deleted files:

```
% git diff merge_result
```

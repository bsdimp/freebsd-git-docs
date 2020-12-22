# How to do vendor imports

**THIS IS A DRAFT DOCUMENT AND CONTAINS SOME WORKING NOTES**

With the FreeBSD Subversion to Git conversion complete,	it's time to
look at how to do a vendor import.

Note: This document follows the convention that the `freebsd` origin
is the source of truth. If you use a different convention, replace
freebsd with your name.

There are hundreds, if not thousands of	tags related to	vendor imports
that the FreeBSD project has created over the years. To	keep all this
clutter from getting in the way, vendor	branches were imported in
their own name space, in much the same way notes are.

To import notes (helpful when you want to look up svn revisions), for
example, one would clone the repository and then do:
```
git config --add remote.freebsd.fetch '+refs/notes/*:refs/notes/*' && git fetch
```

All vendor branches and tags start with `vendor/` and are branches and
tags visible by default. Prior drafts of this document had them
directly under `refs/vendor` rather than `refs/head/vendor` so early adopters
that have the older refs may need to clean them up.

We'll explore an example for updating NetBSD's mtree that's in our
tree. The vendor branch for this is `vendor/NetBSD/mtree`.

## Updating an old vendor import

Since the trees we have in vendor branchesare usually a tiny subset of
the FreeBSD, it's best to do them with work trees since the process is
quite fast. Make sure that whatever directory you choose (the
`../mtree`) argument is empty and doesn't conflict.
```
% git worktree add ../mtree vendor/NetBSD/mtree
```
### Update the Sources in the Vendor Branch

I have my copy of NetBSD checked out from their github mirror in
`~/git/NetBSD`, so I'll update from there: Note that "upstream" might
have added or removed files, so we want to make sure deletions are
propagated as well. rsync(1) is commonly installed, so I'll use that.
```
% cd ../mtree
% rsync -va --del ~/git/NetBSD/usr.sbin/mtree/ .
% git add -A
% git status
...
% git diff --staged
...
% git commit -m"Vendor import of NetBSD's mtree at 2020-12-11"
[vendor/NetBSD/mtree 8e7aa25fcf1] Vendor import of NetBSD's mtree at 2020-12-11
 7 files changed, 114 insertions(+), 82 deletions(-)
% git tag -a vendor/NetBSD/mtree/20201211
```

Note: I used git commit -a here because I was sure I knew what I was
doing and I'd run the `git diff` and `git status` commands to make
sure nothing weird was present. Normally I only recommend using that
when you're an expert and have checked what's going on to be
sure. Also I used `-m` to illustrate, but you should compose a proper
message in an editor.

It's also important to create an annotated tag, otherwise the push
will be rejected. Only annotated tags are allowed to be pushed.

### Updating the FreeBSD Copy
Now you need to update the mtree in FreeBSD. The sources live in `contrib/mtree` since it's upstream software.

At this point you can push the updated vendor branch upstream:
```
% git push --follow-tags freebsd vendor/NetBSD/mtree
```

`--follow-tags` tells `git push` to also push tags associated with the locally committed revision.

### Updating the FreeBSD source tree

```
% cd ../src
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree
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
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree
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

## Creating a new vendor branch
There's a number of ways to create a new vendor branch. The easiest is
to create a new repository and then merge that with FreeBSD. Let's say
we're importing `glorbnitz` into the FreeBSD tree, release 3.1415. For
the sake of simplicity, we'll not trim this release. It's a user
command that puts the nitz device into different magical glorb states.

### Create the repo
```
% cd /some/where
% mkdir glorbnitz
% cd glorbnitz
% git init
% git checkout -b vendor/glorbnitz
```

At this point, you have a new repo, where all new commtis will go on
the `vendor/glorbnitz` branch.

### Copy the sources in
Since this is a new import, you can just cp the sources in, or use tar or
even rsync as shown above. And we'll add everything, assuming no dot files.
```
% cp -r ~/glorbnitz/* .
% git add *
```

At this point, you should have a pristine copy of glorbnitz ready to commit.

```
% git commit -m"Import GlorbNitz frobnosticator revision 3.1415"
```
As above, I used `-m` for simplicity, but you should likely create a
ncommit message that explains what a Glorb is and why you'd use a Nitz
to get it. Not everybody will know.

### Now import it into our repository
Now you need to import the branch into our repository.
```
% cd /path/to/freebsd/repo/src
% git remote add glorbnitz /some/where/glorbnitz
% git fetch glorbnitz vendor/glorbnitz
```
Not the vendor/glorbnitz branch is in the repo. At this point the
`/some/where/glorbnitz` can be deleted, if you like. It was only a means
to an end.

### Tag and push
Steps from here on out are much the same as they are in the case of
updating a vendor branch, though w/o the updating the vendor
branch step.
```
% git worktree add ../glorbnitz vendor/glorbnitz
% cd ../glorbnitz
% git tag --annotate vendor/glorbnitz/3.1415
# Make sure it's good
% git push --follow-tags freebsd vendor/glorbnitz
```
By 'good' we mean:
1. All the right files are present
2. None of the wrong files are present
3. The vendor branch points at something sensible
4. The tag looks good, and is annotated.

### Time to finally merge it into the base tree
```
% cd ../src
% git subtree add -P contrib/glorbnitz vendor/glorbnitz
# Make sure it's good
% git push freebsd
```
Here 'good' means:
1. All the right files, and none of the wrong ones, were merged into contrib/glorbnitz.
2. No other changes are in the tree
3. The commit messages look good.

Note: This hasn't connected `glorbnitz` to the build yet. How to do
that hasn't changed and is beyond the scope of this article.

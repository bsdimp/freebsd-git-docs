# How to MFC

** NOTE: THIS IS WORK IN PROGRESS and current incomplete **

Note: This document uses the convention where the upstream origin name
is `freebsd` as suggested in other docs.

## Summary

MFC workflow can be summarized as `git cherry-pick -x` plus git commit
--amend to adjust the commit message. For multiple commits, use `git rebase -i`
to squash them together and edit the commit message.

## Single commit MFC

```
% git checkout stable/X
% git cherry-pick -x $HASH --edit
```

For MFC commits, for example a vendor import, you would need to specify one parent for cherry-pick
purposes.  Normally, that would be the "first parent" of the branch you are cherry-picking from, so:

```
% git checkout stable/X
% git cherry-pick -x $HASH -m 1 --edit
```

If things go wrong, you'll either need to abort the cherry-pick with `git cherry-pick --abort` or fix it
up and do a `git cherry-pick --continue`.

Once the cherry-pick is finished, push with `git push`.  If you get an error due to losing the commit race,
use `git pull --rebase` and try to push again.

## Multiple commit MFC

```
% git checkout -b tmp-branch stable/X
% for h in $HASH_LIST; do git cherry-pick -x $h; done
% git rebase -i stable/X
# mark each of the commits after the first as 'squash'
# edit the commit message to be sane
% git push freebsd HEAD:stable/X
```

If the push fails due to losing the commit race, rebase and try again:

```
% git checkout stable/X
% git pull
% git checkout tmp-branch
% git rebase stable/X
% git push freebsd HEAD:stable/X
```

Once the MFC is complete, you can delete the temporary branch:

```
% git checkout stable/X
% git branch -d tmp-branch
```

## MFC a vendor import

Vendor imports are the only thing in the tree that creates a merge
commit in the main line. Cherry picking merge commits into stable/XX
presents an additional difficulty because there are two parents for a
merge commit. Generally, you'll want the first parent's diff since
that's the diff to mainline (though there may be some exceptions).

```
% git cherry-pick -x -m 1 $HASH
```
is typically what you want. This will tell cherry-pick to apply the correct diff.

There are some, hopefully, rare cases where it's possible that the
mainline was merged backwards by the conversion script. Should that be
the case (and we've not found any yet), you'd change the above to '-m 2'
to pickup the proper parent. Just do
```
% git cherry-pick --abort
% git cherry-pick -x -m 2 $HASH
```
to do that. The `--aboort` will cleanup the failed first attempt.

## Redoing a MFC

If you do a MFC, and it goes horribly wrong and you want to start over,
then the easiest way is to use `git reset --hard` like so:
```
% git reset --hard freebsd/stable/12
```
though if you have some revs you want to keep, and others you don't,
using 'git rebase -i' is better.

## Considerations when MFCing

When committing source commits to stable and releng branches, we have
the following goals:

1. Clearly mark direct commits distinct from commits that land a
   change from another branch
2. Avoid introducing known breakage into stable and releng branches
3. Allow developers to determine which changes have or have not been
   landed from one branch to another

With subversion, we used the following practices to achieve these goals:

1. Using 'MFC' and 'MFS' tags to mark commits that merged changes from
   another branch
2. Squashing fixup commits into the main commit when merging a change
3. Recording mergeinfo so that `svn mergeinfo --show-revs` worked

With Git, we will need to use different strategies to achieve the same
goals.  This document aims to define best practices when merging
source commits using git that achieve these goals.  In general, we aim
to use git's native support to achieve these goals rather than
enforcing practices built on subversion's model.

One general note: due to technical differences with Git, we will not
be using git "merge commits" (created via `git merge`) in stable or
releng branches.  Instead, when this document refers to "merge
commits", it means a commit originally made to `main` that is
replicated or "landed" to a stable branch, or a commit from a stable
branch that is replicated to a releng branch with some varation of
`git cherry-pick`.

## Commit message standards

### Marking MFCs

There are two main options for marking MFCs as distinct from direct
commits:

1. One option that matches our existing practice (the wisdom of which
   I'm not commenting on) would mark MFCs like this in the commit
   message:

```
MFC: 12def6789a3a,ac32ee4a5c2e
```

   where the first 12 digits of the hash is used to mark the commit message.
   This "abbreviated hash" can be retrieved by:

```
git show --format=%p --no-patch $full_hash
```

   This preserves the information, but isn't 'git standard'.  It also
   requires committers to manually edit commit messages to include
   this information when merging.

2. Use the `-x` flag with `git cherry-pick`.  This adds a line to the
   commit message that includes the hash of the original commit when
   merging.  Since it is added by Git directly, committers do not have
   to manually edit the commit log when merging.

We feel that the second option is simpler going forward.

### Finding Eligible Hashes to MFC

Git provides some built-in support for this via the `git cherry` and
`git log --cherry` commands.  These commands compare the raw diffs of
commits (but not other metadata such as log messages) to determine if
two commits are identical.  This works well when each commit from head
is landed as a single commit to a stable branch, but it falls over if
multiple commits from main are squashed together as a single commit to
a stable branch.

There are a few options for resolving this:

1. We could ban squashing of commits and instead require that committers
   stage all of the fixup / follow-up commits to stable into a single
   push.  This would still achieve the goal of stability in stable and
   releng branches since pushes are atomic and users doing a simple pull
   will never end up with a tree that has the main commit without the
   fixup(s).  `git bisect` is also able to cope with this model via
   `git bisect skip`.

2. We could adopt a consistent style for describing MFCs and write
   our own tooling to wrap around `git cherry` to determine the list
   of eligible commits.  A simple approach here might be to use the
   syntax from `git cherry-pick -x`, but require that a squashed
   commit list all of the hashes (one line per hash) at the end of
   the commit message.  Developers could do this by using
   `git cherry-pick -x` of each individual commit into a branch and
   then use `git rebase` to squash the commits down into a single
   commit, but collecting the `-x` annotations at the end of the
   landed commit log.

### Trim Metadata?

One area that was not clearly documented with subversion (or even CVS)
is how to format metadata in log messages for MFC commits.  Should
it include the metadata from the original commit unchanged, or should
it be altered to reflect information about the MFC commit itself?

Historical practice has varied, though some of the variance is by
field.  For example, MFCs that are relevant to a PR generally
include the PR field in the MFC so that MFC commits are included
in the bug tracker's audit trail.  Other fields are less clear.  For
example, Phabricator shows the diff of the last commit tagged to a
review, so including Phabricator URLs replaces the `main` commit with
the landed commits.  The list of reviewers is also not clear.  If a
reviewer has approved a change to `main`, does that mean they have
approved the MFC commit?  Is that true if it's identical code only,
or with merely trivial reworkes? It's clearly not true for more
extensive reworks. Even for identical code what if the commit doesn't
conflict but introduces an ABI change?  A reviewer may have ok'd a
commit for `main` due to the ABI breakage but may not approve of
merging the same commit as-is. One will have to use one's best
judgement until clear guidelines can be agreed upon.

For MFCs regulated by re@, new metadata fields are added, such as
the Approved by tag for approved commits.  This new metadata will have
to be added via `git commit --amend` or similar after the original
commit has been reviewed and approved.  We may also want to reserve
some metadata fields in MFC commits such as Phabricator URLs for use
by re@ in the future.

Preserving existing metadata provides a very simple workflow.
Developers can just use `git cherry-pick -x` without having to edit
the log message.

If instead we choose to adjust metadata in MFCs, developers will
have to edit log messages explicitly via the use of `git cherry-pick
--edit` or `git commit --amend`.  However, as compared to svn, at
least the existing commit message can be pre-populated and metadata
fields can be added or removed without having to re-enter the entire
commit message.

The bottom line is that developers will likely need to curate their
commit message for MFCs that are non-trivial.

## Looking for things to MFC

If you are looking for changes to MFC, the following may help:
```
% git log --cherry stable/12 main -- bin/ls
```

## Scripts

We currently have no scripts.

## Examples

### Merging a Single Subversion Commit

This walks through the process of merging a commit to stable/12 that
was originally committed to head in Subversion.  In this case, the
original commit is r368685.

The first step is to map the Subversion commit to a Git hash.  Once
you have fetched refs/notes/commits, you can pass the revision number
to `git log --grep`:

```
> git log main --grep 368685
commit ce8395ecfda2c8e332a2adf9a9432c2e7f35ea81
Author: John Baldwin <jhb@FreeBSD.org>
Date:   Wed Dec 16 00:11:30 2020 +0000

    Use the 't' modifier to print a ptrdiff_t.
    
    Reviewed by:    imp
    Obtained from:  CheriBSD
    Sponsored by:   DARPA
    Differential Revision:  https://reviews.freebsd.org/D27576

Notes:
    svn path=/head/; revision=368685
```

Next, MFC the commit to a `stable/12` checkout:

```
git checkout stable/12
git cherry-pick -x ce8395ecfda2c8e332a2adf9a9432c2e7f35ea81 --edit
```

Git will invoke the editor.  Use this to remove the metadata that only
applied to the original commit (Phabricator URL and Reviewed by).
After the editor saves the updated log message, Git completes the
commit:

```
[stable/12 3e3a548c4874] Use the 't' modifier to print a ptrdiff_t.
 Date: Wed Dec 16 00:11:30 2020 +0000
 1 file changed, 1 insertion(+), 1 deletion(-)
```

The contents of the MFCd commit can be examined via `git show`:

```
> git show
commit 3e3a548c487450825679e4bd63d8d1a67fd8bd2d (HEAD -> stable/12)
Author: John Baldwin <jhb@FreeBSD.org>
Date:   Wed Dec 16 00:11:30 2020 +0000

    Use the 't' modifier to print a ptrdiff_t.
    
    Obtained from:  CheriBSD
    Sponsored by:   DARPA
    
    (cherry picked from commit ce8395ecfda2c8e332a2adf9a9432c2e7f35ea81)

diff --git a/sys/compat/linuxkpi/common/include/linux/printk.h b/sys/compat/linuxkpi/common/include/linux/printk.h
index 31802bdd2c99..e6510e9e9834 100644
--- a/sys/compat/linuxkpi/common/include/linux/printk.h
+++ b/sys/compat/linuxkpi/common/include/linux/printk.h
@@ -68,7 +68,7 @@ print_hex_dump(const char *level, const char *prefix_str,
                        printf("[%p] ", buf);
                        break;
                case DUMP_PREFIX_OFFSET:
-                       printf("[%p] ", (const char *)((const char *)buf -
+                       printf("[%#tx] ", ((const char *)buf -
                            (const char *)buf_old));
                        break;
                default:
```

The MFC commit can now be published via `git push`

```
git push freebsd
Enumerating objects: 17, done.
Counting objects: 100% (17/17), done.
Delta compression using up to 4 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (9/9), 817 bytes | 204.00 KiB/s, done.
Total 9 (delta 5), reused 1 (delta 1), pack-reused 0
To gitrepo-dev.FreeBSD.org:src.git
   525bd9c9dda7..3e3a548c4874  stable/12 -> stable/12
```

### Merging a Single Subversion Commit with a Conflict

This example is similar to the previous example except that the
commit in question encounters a merge conflict.  In this case, the
original commit is r368314.

As above, the first step is to map the Subversion commit to a Git
hash:

```
> git log main --grep 368314
commit 99963f5343a017e934e4d8ea2371a86789a46ff9
Author: John Baldwin <jhb@FreeBSD.org>
Date:   Thu Dec 3 22:01:13 2020 +0000

    Don't transmit mbufs that aren't yet ready on TOE sockets.
    
    This includes mbufs waiting for data from sendfile() I/O requests, or
    mbufs awaiting encryption for KTLS.
    
    Reviewed by:    np
    MFC after:      2 weeks
    Sponsored by:   Chelsio Communications
    Differential Revision:  https://reviews.freebsd.org/D27469

Notes:
    svn path=/head/; revision=368314
```

Next, MFC the commit to a `stable/12` checkout:

```
git checkout stable/12
git cherry-pick -x 99963f5343a017e934e4d8ea2371a86789a46ff9 --edit
Auto-merging sys/dev/cxgbe/tom/t4_cpl_io.c
CONFLICT (content): Merge conflict in sys/dev/cxgbe/tom/t4_cpl_io.c
warning: inexact rename detection was skipped due to too many files.
warning: you may want to set your merge.renamelimit variable to at least 7123 and retry the command.
error: could not apply 99963f5343a0... Don't transmit mbufs that aren't yet ready on TOE sockets.
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

In this case, the commit encountered a merge conflict in
sys/dev/cxge/tom/t4_cpl_io.c as kernel TLS is not present in
stable/12.  Note that Git does not invoke an editor to adjust the
commit message due to the conflict.  `git status` confirms that this
file has merge conflicts:

```
> git status
On branch stable/12
Your branch is up to date with 'upstream/stable/12'.

You are currently cherry-picking commit 99963f5343a0.
  (fix conflicts and run "git cherry-pick --continue")
  (use "git cherry-pick --skip" to skip this patch)
  (use "git cherry-pick --abort" to cancel the cherry-pick operation)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   sys/dev/cxgbe/tom/t4_cpl_io.c

no changes added to commit (use "git add" and/or "git commit -a")
```

After editing the file to resolve the conflict, `git status` shows the
conflict as resolved:

```
> git status
On branch stable/12
Your branch is up to date with 'upstream/stable/12'.

You are currently cherry-picking commit 99963f5343a0.
  (all conflicts fixed: run "git cherry-pick --continue")
  (use "git cherry-pick --skip" to skip this patch)
  (use "git cherry-pick --abort" to cancel the cherry-pick operation)

Changes to be committed:
        modified:   sys/dev/cxgbe/tom/t4_cpl_io.c
```

The cherry-pick can now be completed:

```
> git cherry-pick --continue
```

Since there was a merge conflict, Git invokes the editor to
adjust the commit message.  Trim the metadata fields from the
commit log from the original commit to head and save the
updated log message.

The contents of the MFC commit can be examined via `git show`:

```
> git show
commit 525bd9c9dda7e7c7efad2d4570c7fd8e1a8ffabc (HEAD -> stable/12)
Author: John Baldwin <jhb@FreeBSD.org>
Date:   Thu Dec 3 22:01:13 2020 +0000

    Don't transmit mbufs that aren't yet ready on TOE sockets.
    
    This includes mbufs waiting for data from sendfile() I/O requests, or
    mbufs awaiting encryption for KTLS.
    
    Sponsored by:   Chelsio Communications
    
    (cherry picked from commit 99963f5343a017e934e4d8ea2371a86789a46ff9)

diff --git a/sys/dev/cxgbe/tom/t4_cpl_io.c b/sys/dev/cxgbe/tom/t4_cpl_io.c
index 8e8c2b8639e6..43861f10b689 100644
--- a/sys/dev/cxgbe/tom/t4_cpl_io.c
+++ b/sys/dev/cxgbe/tom/t4_cpl_io.c
@@ -746,6 +746,8 @@ t4_push_frames(struct adapter *sc, struct toepcb *toep, int drop)
                for (m = sndptr; m != NULL; m = m->m_next) {
                        int n;
 
+                       if ((m->m_flags & M_NOTAVAIL) != 0)
+                               break;
                        if (IS_AIOTX_MBUF(m))
                                n = sglist_count_vmpages(aiotx_mbuf_pages(m),
                                    aiotx_mbuf_pgoff(m), m->m_len);
@@ -821,8 +823,9 @@ t4_push_frames(struct adapter *sc, struct toepcb *toep, int drop)
 
                /* nothing to send */
                if (plen == 0) {
-                       KASSERT(m == NULL,
-                           ("%s: nothing to send, but m != NULL", __func__));
+                       KASSERT(m == NULL || (m->m_flags & M_NOTAVAIL) != 0,
+                           ("%s: nothing to send, but m != NULL is ready",
+                           __func__));
                        break;
                }
 
@@ -910,7 +913,7 @@ t4_push_frames(struct adapter *sc, struct toepcb *toep, int drop)
                toep->txsd_avail--;
 
                t4_l2t_send(sc, wr, toep->l2te);
-       } while (m != NULL);
+       } while (m != NULL && (m->m_flags & M_NOTAVAIL) == 0);
 
        /* Send a FIN if requested, but only if there's no more data to send */
        if (m == NULL && toep->flags & TPF_SEND_FIN)
```

The MFC commit can now be published via `git push`

```
git push freebsd
Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Delta compression using up to 4 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (7/7), 819 bytes | 117.00 KiB/s, done.
Total 7 (delta 6), reused 0 (delta 0), pack-reused 0
To gitrepo.FreeBSD.org:src.git
   f4d0bc6aa6b9..525bd9c9dda7  stable/12 -> stable/12
```

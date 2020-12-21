# How to MFC

** NOTE: THIS IS WORK IN PROGRESS and current woefully incomplete **

Note: This document uses the convention where the upstream origin name
is `freebsd` as suggested in other docs.

## Commit message standards

### Marking Merges

As with subversion, we wish to mark commits to stable branches that
are merges from main distinctly from direct commits.  There are two
main options:

1. One option that matches our existing practice (the wisdom of which
   I'm not commenting on) would mark MFCs like this in the commit
   message:

```
MFC: 12def6789a,ac32ee4a5c
```

   where the first 10 digits of the hash (might be longer in a large repository)
   is used to mark the commit message. This "abbreviated hash" can be retrieved by:

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

### Finding Eligible Merges

One feature some developers have found happy with subversion is
determining which commits have or have not been merged (for example,
`svn mergeinfo --show-revs eligible`).

Git provides some built-in support for this via the `git cherry` and
`git log --cherry` commands.  These commands compare the raw diffs of
commits (but not other metadata such as log messages) to determine if
two commits are identical.  This works well when each commit from head
is merged as a single commit to a stable branch, but it falls over if
multiple commits from main are squashed together as a single commit to
a stable branch.

There are a few options for resolving this:

1. We could ban squashing of commits and instead require that committers
   stage all of the fixup / follow-up commits to stable into a single
   push.

2. We could adopt a consistent style for describing merges and write
   our own tooling to wrap around `git cherry` to determine the list
   of eligible commits.  A simple approach here might be to use the
   syntax from `git cherry-pick -x`, but require that a squashed
   commit list all of the hashes (one line per hash) at the end of
   the commit message.  Developers could do this by using
   `git cherry-pick -x` of each individual commit into a branch and
   then use `git rebase` to squash the commits down into a single
   commit, but collecting the `-x` annotations at the end of the
   merged commit log.

### Trim Metadata

`git cherry-pick` will copy the entire log message of the original
commit as the log message for the merge to stable.  This is useful,
but metadata fields that only apply to the head commit should be
removed or updated for the MFC.  For example, phabriactor URLs or
reviewers should be removed.  Metadata should only be included if it
is relevant to the merge operation (for example, if a developer
reviews the merge prior to commit, or if re@ approves a merge to a
stable branch).  Using `--edit` with `git cherry-pick` allows the
commit message to be fixed to remove incorrect metadata.  The commit
log can always be updated prior to pushing via `git commit --amend` or
the `reword` action in `git rebase -i`.

## Single commit MFC

```
% git checkout stable/X
% git cherry-pick -x $HASH --edit
```

For merge commits, for example a vendor import, you would need to specify one parent for cherry-pick
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

Once the merge is complete, you can delete the temporary branch:

```
% git checkout stable/X
% git branch -d tmp-branch
```

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
original commit is r368314.

The first step is to map the Subversion commit to a Git hash.  Once
you have fetched refs/notes/commits, you can pass the revision number
to `git log --grep`:

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

Next, merge the commit to a `stable/12` checkout:

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
stable/12.  `git status` confirms that this file has merge conflicts:

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

The contents of the merged commit can be examined via `git show`:

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

The merged commit can now be published via `git push`

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

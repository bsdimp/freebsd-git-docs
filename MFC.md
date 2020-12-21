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

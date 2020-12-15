# How to MFC

** NOTE: THIS IS WORK IN PROGRESS and current woefully incomplete **

## Commit message standards

Git gives some help for committing cherry picks and keeping track. Git computes a 'diff index' for each commit that's applied and uses that to detect when it
doesn't need to include the patch in a rebase, or as part of git log --cherry described below.

This works gret for single commits, but the project tradition is to squash multiple changes.
This defeats the diff index that git does. So, we'd need to mark them somehow. Keeping with
the traditional way we've marked commits (the wisdom of which I'm not commenting on), we'd
want to have MFCs marked like this in the commit message.
```
MFC: 12def6789a,ac32ee4a5c
```
where the first 10 digits of the hash is used to mark the commit message. This preserves the information,
but isn't 'git standard' which is just to have a line of text that says the commit was from hash
whatever without the 'key: value' structure. it also wouldn't help the `git log --cherry` command, unless
we would use that to come up with candiates, and then use the MFC lines to weed them out.
Altnernatively, adding a note either to the src (saying where it had been merged to) or the destination
(saying where it had been merged from) would allow for easier parsing and after-the-fact fixups
similar to svn merge --record-only.

There are also advantages to all the commits on stable/X having something to identify them as
being merge commits (so you can tell if you inadvertently reverse args so you got commits to stable/12
instead of commits on main).

## Single commit MFC

```
% git checkout stable/X
% git cherry-pick -x $HASH --edit
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
% git push origin HEAD:stable/X
```

If the push fails due to losing the commit race, rebase and try again:

```
% git checkout stable/X
% git pull
% git checkout tmp-branch
% git rebase stable/X
% git push origin HEAD:stable/X
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

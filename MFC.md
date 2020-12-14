# How to MFC

** NOTE: THIS IS WORK IN PROGRESS and current woefully incomplete **

## Single commit MFC

```
% git checkout stable/X
% git cherry-pick -x $HASH
```

## Multiple commit MFC

```
% git checkout stable/X
% git checkout -b tmp-branch
% for h in $HASH_LIST; do git cherry-pick -x $h; done
% git reabse -i stable/X
# mark each of the commits after the first as 'squash'
# edit the commit message to be sane
% git checkout stable/X
% git merge --ff-only tmp-branch
% git branch -d tmp-branch
```

## Looking for things to MFC

git log --cherry-pick

## Scripts

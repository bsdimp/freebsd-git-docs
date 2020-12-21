# Github mirroring plan

## FreeBSD History

The FreeBSD project is currently mirroring to github an experimental
tree. Due to the large number of bugs that adversely affects the
quality of the tree, and its day to day use for anything more
complicated that merging and commits and simple history browsing, the
move to git project opted to redo the hashes and fix many historical
mistakes as part of its effort. As such, a migration plan is needed,
so this describes it.

## The Plan for src

1. fork the 'beta hashes' repo on github and call it freebsd-legacy
2. push the new repo, the stable/*, releng/* and release/* branches/tags
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


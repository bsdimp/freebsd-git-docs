# Design notes

I've abstracted the README from git_conv here. It was written by
uqs@. This is still quite rough, but contains useful information, even
in its currently lightly-edited form from there. More revision will be
forthcoming.

## Gimme the repo!

```
git clone https://cgit-beta.freebsd.org/src.git && cd src
git config --add remote.freebsd.fetch '+refs/notes/*:refs/notes/*' && git fetch
```

Same for the `doc` and `ports` repos. You can expect some things to be
different. Most importantly you should check whether branch and mergepoints
(especially for vendor branches, but also project branches) are there and make
sense.

Neither `vendor`, `user` or `projects` branches are "visible" by default.
`backups` refs are deleted branches and `cvs2svn` contains some of the detritus
left over from the CVS days and the cvs2svn conversion. The `internal`
namespace for now only has the `access` and `mentors` file, detailing when
people got their various commit bits.

```
git config --add remote.freebsd.fetch '+refs/vendor/*:refs/vendor/*'
git config --add remote.freebsd.fetch '+refs/projects/*:refs/projects/*'
git config --add remote.freebsd.fetch '+refs/user/*:refs/user/*'
git config --add remote.freebsd.fetch '+refs/backups/*:refs/backups/*'
git config --add remote.freebsd.fetch '+refs/cvs2svn/*:refs/cvs2svn/*'
git config --add remote.freebsd.fetch '+refs/internal/*:refs/internal/*'
git fetch
```

Note that `internal`, `projects` and `user` branches also exist for the `doc`
repo and `ports` has `projects` and `releng` as well. vendor, cvs2svn and
backups are exclusive to the `src` repo though.

- `user/` branches are never merged back into `master`
- MFHs into `user/` or `projects/` branches are just cherry-picks to keep `git
  log --graph` somewhat readable and as merges wouldn't convey any useful
  information, really.
- `vendor` **tags** were never flattened post-creation, as that would advance
  them off of the mainline branch and make them invisible to a simple `git log`
- `release/1.0_cvs` et al. are snapshots of the checked out CVS source
  code, including expanded $Id$ tags.
- no other keywords are expanded
- various vendor-foo suffixes have been collapsed into 1 vendor namespace,
  except for a few vendors where merging the userland and kernel bits is not
  straightforward due to how they interleave with the merge and branch history.
- some branches have their history "extended", that is, commits under the
  `cvs2svn` area were properly attached.
- ... and most of these commits have actually been inlined directly into the
  mainline tree to keep the history more "linear" and associate the commit with
  the original author and commit message.

## How to analyze the results

Here are some tips that were deemed useful in making sense of resulting
history. Please add a "graph notes log" alias to your `.gitconfig`:
```
[alias]
  gnlog = log --graph --pretty=format:'%Cred%h %C(green)%t %Creset %C(red)%ad %Creset-%C(yellow)%d%Creset %s %n      %N %-GG' --date=short
```

### Show the full tree of a vendor area

```
git gnlog `git show-ref|grep vendor/sendmail|cut -d" " -f1`
```

### Find a certain SVN revision on master

```
git show -p `git log --format=%h --notes --grep=revision=294706$`
```

### Show how/where/when a vendor branch was merged into master over time

```
git gnlog vendor/zstd/dist master
```
(but you'll need to search in the massive output for where the vendor branch is
being merged, if you know a better way to represent this, please let us know!)

### Look for commits with more than 5 parents and log them

```
git log --format='%H %P' --all|awk '{if (NF > 5) { print NF " " $0}}'|sort -rn|cut -d" " -f2|xargs -n1 -I% git snlog -n 1 %
```

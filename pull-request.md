# How to deal with Pull Requests

Various Git collaboration tools allow random contributors to send us pull
requests (PR, yes, unfortunately the same abbreviation as Problem Report). With
Git, these are comparatively easy to pull down, inspect and test, as well as
rebase onto our `main` branch and push to our source of truth.

However, we cannot use the integrated 1-click buttons on these tools, as we see
them as read-only mirrors and the pushes need to go directly to FreeBSD's
infrastructure instead.

Note that we disallow merges (except for vendor merges) so we want a "linear
history" and need to do the equivalent of a "Rebase and merge".

# GitHub

As we had the legacy `master` branch on GitHub for quite some time, there are
old PRs that are based on them. If you pull down anything before PR #449,
you'll likely get a commit that's based on `master` instead of `main`.

Add GitHub as a remote (only showing -src here), add a refspec to pull down
all pull requests, fetch them, and clean up after our "legacy" master
conversion. This is a one-time step and only needed for GitHub.
```
% git remote add github https://github.com/freebsd/freebsd-src
% git config --add remote.github.fetch "+refs/pull/*:refs/remotes/github/pull/*"
% git fetch github && git gc
```

Or fetch a very specific PR, only.
```
% git fetch github "refs/pull/422/*:refs/remotes/github/pull/422/*"
remote: Enumerating objects: 19, done.
remote: Counting objects: 100% (19/19), done.
remote: Total 23 (delta 19), reused 19 (delta 19), pack-reused 4
Unpacking objects: 100% (23/23), 6.23 KiB | 102.00 KiB/s, done.
From github.com:freebsd/freebsd-src
 * [new ref]                   refs/pull/422/head  -> github/pull/422/head
 * [new ref]                   refs/pull/422/merge -> github/pull/422/merge
% git show-ref | grep /pull/
63c82ec5c3833b50e45c31c59a04d7f9a827abb8 refs/remotes/github/pull/422/head
1d1d776bd0f036130446f3d331aef3dd131ef8a3 refs/remotes/github/pull/422/merge
```

We can ignore the merge head, as we want to pick the (potentially many) commits
off of the head and rebase them onto our `main`.

# GitLab et al.

```
% git remote add gitlab https://gitlab.com/FreeBSD/freebsd-src.git
% git config --add remote.gitlab.fetch "+refs/merge-requests/*:refs/remotes/gitlab/merge-requests/*"
% git fetch gitlab
```

Adapt this to wherever you got the pull or merge request from.

# Common ways to integrate the Pull Request

We'll take the GitHub example, but GitLab and co. will be just the same.

Run `git log github/pull/422/head` and you see 3 commits by a contributor,
followed by commits that all come from "foo <foo@FreeBSD.org>". This is an
indication that it's on the legacy `master` branch.

While you can use `rebase A..B --onto C`, yours truly is often confused by the
ordering of the arguments and a series of cherry-picks is likely easier. So in
this case, let's create a new branch and see if stuff applies cleanly on top of `main`.

```
% git checkout -b pull_422 freebsd/main
% git cherry-pick github/pull/422/head~2
Auto-merging sys/sys/errno.h
Auto-merging sys/kern/imgact_elf.c
[pull_422 df45283b47f9] * Remove redundant code in __elfN(map_insert) * Do more check on the executable file. * Add a new error return constant ESUCCESS to make the return code more clear.
 Author: Wuyang Chung <wuyang.chung1@gmail.com>
 Date: Wed Jan 22 19:05:14 2020 +0800
 2 files changed, 36 insertions(+), 99 deletions(-)
% git cherry-pick github/pull/422/head~1
Auto-merging sys/kern/imgact_elf.c
[pull_422 07bb52c328c8] Modify comment.
 Author: Wuyang Chung <wuyang.chung1@gmail.com>
 Date: Wed Jan 22 19:10:33 2020 +0800
 1 file changed, 1 insertion(+), 1 deletion(-)
% git cherry-pick github/pull/422/head
Auto-merging sys/sys/errno.h
[pull_422 0a27e19be827] Add #ifndef _POSIX_SOURCE around ESUCCESS.
 Author: Wuyang Chung <wuyang.chung1@gmail.com>
 Date: Fri Jan 24 16:31:41 2020 +0800
 1 file changed, 2 insertions(+)

```

## Massaging the commit history and content

Inspect the commits with `git log -p --format=fuller`, note that Author is the
contributor and Committer will be you. AuthorDate will also be way in the past,
this is fine. If the Author fields contains egregious swear words or the like,
maybe get them to change this.

In our case, there's a trailing whitespace introduced by the commit, something
you might want to fix and `git commit --amend` or use `git rebase -i` to squash
things. Especially since there are 2 trivial changes in a commit and an ifdef
wrapper, this is a case of squashing things down into 1 commit. There are
multiple ways to do this.

```
% git rebase -i HEAD~10
<squash the last 2 commits into the 3rd-last commit, edit the commit message to something useful>
% vim sys/kern/imgact_elf.c  <get rid of the trailing whitespace here>
% git commit --amend sys/kern/imgact_elf.c
```

You should amend the commit message with a pointer to the review that might've
happened on the website, similar to phabricator. As a simple pull request
number is ambiguous, please use the full URL.

```
Pull Request:	https://github.com/freebsd/freebsd-src/pull/422
```

## Double check your work

We now have a single commit (without trailing whitespace), that looks like so:

```
% git log -n1 --format=fuller
commit 61ea2b75b055bd7ef6edf0224adefaf457ce4c4a (HEAD -> pull_422)
Author:     Wuyang Chung <wuyang.chung1@gmail.com>
AuthorDate: 2020-01-22 19:05:14 +0800
Commit:     Ulrich Spörlein <uqs@FreeBSD.org>
CommitDate: 2021-01-06 14:14:38 +0100

    * Remove redundant code in __elfN(map_insert)
    * Do more check on the executable file.
    * Add a new error return constant ESUCCESS to make the return code more clear.
    
    Also add #ifndef _POSIX_SOURCE around ESUCCESS.
<let's also double check whether we did some merge commit oopsie>
% git log --graph --oneline
```

## Pushing upstream

If that passes all the build and runtime checks, it can be pushed to FreeBSD,
but since time has passed, we first need to rebase onto `origin/main` again. As
we branched off of `origin/main`, this is a simple `git pull --rebase` away
(pull always implies fetching first).


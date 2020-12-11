# Git FAQ

This document provides a number of targeted answers to questions that
are likely to come up often for users, developer and integrators.

## Users

### How do I track -current and -stable with only one copy of the repo?

**Q:** Although disk space is not a huge issue, it's more efficient to use
only one copy of the repository. With SVN mirroring, I could checkout
multiple trees from the same repo. How do I do this with Git?

**A:** You can use Git worktrees. There's a number of ways to do this,
but the simplest way is to do a clone to track -current, and a
worktree to track stable releases. While using a 'bare repository'
has been put forward as a way to cope, it's more complicated and will not
be documented here.

First, you need to clone the FreeBSD repository, shown here cloning into
`freebsd-current` to reduce confusion. $URL is whatever mirror works
best for you:
```
% git clone $URL freebsd-current
```
then once that's cloned, you can simply create a worktree from it:
```
% cd freebsd-current
% git worktree add ../freebsd-stable-12 stable/12
```
this will checkout `stable/12` into a directory named `freebsd-stable-12`
that's a peer to the `freebsd-current` directory. Once created, it's updated
very similar to how you might expect:
```
% cd freebsd-current
% git checkout main
% git pull --ff-only
# changes from upstream now local and current tree updated
% cd ../freebsd-stable-12
% git merge --ff-only origin/stable/12
# now your stable/12 is up to date too
```
I recommend using `--ff-only` because it's safer and you avoid
accidentally getting into a 'merge nightmare' where you have an extra
change in your tree, forcing a complicated merge rather than a simple
one.

## Developers

### Ooops! I committed to `main` instead of a branch.

**Q:** From time to time, I goof up and commit to main instead of to a
branch. What do I do?

**A:** First, don't panic.

Second, don't push. In fact, you can fix almost anything if you
haven't pushed. All the answers in this section assume no push
has happened.

The following answer assumes you committed to `main` and want to
create a branch called `issue`:
```
% git checkout -b issue     # Create the issue branch
% git checkout main         # Go back to main branch
% git reset --hard HEAD^    # Reset what main references
% git checkout issue        # Back to where you were
```
The above assumes one commit, hence the `HEAD^`. You can put anything
there, but `HEAD^` or `HEAD^^` are the most typical.

### Ooops! I committed something to the wrong branch!

**Q:** I was working on feature on the `wilma` branch, but
accidentally committed a change relevant to the `fred` branch
to that branch. What do I do?

**A:** The answer is similar to the previous one, but with
cherry picking. This assumes there's only one commit on wilma,
but will generalize to more complicated situations. It also
assumes that it's the last commit on wilma (hence using wilma
in the `git cherry-pick` command), but that too can be generalized.

```
# We're on branch wilma
% git checkout fred		# move to fred branch
% git cherry-pick wilma		# pull in wrong commit
% git checkout wilma		# go back to wilma branch
% git reset --hard HEAD^	# move what wilma refers to back 1 commit
```
Git experts would first rewind the wilma branch by 1 commit, switch over to
fred and then use `git reflog` to see what that 1 deleted commit was and
cherry-pick it over.

**Q:** But what if I want to commit a few changes to `main`, but
keep the rest in `wilma` for some reason?

**A:** The same technique above also works if you are wanting to
'land' parts of the branch you are working on into `main` before the
rest of the branch is ready (say you noticed an unrelated typo, or
fixed an incidental bug). You can cherry pick those changes into main,
then push to the parent repo. Once you've done that, cleanup couldn't
be simpler: just `git rebase -i` since Git will notice you've done
this and omit the common changes automatically (even if you had to
change the commit message or tweak the commit slightly). There's no
need to switch back to wilma to adjust it: just rebase!

**Q:** I want to split off some chagnes from branch `wilma` into branch `fred`

**A:** The more general answer would be the same as the
previous. You'd checkout/create the `fred` branch, cherry pick the
changes you want from `wilma` one at a time, then rebase `wilma` to
remove those changes you cherry picked. `git rebase -i main wilma`
will toss you into an editor, and remove the `pick` lines that
correspond to the hashes you committed to `fred`. If all goes well,
and there are no conflicts, you're done. If not, you'll need to
resolve the conflicts as you go.

The other way to do this would be to checkout `wilma` and then create
the branch `fred` to point to the same point in the tree. You can then
`git rebase -i` both these branches, selecting the changes you want in
`fred` or `wilma` by retaining the pick likes, and deleting the rest
from the editor. Some people would create a tag/branch called
`pre-split` before starting in case something goes wrong in the split,
you can undo it with the following sequence:
```
% git checkout pre-split	# Go back
% git branch -D fred		# delete the fred branch
% git checkout -B wilma		# reset the wilma branch
% git branch -d pre-split	# Pretend it didn't happen
```
the last step is optional. If you are going to try again to
split, you'd omit it.

**Q:** But I did things as I read along and didn't see your advice at
the end to create a branch, and now `fred` and `wilma` are all
screwed up. How do I find what `wilma` was before I started. I don't
know how many times I moved things around.

**A:** All is not lost. You can figure out it, so long as it hasn't
been too long, or too many commits (hundreds).

So I created a wilma branch and committed a couple of things to it, then
decided I wanted to split it into fred and wilma. Nothing weird
happened when I did that, but lot's say it did. The way to look at
what you've done is with the `git reflog`:
```
% git reflog
6ff9c25 (HEAD -> wilma) HEAD@{0}: rebase -i (finish): returning to refs/heads/wilma
6ff9c25 (HEAD -> wilma) HEAD@{1}: rebase -i (start): checkout main
869cbd3 HEAD@{2}: rebase -i (start): checkout wilma
a6a5094 (fred) HEAD@{3}: rebase -i (finish): returning to refs/heads/fred
a6a5094 (fred) HEAD@{4}: rebase -i (pick): Encourage contributions
1ccd109 (origin/main, main) HEAD@{5}: rebase -i (start): checkout main
869cbd3 HEAD@{6}: rebase -i (start): checkout fred
869cbd3 HEAD@{7}: checkout: moving from wilma to fred
869cbd3 HEAD@{8}: commit: Encourage contributions
...
%
```

Here we see the changes I've made. You can use it to figure out where
things when wrong. I'll just point out a few things here. The first
one is that HEAD@{X} is a 'commitish' thing, so you can use that as an
argument to a command. Though if that command commits anything to the
repo, the X numbers change. You can also use the hash (first column)
as well.

Next 'Encourage contributions' was the last commit I did to `wilma`
before I decided to split things up. You can also see the same hash is
there when I created the `fred` branch to do that. I started by
rebasing `fred` and you see the 'start', each step, and the 'finish'
for that process. While we don't need it here, you can figure out
exactly what happened.  Fortunately, to fix this, you can follow the
prior answer's steps, but with the hash `869cbd3` instead of
`pre-split`. While that set of a bit verbose, it's easy to remember
since you're doing one thing at a time. You can also stack:
```
% git checkout -B wilma 869cbd3
% git branch -D fred
```
and you are ready to try again. The checkout -B with the hash combines
checking out and creating a branch for it. The -B instead of -b forces
the movement of a pre-existing branch. Either way works, which is what's
great (and awful) about Git. One reason I tend to use `git checkout -B xxxx hash`
instead of checking out the hash, and then creating / moving the branch
is purely to avoid the slightly distressing message about detached heads:
```
% git checkout 869cbd3
M	faq.md
Note: checking out '869cbd3'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 869cbd3 Encourage contributions
% git checkout -B wilma
```
this produces the same effect, but I have to read a lot more and severed heads
aren't an image I like to contemplate.

### Ooops! I did a 'git pull' and it created a merge commit, what do I do?

**Q:** I was on autopilot and did a 'git pull' for my development tree and
that created a merge commit on the mainline. How do I recover?

**A:** This can happen when you have changes in your tree that aren't properly on
a branch when you do the pull.

There's two phases for recovering. The first is to figure out what the changes were
that were on the main branch. The second is recovering.

As we discovered above, `git reflog` will help you sort out what at least the hash
was before the change. Once you have that, you can find what changes were present
that caused the disruption. At this point, you should drop a branch to that change
so you don't have to deal with hashes too much:
```
% git checkout -B tmp-branch $HASH
```
where $HASH is the hash in question (it's almost always one of the hashes of the merge
commit that's created to bring the changes in, so `git log` would have told you
the hash as well).

Now that you have a temporary tag, you can fix main branch by forcing the local
main branch to match the remove main branch:
```
% git checkout -B main origin/main
```

You can now use tmp-branch created above to pull in changes from there
to a proper branch off of main for easier rebasing. If it's one or
two changes, you may be able to do this with a
```
% git rebase -i main tmp-branch
```
since that will look for the all the changes from the `tmp-branch` after it
branched from `main`. Since the most common cause of this issue is
```
% git commit ...
% git commit ...
% git commit ...
% git pull
```
Those 3 commits will be relative to `main` and can easily be rebased to the
recreated 'main' branch above (and since we gave the branch a name, they are on a proper branch)

**Q:** But I hate the name `tmp-branch`. Can I change it easily?
**A:** Yes.
```
% git checkout tmp-branch
% git checkout -b good-name
% git branch -d tmp-branch
```
is what I do, though I have a feeling git has an obscure, one step command to do it too.
There's nothing magical about branches in git: they are just labels on a DAG that can be moved
forward by a commit. So the above works because you're just swapping out a label. There's no
metadata about the branch that needs to be preserved due to this.

Get both of them in the same repo.

### Mixing and matching branches

**Q:** So I have two branches `worker` and `async` that I'd like to combine into one branch called `feature`
while maintaining the commits in both.

**A:** This is a job for cherry pick.

```
% git checkout worker
% git checkout -b feature	# create a new branch
% git cherry-pick main..async	# bring in the changes
```
You now have one branch called `feature`. This branch combines commits
from both branches. You can further curate it with `git rebase`.

**Q:** OK Wise Guy. That was too easy. I have a branch called `driver` and I'd like
to break it up into `kernel` and `userland` so I can evolve them separately and commit
each branch as it becomes ready.

**A:** This takes a little bit of prep work, but `git rebase` will do the heavy
lifting here.

```
% git checkout driver		# Checkout the driver
% git checkout -b kernel	# Create kernel branch
% git checkout -b userland	# Create userland branch
```
Now you have two identical branches. So, it's time to separate out the commits.
We'll assume first that all the commits in `driver` go into either the `kernel`
or the `userland` branch, but not both.

```
% git rebase -i main kernel
```
and just include the changes you want (with a 'p' or 'pick' line) and
just delete the commits you don't (this sounds scary, but if worse
comes to worse, you can throw this all away and start over with the
`driver` branch since you've not yet moved it).

```
% git rebase -i main userland
```
and do the same thing you did with the `kernel` branch.

**Q:** Oh great! I followed the above and forgot a commit in the `kernel` branch.
How do I recover?

**A:** You can use the `driver` branch to find the hash of the commit is missing and
cherry pick it.
```
% git checkout kernel
% git log driver
% git cherry-pick $HASH
```

**Q:** OK. I have the same situation as the above, but my commits are all mixed up. I need
parts of one commit to go to one branch and the rest to go to the other. In fact, I have
several. Your rebase method to select sounds tricky.

**A:** In this situation, you'd be better off to curate the original branch to separate
out the commits, and then use the above method to split the branch.

So let's assume that there's just one commit with a clean tree. You
can either use `git rebase` with an `edit` line, or you can use this
with the commit on the tip. The steps are the same either way. The
first thing we need to do is to back up one commit while leaving the
changes uncommitted in the tree:
```
% git reset HEAD^
```
Note: Do not, repeat do not, add --hard here since that also removes the changes from your tree.

Now, if you are lucky, the change needing to be split up falls entirely along file lines. In that
case you can just do the usual `git add` for the files in each group than do a `git commit`. Note:
when you do this, you'll lose the commit message when you do the reset, so if you need it for
some reason, you should save a copy (though `git log $HASH` can recover it).

If you are not lucky, you'll need to split apart files. There's another tool to do that which you
can apply one file at a time.
```
git add -i foo/bar.c
```
will step through the diffs, prompting you, one at time, whether to include or exclude the hunk.
Once you're done, `git commit` and you'll have the remainder in your tree. You can run it
multiple times as well, and even over multiple files (though I find it easier to do one file at a time
and use the `git rebase -i` to fold the related commits together).

## Cloning and Mirroring

**Q:** I'd like to mirror the entire git repo, how do I do that?

**A:** If all you want to do is mirror, then
```
% git clone --mirror https://git.freebsd.org/ports.git
```
will do the trick. However, there's two disadvantages for this if you
want to use it for anything other than a mirror you'll reclone.

First, this is a 'naked repo' which has the repository files, but no
checked out copy. This is great for mirroring, but terrible for day to
day work. There's a number of ways around this with 'git subtree', but
if you aren't using it for further local clones, then it's a poor match.

Second, the git normally rewrites the refs (branch name, tags, etc)
from upstream so that your local refs can evolve independently of
upstream. This means that you'll lose changes if you are committing to
this repo on anything other than private project branches.

So to do the worktree thing,
```
% git clone --mirror https://cgit-beta.freebsd.org/ports.git ports.git
% cd ports.git
% git worktree add ../ports main
% git worktree add ../quarterly branches/2020Q4
% cd ../ports
```

**Q:** So what can I do instead?

**A:** Well, you can grab all of the refs in the upstream repo. git clones everything
via a 'refspec' and by default the refspec is:
```
        fetch = +refs/heads/*:refs/remotes/origin/*
```
which says just fetch the interesting refs that are used to create branches and tags.

However, the FreeBSD repo has a number of other things in it. To see
those, you'll have to fetch everything. To setup your repo to do that
```
git config --add remote.origin.fetch '+refs/*:refs/origin/*'
```
which will give you everything in the repo. Please note, that this
also grabs all the unconverted vendor branches and the number of refs
assocaited with them is quite large.

You'll need to refer to these 'refs' with their full name because they
aren't in the `head` namespace.
```
        git log refs/origin/vendor/zlib/1.2.10
```
would look at the log for the vendor branch for zlib starting at 1.2.10.

## Integrators

# A brief note about Subversion Projects and User branches

In the conversion to git, we plan on discontinuing the projects and
user branches. This should have been communicated sooner, but has not
been.

The historical projects and user branches will be converted to git and
be read only. You'll not be able to push commits to them once we've
made the conversion. These are lightly used today, and it is simpler
to discontinue this service than to deal with the logistical issues
having them in the tree would cause.

To not pollute the branch namespace, they will be not
fetched by default. One needs to add the following to the remote
for `freebsd`
```
	fetch = +refs/projects/*:refs/projects/*
	fetch = +refs/user/*:refs/user/*
```
to fetch them. You can also choose to fetch a smaller portion of
that namespace if you like.

Since they are in a separate namespace, you'll need to add `ref/`
to the front of them name. To re-create your old user branch:
```
% git checkout -b masq ref/user/imp/masq
```
This will create a local branch named masq. But you'll be unable to
push these branches to FreeBSD's src repo.

We've done this to simplify the FreeBSD repository. Going forward, it
will just be for project branches. Collaberative works will be done in
forked repositories. There are several sites that can host this. While
at the present we have no plans to offer hosting services for this
purpose, but if there's demand those plans can change in the future.
Most of this work has alrady moved, however.

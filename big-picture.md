# Big Picture Overview of FreeBSD's git migration

## Schedule

| Date | Event |
|------|-------|
| December 5th, 2020 | Doc repo transition started |
| December 19th, 2020 | Src repo transition to start |
| End of March, 2021 | Ports repo slated to transition to git |

## Main Design Points

The first phase of FreeBSD's migration to git focuses on replacing
Subversion with Git.  This is a modest goal where the project's work
flow will remain much as it is today, but with Subversion replaced
with Git. The bug tracking (bugzilla) and code review (phabricator)
will remain in place. We'll self-host the git repo and manage
the first layer of mirrors ourselves. We'll publish the repository to
github, gitlab and other places that are useful for our developers,
users and contributors.

The project will strive to maintain a linear history in the new src
repo.  Developers will be expected to land their changes as a logical
series of commits, much as has become the norm in other projects. For
the most part, developers are doing this today, so using a tool (git)
that facilitates this will help increase compliance and improve code
quality.

MFCs will be done as cherry picks from the main repo, with some
decoration added to the commit messages to help us track MFCs. Git
provides most of what's needed to track cherry picks, but fails when
the cherry picks are squashed as has been the project custom (since
current is for new features to mature, stable should have those
features committed all at once). The details of this will likely
evolve in the months after the conversion.

Vendor branches will be the one exception to the linear history
requirement. The vendor branch in git is similar to how it was in
subversion: a tree of files that are similar to the way they are
distributed naively. All the merge commits from the old vendor trees
have been converted to git branches. Just like with subversion, the
vendor branches are not fetched by default, at least until they are
bootstrapped. Like the conversion from CVS to Subversion, vendor
branches will need to be bootstrapped the first time there's a new
import. vendor branches are then merged to the main branch with `git
subtree merge`. Since we're not using submodules, users of the tree
need not install submodules, recursively grab them or put up with any
of the logistical issues submodules bring to the table.

Hashes changing from beta github tree. Due to a large number of bad
translations that have happened when exporting from subversion to
github, we're rerunning the conversion process. This will necessarily
create new hashes for the entire tree. The errors that prompted this
decision were such that every time we had to do anything with history,
such as extract a program for its own repo, the errors would get in
the way and create a lot of extra work. They also manifested
themselves in other ways that caused confusion. Rather than put up
with these quirks for all time, and waste resources forever on it, we
decided to redo the conversion. This also allowed us to correct a
few names that were wrong and fix other less important artifacts of
the translation. Fortunately, git has a number of tools that allow
easy transition from one stream of hashes to another.

When we converted to Subversion, there were no good places to host
collaborative work. The project filled the gap by creating two
hierarchies in the subversion repository: the projects hierarchy for
work that would likely be merged, and the user hierarchy for work that
would not. Since that time, services like github, gitlab, sourceforge,
project.net, and others have sprung up to provide collaboration
points. As such, in the first phase, we're converting the old projects
and user branches. But they will become read only. And they won't be
in the default revs that are fetched (you'll have to opt-in to get
them). Over the past few years, traffic has dropped significantly in
these branches as the work has already migrated to other places. Once
the initial phase is done, the project may allow creation of 'forks'
or the hosting of other repositories in the project's git hosting
environment. It's an open question as to whether the work to create
and manage this infrastructure provides enough benefit.

The name of the default branch will be `main` in keeping with the new
git default name. The beta mirror at github used the old default of
`master` as it's default branch. Users migrating from that will need
to adjust their tooling.

The plan for github is still working out the details. The basic plan
is to save the current refs/branches/tags in a separate name space to
allow current users to refer to the old names. We'll then push the new
repo to github. By default, new users will get only the new stuff,
while old users won't have the old stuff deleted for a while. This
should provide for an orderly transition.

The ports tree needs to make the switch just before a quarterly branch
to avoid needing to mirror changes to subversion. Since there are a
number of details to work out, they are planning to make the switch in
March to allow time to consolidate their plans.

The docs tree has been converted. We're in the process of mopping up
from the different issues related to this that have come to light. So
far it's the usual thing you'd expect: the automated web site building
script needed to be changed to pull from git instead of subversion,
for example.

Release engineering plans to do future stable/12 releases from
subversion to ensure maximum compatibility with current users
(especially $FreeBSD$ tags). Starting with 13.0, releases will be cut
from git.

The $FreeBSD$ tag is being phased out. Git provides no way to
implement it in a sane and helpful way. We're recommending the project
keep $FreeBSD$ in the source tree until stable/12 is retired.

Git uses hashes instead of a sequentially increasing number. The
conversion will include notes (that must be fetched separately) to
allow a git revision to map to a subversion revision. A script is
planned for the tools tree in the src repo.


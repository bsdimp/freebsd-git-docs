# Github mirroring plan

## FreeBSD History

The FreeBSD project is currently mirroring to github an experiemental
tree. Due to the large number of bugs that adversely affects the
quality of the tree, and its day to day use for anything more
complicated that merging and commits and simple history browsing, the
move to git project opted to redo the hashes and fix many historical
mistakes as part of its effort. As such, a migration plan is needed,
so this describes it.

The current https://svn.freebsd.org/base repo is mirrored to github as
https://github.com/freebsd/freebsd with the branches and tags
preserved as appropriate for git. So, base/head is the represented as
the 'master' branch. All the other branches map from base/branch/name
to branch/name. Tags are similarly preserved. The
https://svn.freebsd.org/doc repo is mirrored to
https://github.com/freebsd/freebsd-doc and the
https://svn.freebsd.org/ports to
https://github.com/freebsd/freebsd-ports (both with the same
mappings).

## Git Background

A git repo is usually clone with a 'git clone' command. This leads to
an interesting condundrum. The FreeBSD project has /usr/src,
/usr/ports and /usr/doc. It would be nice to have project resources
setup to by default populate those areas with a simple git clone.

On the other hand, these names are fairly generic. In the larger
world, src, doc and ports are too generic to know what's going on. The
typical practice is to have some descriptive name or some varation on
that descriptive name. Software whose source of truth is elsewhere,
but mirrored to github often has a different name than internal name
that's used for this reason (though sometimes they don't). Sadly,
github does not allow for an aribtrary number of layers beyond this
(so we can't call the repo freebsd/src, for example).

# doc mirroring

Today, 

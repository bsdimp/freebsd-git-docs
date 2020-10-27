# How FreeBSD implements git

## Initial Repositories

There will be three initial repositories.

The `doc` repository will contain all the project's documentation.

The `src` repository will contain the base OS source code.

The `ports` repository will contain the ports system.

Note: Theres discussions about prefixing freebsd- to the front of those.

In addition to these main repositories, a number of specialized
repositories exist to cooridnate work on things like packages and
graphics drivers. These repositories have their own rules and are
organized in a way that makes the best sense for the group working on
these things. Their organization is beyond the scope of this document.

## The `doc` repository.

The documentation repository ('doc') is the simplest repository. It has
no branches (large scale collaberation projects are done with
forks). It has a number of tags that describe the doc tree at various
releases.

The doc tree contains the content for the FreeBSD handbook, the web
site and different incidental articles about the project and its
operation.

The repository uses a 'rebase' model. In this model, there are no
merge commits, only curated patches that have been rebased forward to
the tip of the tree and are 'fast forward'able and contain no merge
commits. Force commits are forbidden.

This may undergo changes after the conversion to asciidoc completes.

## The `ports` repository

The ports repository has all the files we use to generate our
packages. It's structure is currently being debated.

## The `src` repository

The 'src' reposiotry contains all the source code needed to build
FreeBSD.

There are four kinds of branches in this repository.

The 'main' branch contains the mainline development of the operating
system. This is "FreeBSD CURRENT" and all new work is expected to come
in here when ready.

The `stable/\*` branches are the projects stable, release branches. We
create a new one every couple of years and maintain it for a
while. Changes that are mature are cherry picked from the `main`
branch into these branches (though sometimes direct commits happen, or
the commits need to be tweaked).

The `releng/\*` branches are branches for the release engineer to
create releases for the project.

The `vendor/\*` branches are vendor branches. These are used to manage
importing new software and help track local changes not yet in
upstream (or that will never be in upstream).


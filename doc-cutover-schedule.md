# FreeBSD Doc tree conversion

The FreeBSD project will be converting from Subversion to Git in the
coming weeks and months. The doc repo will be converted starting
December 5th. This document provides a detailed schedule.

## Timetable

Note: specific times are subject to change as the detailed logistics are worked out.

| time (UTC)    | who     | what                                                     |
| ------------- | ------- | -------------------------------------------------------- |
| Dec 1st 23:00 | imp     | Send mail to community about cutover                     |
| Dec 2nd 23:59 | so@     | Last Advisory before cut over                            |
| Dec 4th  0:00 | re@     | Last snapshot before cut over starts                     |
| Dec 4th 16:00 | uqs     | Freeze hashes for freebsd-doc                            |
| Dec 4th 16:01 | lwhsu   | Finalize Git repo at freebsd.org repo                    |
| Dec 4th 23:30 | gjb     | Commit updates to webupdate/webupdate.wrapper to SVN     |
| Dec 5th  0:00 | gjb     | Start switch website / handbook building from SVN to Git |
| Dec 5th 12:00 | gjb     | GO/NOGO on switch finalization                           |
| Dec 8th  2:58 | uqs     | Turn off scheduled SVN -> Git converter                  |
| Dec 8th  2:59 | lwhsu   | Make a final commit to Subversion                        |
| Dec 8th  3:00 | lwhsu   | Turn off write access to Subversion                      |
| Dec 8th  3:01 | lwhsu   | Snapshot the Subversion repository repo filesystem       |
| Dec 8th  3:02 | uqs     | Start final run of SVN to Git converter                  |
| Dec 8th  3:10 | uqs     | Push converted tree to GitHub/GitLab                     |
| Dec 8th 10:00 | lwhsu   | Turn on push to Git                                      |
| Dec 8th 10:01 | lwhsu   | Push 'Welcome to Git' commit                             |
| Dec 8th 12:00 | so@     | Next advisory window opens                               |
| Dec 11th 0:00 | re@     | Next snapshot starts                                     |

## Open Issues

1. What about GitLab? (bapt?)
2. Final plan for GitHub conversion (imp)
3. Finish documentation on new doc workflow -- in progress (imp)

Announcement draft: https://hackmd.io/0r0OnsTjRTyojFeDpJZBng

## Old vs New URL translation table

SVN infra -> Git infra map

| Item                                     | SVN                             | GIT                                 |
| ---------------------------------------- | ------------------------------- | ----------------------------------- |
| Web-based repository browser             | https://svnweb.freebsd.org      | https://cgit.freebsd.org            |
| Distributed mirrors for anonymous readonly checkout/clone | https://svn.freebsd.org svn://svn.freebsd.org | https://git.freebsd.org git+ssh://anongit@git.freebsd.org |
| Read/write Repository for committers (*) | svn+ssh://(svn)repo.freebsd.org | git+ssh://git@(git)repo.freebsd.org |

(*) Before all repositories in SVN have been migrated, the repo.freebsd.org will be pointing to one of:
    - svnrepo.freebsd.org
    - gitrepo.freebsd.org
    please use the hostname that explicitly includes the VCS name to access the right repositories during the migration. repo.freebsd.org will be the canonical FreeBSD Git repository for the committers after all the repositories migrated to Git.

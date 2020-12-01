# FreeBSD Doc tree conversion

The FreeBSD project will be converting from subversion to git in the
coming weeks and months. The doc repo will be converted starting
December 5th. This document provides a deatiled schedule.

## Timetable

**DRAFT** This is still being revised

| time (UTC)    | who         | what                                                     |
| ------------- | ----------- | -------------------------------------------------------- |
| Dec 1st 23:00 | imp         | Send mail to community about cutover                     |
| Dec 2nd 23:59 | so@         | Last Advisory before cut over                            |
| Dec 4th  0:00 | re@         | Last snapshot before cut over starts                     |
| Dec 4th 16:00 | uqs         | freeze hashes for freebsd-doc                            |
| Dec 4th 16:01 | lwhsu       | finalize git repo at freebsd.org repo                    |
| Dec 4th 23:30 | gjb         | commit updates to webupdate/webupdate.wrapper to svn     |
| Dec 5th  0:00 | gjb         | start switch website / handbook building from svn to git |
| Dec 5th 12:00 | gjb         | GO/NOGO on switch finalization                           |
| Dec 8th  2:58 | uqs         | turn off scheduled svn -> git converter                  |
| Dec 8th  2:59 | lwhsu       | Make a final commit to subversion                        |
| Dec 8th  3:00 | lwhsu       | turn off write access to subversion                      |
| Dec 8th  3:01 | lwhsu       | snapshot the subversion repository repo filesystem       |
| Dec 8th  3:02 | lwhsu       | start final run of svn to git converter                  |
| Dec 8th  3:10 | lwhsu/uqs   | push converted tree to github/gitlab                     |
| Dec 8th 10:00 | lwhsu       | turn on push to git                                      |
| Dec 8th 10:01 | lwhsu       | push 'welcome to git' commit                             |
| Dec 8th 12:00 | so@         | Next advisory window opens                               |
| Dec 11th 0:00 | re@         | Next snapshot starts                                     |

## Open Issues

1. What about gitlab? (bapt?)
2. Final plan for github conversion (imp)
3. Finish documentation on new doc workflow -- in progress (imp)

Announcement draft: https://hackmd.io/0r0OnsTjRTyojFeDpJZBng

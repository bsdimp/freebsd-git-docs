# Important URLs of the FreeBSD Git Infrastructure

This document contains the important URLs of the Git infrastructure of the
FreeBSD project, and the mapping for the Subversion infrastructure.

## Old vs New URL translation table

Following is the quick map for the essential URLs from Subversion to Git:

`${repo}` can be replaced by `doc`, `ports` and `src`. (*)

| Item                                     | Subversion                             | Git                                 |
| ---------------------------------------- | ------------------------------- | ----------------------------------- |
| Web-based repository browser             | `https://svnweb.freebsd.org/${repo}`      | `https://cgit.freebsd.org/${repo}`            |
| Distributed mirrors for anonymous read-only checkout/clone | `https://svn.freebsd.org/${repo}` `svn://svn.freebsd.org/${repo}` | `https://git.freebsd.org/${repo}.git` `ssh://anongit@git.freebsd.org/${repo}.git` |
| Read/write Repository for committers (**) | `svn+ssh://(svn)repo.freebsd.org/${repo}` | `ssh://git@(git)repo.freebsd.org/${repo}.git` |

  - (*) In subversion, `base` is used for FreeBSD src repository (`src`).
  - (**) Before all repositories in SVN have been migrated, the URL of the
    read-write repository, repo.freebsd.org, will be pointing to one of:
      - svnrepo.freebsd.org
      - gitrepo.freebsd.org

    Please use the hostname that explicitly includes the VCS name to
    access the right repositories during the migration. `repo.freebsd.org`
    will be the canonical FreeBSD Git repository for the committers after
    all the repositories migrated to Git.

There are also external mirrors maintained by project members available, please refer to "External mirrors" section.

### SSH related information

 - `ssh://${user}@${url}/${repo}.git` can be written as `${user}@${url}:${repo}.git`, i.e., following two URLs are both valid for passing to git:
     - `ssh://anongit@git.freebsd.org/${repo}.git`
     - `anongit@git.freebsd.org:${repo}.git`

   As well as the read-write repo:
     - `ssh://git@(git)repo.freebsd.org/${repo}.git`
     - `git@(git)repo.freebsd.org:${repo}.git`

- gitrepo.FreeBSD.org host key fingerprints:

  - ECDSA key fingerprint is `SHA256:seWO5D27ySURcx4bknTNKlC1mgai0whP443PAKEvvZA`
  - ED25519 key fingerprint is `SHA256:lNR6i4BEOaaUhmDHBA1WJsO7H3KtvjE2r5q4sOxtIWo`
  - RSA key fingerprint is `SHA256:f453CUEFXEJAXlKeEHV+ajJfeEfx9MdKQUD7lIscnQI`

- git.FreeBSD.org host key fingerprints:

  - ECDSA key fingerprint is `SHA256:/UlirUAsGiitupxmtsn7f9b7zCWd0vCs4Yo/tpVWP9w`
  - ED25519 key fingerprint is `SHA256:y1ljKrKMD3lDObRUG3xJ9gXwEIuqnh306tSyFd1tuZE`
  - RSA key fingerprint is `SHA256:jBe6FQGoH4HjvrIVM23dcnLZk9kmpdezR/CvQzm7rJM`

These are also published as SSHFP records in DNS.


## Web-based repository browser

The FreeBSD project currently uses cgit as the web-based repository browser: https://cgit.freebsd.org/
The URL of the indivirual repository is at: https://cgit.freebsd.org/${repo}/

## For Users

Using `git clone` and `git pull` from the official distributed mirros is recommended. The GeoDNS should direct you to the nearest mirror to you.

## For Developers

This section describes the read-write access for committers to push the commits from develoeprs or contributors.  For read-only access, please refer to the users section above.

We only document the important URLs here, the full information of retrieving all data in the repository is available at:
https://github.com/freebsd/git_conv#gimme-the-repo

### Daily use

* Clone the repository:
  ```
  git clone -o freebsd --config remote.freebsd.fetch='+refs/notes/*:refs/notes/*' https://git.freebsd.org/${repo}.git
  ```
  Then you should have the official mirrors as your remote:
  ```
  $ git remote -v
  freebsd  https://git.freebsd.org/${repo}.git (fetch)
  freebsd  https://git.freebsd.org/${repo}.git (push)
  ```

* Config the FreeBSD committer data:

  The commit hook in repo.freebsd.org checks the "Commit" field matches the
  committer's information in FreeBSD.org.  The easiest way to get the suggested
  config is by executing `/usr/local/bin/gen-gitconfig.sh` script on freefall:

  ```
  $ gen-gitconfig.sh
  [...]
  git config user.name (your name in gecos)
  git config user.email (your login)FreeBSD.org
  ````

* Set the push URL:
  ```
   $ git remote set-url --push freebsd git@gitrepo.freebsd.org:${repo}.git
  ```
  Then you should have separated fetch and push URLs as the most efficient setup:
  ```
  $ git remote -v
  freebsd  https://git.freebsd.org/${repo}.git (fetch)
  freebsd  git@gitrepo.freebsd.org:${repo}.git (push)
  ```
  Again, note that `gitrepo.freebsd.org` will be canonicalized to `repo.freebsd.org` in the future.

* Install commit message template hook:
  If you also cloned the src repository, we recommend creating a symlink to the hook so that future
  updates will be received:
  ```
  ln -s `realpath <path-to-src-checkout>/tools/tools/git/hooks/prepare-commit-msg` .git/hooks/
  ```
  If not, you can also download the commit message hook:
  ```
  fetch https://cgit.freebsd.org/src/plain/tools/tools/git/hooks/prepare-commit-msg -o .git/hooks
  chmod 755 .git/hooks/prepare-commit-msg
  ```

### "admin" branch

The `access` and `mentors` files are stored in a orphan branch, `internal/admin`, in each repository.

Following example is how to check out the `internal/admin` branch to a local branch named `admin`:

```
git config --add remote.freebsd.fetch '+refs/internal/*:refs/internal/*'
git fetch
git checkout -b admin internal/admin
```

For browsing `internal/admin` branch on web:
https://cgit.freebsd.org/${repo}/log/?h=internal/admin

## External mirrors

Those mirrors are not hosted in FreeBSD.org but still maintained by the project members.
Users and developers are welcomed to pull or browse repositories on those mirrors.
The project workflow with those mirrors are still under discussion.

### Codeberg
  - doc: https://codeberg.org/FreeBSD/freebsd-doc

### GitHub
  - doc: https://github.com/freebsd/freebsd-doc

### GitLab
  - doc: https://gitlab.com/FreeBSD/freebsd-doc

## Mailing lists

General usage and questions about git in the FreeBSD project: [freebsd-git](https://lists.freebsd.org/mailman/listinfo/freebsd-git)

Commit messages will be sent to following mailing lists:

- [dev-commits-doc-all](https://lists.freebsd.org/mailman/listinfo/dev-commits-doc-all): All changes to the doc repository
- [dev-commits-ports-all](https://lists.freebsd.org/mailman/listinfo/dev-commits-ports-all): All changes to the ports repository
- [dev-commits-ports-main](https://lists.freebsd.org/mailman/listinfo/dev-commits-ports-main): All changes to the "main" branch of the ports repository
- [dev-commits-ports-branches](https://lists.freebsd.org/mailman/listinfo/dev-commits-ports-branches): All changes to the quarterly branches of the ports repository
- [dev-commits-src-all](https://lists.freebsd.org/mailman/listinfo/dev-commits-src-all): All changes to the src repository
- [dev-commits-src-main](https://lists.freebsd.org/mailman/listinfo/dev-commits-src-main): All changes to the "main" branch of the src repository (the FreeBSD-CURRENT branch)
- [dev-commits-src-branches](https://lists.freebsd.org/mailman/listinfo/dev-commits-src-branches): All changes to all stable branches of the src repository

For more information, please refer to the "Commit message lists" section of C.2. "Mailing Lists" in handbook: https://www.freebsd.org/doc/en/books/handbook/eresources-mail.html

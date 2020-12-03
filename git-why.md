# FreeBSD moving to Git: Why

With luck, I'll be writing a few blogs on FreeBSD's move to Git later this year. Today, we'll start with "why"?

## Why?

There's a number of factors motivating the change. We'll explore the reasons, from long term viability of Subversion, to wider support for tools that will make the project better. Today I'll enumerate these points. There are some logistical points around how the decision was made. I'll not get into the politics about how we got here. While interesting for insiders who like to argue and quibble, they are no more relevant to the larger community that the color of the delivery truck that delivered groceries to your grocer this morning (even if it had the latest episode of a cool, scrappy cartoon cat that was involved in a multi-year arc wooing the love of his life by buying food at this store).

### Apache has moved on, so has LLVM
The Apache Foundation used to be the care taker and main user for Subversion. They used Subversion for all their repos. While they are still technically the caretaker of Subversion, they've moved all their repositories to Git. This is a worrying development because the foreseeable outcome of this will be less Subversion development. This will mean the FreeBSD project will need to undertake supporting Subversion if we remain on it in the long term. FreeBSD is now the last, large, Open Source project using Subversion. LLVM has made its transition to Git recently. There are very real concerns about the health and viability of the Subversion ecosystem, especially when compared to the thriving, vibrant Git ecosystem.

### Better CI support
Git have more support for newer CI tools than Subversion. This will allow us, once things are fully phased in, to increase the quality of the code going into the tree, as well as greatly reduce build breakages and accidental regressions. While one can use CI tools outside of Git, integration into a Git workflow requires less discipline on the part of developers, making it easy for them to fix issues found by CI as part of the commit/merge process before they affect others.

### Better merging
Git merging facilities are much better than Subversion. You can more easily curate patches as well since Git supports a rebase workflow. This allows cleaner patches that are logical bits for larger submissions. Subversion can't do this.

Git also allows integration of multiple Git repositories with subtree rewriting via 'git subtree merge'. This allows for easier tracking of upstream projects and will allow us to improve the vendor import workflow.

### Robust mirroring
Git can easily and robustly be mirrored. Subversion can be mirrored, but that mirroring is far from robust. One of the snags in the Git migration is that different SVN mirrors have different data than the main repo or each other. Mirroring in Git is built into the work flow. Since every repo is cloned, mirroring comes along for free. And there's a number of third party mirroring sites available, such as GitHub, GitLab and SourceForge. These sites offer collaboration and CI add-ons as well.

Git can sign tags and commits. Subversion cannot. We can increase the integrity of the source of truth though these measures.

### Features from 3rd party sites
Mirroring also opens up more 3rd party plug-ins. In terms of automated testing and continuous integration, GitLab can do some things while GitHub can do other things. Tests can be run when branches are pushed. Both platforms have significant collaboration tools as well, which will support groups going off and creating new features for FreeBSD. While one can use these things to a limited degree, with Subversion mirrored to GitHub, the full power of these tools isn't fully realized without a conversion to Git.

The wide range of tools available on these sites, or in stand-alone configurations, will allow us to have both pre-commit checks, as well as long-term post-commit tests running on a number of different platforms. This will allow the project to leverage existing infrastructure where it makes financial sense to let others run the tests, while still allowing the project to retain control of the bits that are critical to our operations.

### Improved user-submitted patch tracking and integration
One area we've struggled with as a project is patch integration. We have no clear way to submit patches that will be acted on in a timely fashion, or at least that's the criticism. We do have ways, but they are only partially effective at integrating patches into the tree. Pull requests, of various flavors, offer a centralized way to accept patches, and have tools to review them. This should lower the friction to people submitting patches, as well as making it easier for developers to review the patches. Other projects have reported increased patch flow when moving to Git. This can also be coupled with automated testing and other validation of the patch before the developer looks at it, which addresses one of the big issues with past systems: very low signal to noise ratio. While not a panacea, it will make things better and more widely use the scarce developer time.

### Collaboration
Some have said that Git, strictly speaking, isn't a pure source code management system. It is really a collaboration tool that also supports versioning. This may sound like a knock on Git, but really it's git's greatest strength. Git's distributed model allows for easier and more frequent collaboration. Whole webs sites have been built around this concept, and show the power of easy collaboration to amplify efforts and build community.

### Skill set
All of this is before the skill set arguments are made about kids today just knowing Git, but needing to learn Subversion. That has increasingly been a source of friction. This argument is supported by anecdotal evidence of people complaining about having to learn Subversion, about professors having to take time to teach it, etc. In addition, studies in the industry have shown a migration to Git away from other alternatives. Git now has between 80% and 90% of the market, depending on which data you look at. It's so widely used that our non-use of it is a source of friction for new developers.

### Developers are already moving
More and more of the active developers on the project use Git for all or part of their development efforts. This has lead to a minor amount of friction because all these ways are not standardized, have fit and finish issues when pushing the changes into subversion, and cause friction. The friction is somewhat offset by the increase in productivity they offer these developers. The thinking is that having Git as the source of truth will unlock the potential of Git even more to increase productivity.

## Downsides
First, Git has no keyword expansion support for implementing $FreeBSD$. While one can quibble over the Git add-ins that do this sort of thing, the information added isn't nearly as useful as Subversions'. This will represent a loss. However, tagged keyword expansion has been going out of style for a long time. It used to be that source code control systems would embed the commit history in files committed, only to discover weird things happened when they were imported into other projects so the practice withered. We ran into a similar issue years ago with $Id$ and all the projects switched to $FooBSD$ instead. This was useful when source code tracking was weak and companies randomly imported versions: it told us what versions were there. Companies now tend to do this with Git, which has better tracking to the source abilities. The value isn't zero, but it's much lower than when we adopted it in the 90s. Git also doesn't support dealing with all the merge conflicts it causes very well. Since the tool rewards people for importing in a more proper way, more companies appear to be using it that way, lessening the need for explicit source marking. [edit: some don't consider this a loss at all].

Second, Git doesn't have a running count of commits. One can work around this in a number of ways, but there's nothing fundamental that can be used as a commit number as reliably as Subversion. Even so, the workarounds suffice for most uses and many projects are using Git and commit counts successfully today.

Third, the BSDL Git clients are much less mature than the GPL ones. Until recently, there was no C client that could be imported into the tree. While one might debate whether or not that's a good idea, there's a strong cultural heritage of having all you need in the base system that's hard to shrug off. OpenBSD recently wrote Game of Trees (Got) which has an agreeable license (but a completely different command line interface, for good or ill). It has its issues, which aren't relevant here, but is maturing nicely. Even with the current restrictions, it is usable. There is an active port of Got to FreeBSD due to the large number of OpenBSDisms that are baked in (some necessary, some gratuitous). The OpenBSD people are open to having a portable version of Got, so this is encouraging.

Finally, Change Is HARD. It's easy to keep using the same tools, with the same work flows with the same people as you did yesterday. Learning a new system is difficult. Migrating one work flow to another is tricky. You have to balance the accumulated knowledge and tooling benefits vs. the cost it will take to move to something new. The Git migration team views moving from subversion to Git as the first step in many for improving and refining FreeBSD's work flow. As such, we've tried to create a Git workflow that will be familiar to old users and developers, and at the same time allow for further innovation in the future.

## Conclusion
Although it's not without risk and engineering trade-offs, the bulk of the evidence strongly suggests that moving to Git will increase our productivity, improve project participation, grow the community, increase quality and produce a better system in the end. It will give us a platform that we can also re-engineer other aspects of the project. This will come at the cost of retooling our release process, our source distribution infrastructure and developers needing to learn new tools. The cost will be worth it, as it will pay dividends for years to come.

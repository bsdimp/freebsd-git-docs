## Include appropriate metadata in a footer

Commit messages may have one or more of number of standard metadata tags in the final paragraph. Standard tags used in FreeBSD are:

| Tag | Description |
| -------- | -------- |
| PR: | FreeBSD problem report (Bugzilla) number |
| Reported-by: | ID of a 3rd party who reported the issue |
| Reviewed-by: | Reviewer ID |
| Tested-by: | ID of those who have tested the change |
| Approved-by: | Mentor or code owner who approved the change |
| Obtained-from: | Source of a change in another project |
| MFC-after: | Time period before merging the change from Current to Stable |
| MFC-with: | Associated commit that this change should be merged along with |
| MFH: | Yes/No whether ports change should be merged to quarterly branch |
| Relnotes: | Yes/No whether this change should be included in release notes |
| Security: | External reference for a security issue, such as a CVE number |
| Sponsored-by: | Organization or event that sponsored work on the change |
| Differential Revision: | Full URL of code review in FreeBSD's Phabricator instance
| Signed-off-by: | ID certifies compliance with https://developercertificate.org/ |

"ID" indicates either a FreeBSD userid, or a name and email
address. Multiple IDs may be presented as a comma-separated list, or
by repeating metadata tags on subsequent lines.

This represents a change from the prior FreeBSD practice of using
spaces in the metadata tags (other than in `Differential Revision`).
However, this standard conforms to the wider open-source communities use.
The project isn't requiring signed-off-by for commits at this time, but
don't want to preclude it in the future.

In addition, you can replace - with a space in the above tags as a transition aid.

"Submitted by" may still appear, but is generally discouraged as a legacy tag. See [How to deal with Pull Requests](pull-request.md) for details on attribution.

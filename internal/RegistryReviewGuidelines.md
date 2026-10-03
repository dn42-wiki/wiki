# dn42 Registry Reviewer Guide

## Purpose of Reviewing

The primary goal of reviewing is to ensure the quality of changes made to the registry.

Reviewers should verify that the submitted objects are consistent and make logical sense.

**Reviewing exists to help users register, not act as a blocker.**

## Automated Scripts vs. Manual Review

Whilst the automation has improved considerably and is now capable of capturing a lot of common issues, **automation is an aid for reviewers, not the authoritative decision-maker**.

Manual review is still required as automation can't catch everything, common exceptions are:
- Adherence to the allocation policy
- RFC4193 violations
- logical inconsistencies
- pointless comments
- external ASNs

Care should also be taken when the automation ran some time ago as the results may have been superceded.
- This is particularly relevant for IPv4 allocations as the number of exact /27's available is relatively small leading to duplication across registrations.
- Pipeline will respond to review requests and can be re-run on-demand, whilst this is useful for DNSSEC or authentication chagnes, be aware it won't catch duplicate IPv4 ranges.

Automation isn't perfect and can get it wrong, or might simply be unavailable.

## The Manual Review Process

Pull the PR Locally:

``` shell
git fetch --all
git reset --hard origin/master
git pull origin pull/<pr_number>/head
```

Verify signatures
- You can get public gpg keys from gitea using https://git.dn42.dev/username.gpg
- You can get public ssh keys from gitea using https://git.dn42.dev/username.keys

Run The Standard Registry Scripts:

``` shell
./check-my-stuff
./check-pol
```
*tip: run `git diff` after fmt-my-stuff to see what changed*
``` shell
./fmt-my-stuff
git diff
```

Post Results:
- Paste the output of git log --show-signature along with the outputs of check-my-stuff and check-pol as a comment against the PR.

*tip: Format these outputs in your comment using blockquote ` ```text` markers so they render correctly and are not interpreted as markdown.*

## Authentication & Signatures

You cannot use Gitea's in-built signature validation, as it only confirms the private key is loaded in the user's Gitea profile, not that the key matches the authorized mntner object.

Commits signed with a subkey of the master PGP key specified in the mntner object are completely valid and considered best practice.

If a user has multiple valid authentication methods, they should add the auth attribute multiple times on separate lines. Users only need to successfully validate against one `auth` method for approval.

## Common Review Scenarios


**Additional Resources** reviewers should check adherence to the allocation policy: https://dn42.dev/Policies and require the user to provide justification in-line with the policy

**De-aggregation** Announcing many small subnets (like /64s or /60s) instead of the full prefix is highly discouraged. Whilst de-aggregation is technically permitted, reviewers should guide users towards better routing practices.

Users should avoid registering multiple small subnets where possible; it's often a tactic to avoid the de-aggregation checks and in most cases just spams the registry with pointless objects.

**RFC4193 Violations** IPv6 prefixes in the fd00::/8 range should be uniquely random. Sequential or "magic" numbers (e.g., fd42::/48) violate RFC4193 section 3.2. Users should be encouraged to choose a fully random prefix but if they persist they need to explicitly confirm they understand the risks of prefix clashes before proceeding.

**Clearnet ASNs** External ASNs must update the source to be appropriate (e.g. RIPE/ARIN/APNIC). Reviewers should do a "best effort" check to ensure the dn42 user actually owns the clearnet ASN, such as checking WHOIS data or asking the maintainer directly, but this can be tricky depending on the registry.

**E-mails** The e-mail attribute cannot contain comma-separated addresses. Multiple emails require multiple e-mail attributes on separate lines. Contact attributes should not contain email addresses and should use the specific `e-mail` attribute.
- Whilst the `e-mail` attribute is optional in the schema, it should be treated as required. It's key that mntners are contactable and email is a primary method of object recovery if mntner keys are lost. Anonymous services exist if the user doesn't want to include their real email address.

**Personal Data** e.g. addresses or telephone numbers in submissions; users should be reminded of the privacy policy and must provide explicit confirmation that they understand the policy and still want to include personal data

If you get stuck or are unsure, ask for a secondary review. In exceptional circumstances requests can be escalated to the mailing list for community approval.

Mistakes happen, every single other reviewer has broken something at some point.

## Checklists

The checklist feature in gitea allows reviewers to embed interactive, trackable to-do lists by including markdown check boxes in the pull request body. 

Checklists are integrated in to the pipeline automation:
 - Pipeline will automatically add some standard tasks for new mntners
 - Pipeline will fail if there are outstanding checklist items

Reviewers may add their own tasks that will be monitored by pipeline by including HTML comment markers:

```text
<!-- pipeline-tasklist-start -->
---
Please complete the following tasks:

- [ ] I have read the allocation policies: https://dn42.dev/Policies
<!-- pipeline-tasklist-end -->
```

Tasks can be added without the markers, but pipeline won't see them and won't complain if they aren't completed.

## Deletions

When reviewing deletions, give the submitter more leeway. If someone is leaving they may not be interested in strictly adhering to the review policies but we still do want to clear out dead allocations. Submitters may not feel that they can edit objects that are not owned by them (e.g. references from other user's AS-SETs)

Ensure no dangling references are left behind by grepping the entire registry for the deleted aut-num, mntner, or person objects. Check domains or route objects that might point to removed inetnums.

Reviewers can step in if required and add their own commits to the user's PR to clean up any missed references on behalf of a departing user. In this case don't squash the reviewer's cleanup commit with the user's commit, so it remains clear exactly what was done and by whom.

## Communication & PR Management

Try and capture all your review notes and issues in a single comment to avoid frustrating back-and-forth exchanges with the submitter.

It can often make sense to wait a few hours before reviewing a freshly opened or updated PR to ensure the submitter has finished making their changes. New submissions often start with several changes and updates.

Always add Gitea labels to indicate the status of the PR:

| label                      | usage                                                                 |
|:---------------------------|:----------------------------------------------------------------------|
| authentication pending     | The submitter needs to sign the commit or verify authorization        |
| needs confirmation         | The request is technically correct, but clarification is required     |
| needs work                 | The submitter must make changes to fix errors                         |
| needs consensus            | The PR requires community approval on the mailing list                |
| secondary review requested | Used when you need another reviewer to double-check a complex PR      |
| cleanup                    | Designated for non-user, internal maintenance changes (like DN42-MNT) |
| blocked                    | Marks a PR to prevent other reviewers merging it                      |

## Merging PRs

PRs do not need to be updated to be merged. Changes are typically to independent objects so can be merged as long as there are no conflicts.

Do not merge your own changes unless it's for an emergency fix. Even then, you should still try and obtain independent review via the #dn42-registry channel.

## Getting Involved

We constantly need new reviewers to help perform registry reviews and welcome any help; In 2026 the registry averaged ~6 new PRs a day. The key skill for reviewers is a keen eye for spotting details, particularly when looking for issues that evade the automation.

Start by maintaining a presence in #dn42-registry, it's key to have communication between reviewers and the channel is a place to ask questions and get feedback. We also have bots that alert to #dn42-registry if there are problems.

Anyone can comment on PRs so you don't need any special privileges or permission to make a start; the existing reviews will spot your contributions. We recommend new reviewers start by watching the PR queue and commenting on PRs as they come through to get the hang of it.

Even if the PR is ok, leave a comment on the PR to show you've reviewed it, this way the other reviewers can spot that you've already seen it.

Once you get a bit of experience with the process the admin will be able to add you to the registry-wranglers group so that you can can perform merges.

- Start Date: 2016-07-20
- RFC PR: (leave this empty)
- Issue: (leave this empty)

# Summary

QIIME 2 requires a versioning system that guarantees compatibility across many
discrete sub-projects. By adopting a rolling release cycle (coupled with
adherence to lockstep semantic versioning, herein "SemVer"), we can ensure that
stable releases are guaranteed for all QIIME 2 deployments. Additionally, a
rolling release cycle allows for the QIIME team to support the concept of a
long-term support release, which could be advantageous for QIIME users.

# Motivation

The overall architecture of QIIME 2 is a significant departure from QIIME 1,
moving from a monolithic structure to a set of modular components. Ensuring
compatibility across components is a new problem for the QIIME team, and
requires a consistent, easy to follow set of guidelines to minimize friction
for users and developers.

By synchronizing our projects (via SemVer), and establishing a predetermined
release schedule, we can provide guarantees to QIIME 2 users that all component
versions across a particular major/minor version will work together, as
advertised.

For example, the following components would all be guaranteed to be compatible,
as part of the 2.1 release cycle:

- `qiime2==2.1.13`
- `q2-types==2.1.0`
- `q2-diversity==2.1.4`

This simplifies communication during bug reporting, discussions, etc. by
allowing the user to indicate that they are using QIIME 2.1 (or 2.2, 2.3 ...
2.200), and developers should know more or less what that means. Also this
provides a predictable target for plugin developers.

SemVer is simply a vehicle for facilitating a rolling release cycle. The idea
here is that while new features can be added at point releases (2.1, 2.2...),
and features deprecated, no breaking changes to the public APIs are introduced
and upgrading within a major version is designed to be as simple as possible.

There are several advantages afforded by adopting this type of release scheme.
A rolling release schedule:

- is predictable
- can guide development efforts (in the spirit of a sprint)
- can support multiple release versions (e.g. LTS)

# Detailed design

## Overview

Release windows will occur every **X weeks**. The release window will by **Y
days** long. **All** projects under the QIIME 2 umbrella will issue a new
release within the window (lockstep versioning).

- The changelog will be updated to indicate the merged changes since the last
  release [see here][1]. If no changes have occurred, this is simply a version
  bump commit.
- A release branch will be cut from the repo's stable branch (this can be done
  ahead of a release window, too). e.g. `release-2-1`.
- A git tag will be applied to that release branch (e.g. `v2.1.0`).
- A build will be triggered, and will ideally push the build artifacts to
  conda-forge, GitHub Releases/Pages, pypi, etc.

Between releases individual components can release patch updates as needed,
with an appropriate git tag and build (e.g. `v2.1.13`, where 13 is the patch).

## Long-term Support (LTS) Releases

LTS releases allow users that might not be able to upgrade every X weeks to
opt-in to any relevant bugfixes by running LTS releases. LTS releases will
occur ever Z releases, and will proceed just like a normal release, except the
branch label will read `lts-2-5`. These releases will be supported until the
next LTS release comes out. LTS releases will receives critical bugfixes that
are identified during this window. These bugfixes will be released as patch
releases (e.g. `v2.5.3`, where the 3 is the patch). [This blog post][2] has a
nice overview and discussion of this concept.

## Plugins

The impact to plugin developers will be minimal. Plugin developers are under no
obligation to adhere to this release schedule (or versioning scheme), however
it wouldn't be a problem if some (or all) did. All that plugin developers will be
required to do is indicate a minimum version of QIIME that is required for the
plugin to work (e.g. 2.6). This is acceptable because of lockstep versioning.

## Other Notes

This RFC is largely based on [this RFC for the Ember project][3].

## Noteworthy Links

- [Autogen CHANGELOG][1]
- [Ember LTS blog post][2]
- [Ember Release Cycle RFC][3]

# How We Teach This

Education should be straightforward: an announcement post coupled with detailed
background (or perhaps even a link to the final version of this RFC) should
handle any need for justification. As far as users go, we can guide them
towards one of two scenarios:

- Need stability? Stick with an LTS release, and try to upgrade when a new
  LTS is released.
- More flexible? As long as your core components are all on the same
  major/minor release, and your plugins guarantee they support that version,
  you will be good to go.

# Drawbacks

Clearly this represents potentially a lot of work, and introduces quite a bit
of ceremony to the release process (althought ceremony isn't necessarily a bad
thing). By rolling out a build server this should help reduce any
build-specific friction.

There are potentially a lot of simultaneous moving parts here, which can be
difficult to orchestrate, and will require exceptional communication within
the QIIME team, and to the users of QIIME.

# Alternatives

QIIME 2 could choose to not move to a rolling release cycle, and plan to manage
dependencies in an ad-hoc manner. This would not allow for the support of LTS
releases.

# Unresolved questions

Plenty. Feel free to update this document as needed.

[1]: https://github.com/emberjs/data/blob/master/bin/changelog
[2]: http://emberjs.com/blog/2016/02/25/announcing-embers-first-lts.html
[3]: https://github.com/emberjs/rfcs/blob/master/text/0056-improved-release-cycle.md

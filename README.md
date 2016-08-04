# QIIME 2 RFCs

Request For Comments (RFCs) for QIIME 2.

## When

Typically bug fixes and minor enhancements are not viewed as significant
changes, and as such are best addressed via GitHub issues and the pull request
workflow. When in doubt, submit an issue to the relevant project: if the core
team recognizes that the scope exceeds that level, they will point you to this
document and process. Implementation of accepted RFCs (see below for details)
is organized outside of the actual RFC process, so the RFC author is under no
obligation to write the necessary implementation.


## Why

As QIIME 2 reaches a point of relative stability it will be critical to provide
a structured process for proposing significant changes to QIIME 2. This process
aims to provide a platform for rigorous discussion, ideally allowing for all
implications (positive and negative) to be realized (and accounted for) with
the proposed changes.


## What

The RFC process utilizes a formalized [template] to guide the proposal writing
process. Additional sections/subsections may be added to the template as
needed, but please refrain from removing any default sections. The aim of this
template is to ensure that all relevant discussion points are addressed, and
that any alternatives or modifications are presented. When in doubt, contact a
[member] of the QIIME 2 team.


## How

Here are the steps to get started with an RFC:

- Gather feedback. This can be accomplished via a GitHub issue, an email
  exchange, an in-person conversation, etc. Make sure that an RFC is the
  appropriate vehicle for introducing your change.
- Fork this repository: https://github.com/qiime2/rfcs
- On your fork, create a new branch, and title it something related to the RFC
- Copy the file `0000-template.md` to `accepted/0000-new-feature.md`, where
  'new-feature' describes your proposal. Don't worry about changing the
  prefixed RFC number.
- Fill in the RFC template. This is easier said than done: please ensure that
  you have completed all sections and included all relevant considerations and
  viewpoints. **A standard of quality is expected and if it does not meet that
  we are under no obligation to consider your RFC**. This means that RFCs
  should present convincing arguments and fully explore the ramifications of
  the design including drawbacks and alternatives. If these are not yet known
  they should be explicitly stated in the *Unresolved Questions* section of the
  proposal.
- Submit a pull request to this repository. This will alert the QIIME 2 team
  that your proposal is ready to begin the review process. Please include
  a link to the rendered markdown in your pull request comment. Format your
  pull request title as follows: "RFC: new feature", where "new feature" is
  the title of your proposal.

## RFC Lifecycle

The RFC lifecycle is expressed and tracked via pull request tags:

- `needs-guide`: Once an RFC is submitted as a pull request a QIIME team
  [member] will be assigned as an RFC guide. They will be the primary contact
  for questions and assistance, and are responsible for advancing the RFC
  through the review process. If no one is assigned after a week come find us
  on [Slack].
- `preparing-for-review`: This is an initial screening performed by the guide
  to ensure that the RFC meets the guidelines for review (detailed in *How*,
  above). The guide might request minor adjustments to comply with the RFC
  guidelines and will determine if the RFC subject matter is appropriate to
  begin review.
- `under-review`: Congratulations! Now is when the real work begins. The QIIME
  team will announce the review process through broadcasts on [Twitter],
  [Slack], and the [blog]. Review is an iterative process, and will likely
  generate feedback. Be prepared to address this feedback in your RFC. Feedback
  may be given from any interested party. The guide will collate relevant
  feedback and request updates to the RFC. Do not alter the branch history
  during these updates, it is critical to reviewers that GitHub comments remain
  in place during review.
- `final-comments`: The RFC is ready to be accepted and the guide has indicated
  a deadline for final feedback. This is the last opportunity to give input on
  the proposal. Speak now or forever hold your peace.
- `accepted`: Once accepted, the guide will merge the pull request through the
  GitHub interface using the squash and merge option. The guide is also
  responsible for formatting the commit message, which should be formatted as
  follows: "RFC: new feature", where "new feature" is the title of your
  proposal. The guide will then update the RFC front matter (pull request
  number and contributers) and the file name number, which should be
  incremented from the last accepted RFC's number.

Unfortunately, not everything goes according to plan - the following
contingencies are in place for accommodating these realities.

- `on-hold`: There is nothing wrong with this RFC, but further discussion
  should wait until some externality is resolved (e.g. waiting on another RFC).
- `rejected`: The RFC has been rejected, and will include a subtype tag to
  indicate the reasoning:
  - `create-new-rfc`: If the discussion results in significant modifications to
    the RFC, the authors will be asked to close the pull request and issue a
    new one with a summarization of the previous discussion, linking to any
    prior RFCs. The new pull request should be a separate branch which does not
    share commit history with the originals. The aim here is to preserve the
    discussion-to-date while providing a clean slate.
  - `lacks-momentum`: This will occur if the RFC author is unreachable, if
    there is a perceived lack of interest, or consensus cannot be reached.
  - `out-of-scope`: This will occur if the RFC does not align with the goals of
    QIIME.
  - `poor-quality`: This tag is reserved for RFCs with serious deficiencies,
    including (but not limited to): presentation, formatting, technical
    details, or concepts.


## Additional Resources

The RFC process is utilized in many major development efforts. This document
(and process) is based largely upon the [Rust] and [Ember] teams.

[template]: https://github.com/qiime2/rfcs/blob/master/0000-template.md
[member]: https://github.com/orgs/qiime2/people
[Twitter]: https://twitter.com/qiime_
[blog]: https://qiime.wordpress.com
[Slack]: http://qiime2-slackin.qiime.org
[Rust]: https://github.com/rust-lang/rfcs
[Ember]: https://github.com/emberjs/rfcs

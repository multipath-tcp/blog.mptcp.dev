---
layout: post
title:  "Backports"
---

The [previous post]({% post_url 2024-02-02-CI-CI-CI %}) mentioned that January
was still full of various "maintenance" tasks, mainly around the CI, and the
various tests we have. These tasks were somehow blocking other devs and
maintainers work. Without these blocking tasks, more time was available to look
at fixing issues, and preparing new features. Until a huge batch of backports
came! Read on to find out more about what happened in February!

<!--more-->

## Backports

Backports in the Linux kernel are mainly managed by the Stable team. They are
doing an excellent job, handling hundreds of patches per new version. To help
them to achieve that, kernel developers are supposed to tag them (`Cc:`) in
patches built on top of the last development version, and containing a fix. The
patch description should also contain a `Fixes:`, or an indication suggesting
which versions are concerned, and the eventual dependences. Without these
indications, the stable team can miss fixes, even if they try not to by guessing
fixes, thanks to some machine learning algorithms used by their "auto-selection"
scripts.

When fix commits reach Linus Torvalds Git tree, the stable team backports them
to the different versions requiring these corrections. In case of conflicts, the
stable team, they usually notify the different people involved in the patch, or
they backport other patches -- in theory, only harmful patches like clean-ups --
to be able to backport the fix without conflicts. This is sometimes useful to
backport other patches, to be closer to another version that has already been
tested before, to avoid diverging even more, making future fixes even harder to
backport, and possibly introducing new bugs.

On the maintainers side, they have to do a few things:

- Make sure the Stable team will track fixes by having the proper tags in the
  commit messages.

- Check the notifications when patches are being backported:

  - Either to check if it makes sense to backport, especially for dependences,
    and "auto-selected" patches.

  - Or to suggest solutions in case of conflicts: by asking to backport
    dependences, or by sending a new version that can be applied without issues.

- Make sure stable releases are still OK: some companies and individuals monitor
  the different stable versions, and do various tests. If tests are included in
  the kernel (KSelfTests, KUnit, etc.), some CIs are validating them, e.g.
  [LKFT](https://lkft.linaro.org/) are validating MPTCP KSelfTests and KUnit
  tests for us, and they notify us in case of regression. Of course, this works
  only if the tests are reliable, which is not always easy to have.

What can be a bit hard for the maintainers, is that a new stable version is
usually published soon after the first notification, sometimes just 1 or 2 days
after. That doesn't let a lot of time to interrupt other tasks to look at them.
But it is always possible to ask to hold some patches, or revert them after.

Around 125 patches related to MPTCP have been added to the different kernel
versions in February. 33 patches had to be adapted to older versions. 6 other
patches have not been backported, because the conflicts were too important, and
the issues they were fixing were not considered as serious enough.


## Preparing the future v6.9

The kernel development cycle is quite simple: when a new major version is
published, Linus Torvalds accepts new features during the following ~2 weeks.
After this "merge window", our Chief Penguin only accepts bug fixes during the
following ~7 weeks, where a new Release Candidate is published each week.

For MPTCP, we depend on the Networking tree. At any time, we can send fixes for
the `net` tree, but we can only send new features for the next version
(`net-next`) when Linus' "merge window" is closed. In other words, when Linus
publishes the 5th RC, it is really time to look at polishing the new features,
and sending them to the Netdev maintainers.

Note that we are still sending patches, and not pull requests. The current way
of working seems to work well for Netdev maintainers, and it allows other devs
of the core to easily follow the evolution. MPTCP is a bit particular, because
even if the code is well isolated, it is still very tied to TCP. Some specific
TCP features might need to take MPTCP into consideration -- e.g. TCP SYN
Cookies, socket options, etc. -- so it is good to be very visible.

Because the 5th RC has been published mid of February, the end of the month was
also full of reviews -- they take easily one hour per series, sometimes a lot
more, and sometimes even per version, depending on the modifications and if the
developer helped the reviewers by implementing, and documenting all changes --
and various bug-fixes to the recently added features, before sending them
upstream.


## What's next?

A few CI tasks have been scheduled for March: using
[runners with KVM support](https://github.com/multipath-tcp/mptcp_net-next/issues/474)
to get more stable selftests results and to get rid of Cirrus-CI (CPU time
limited), validating [MPTCP BPF tests](https://github.com/multipath-tcp/mptcp_net-next/issues/406)
to track regressions in this area, and switching to
[virtme-ng](https://github.com/multipath-tcp/mptcp_net-next/issues/472) because
the `virtme` tool we use is no longer maintained.

But, hopefully, this should not take too long, and instead new MPTCP features
can be developed, to have MPTCP support in more apps, e.g. the ones developed in
GO and Rust languages.


## Team work

It is important to note that what I presented here so far is mostly what I'm
working on. But I'm not alone in this project. For example, Paolo finished
`TCP_NOTSENT_LOWAT` support with MPTCP, and fixed duplicate subflow creation in
some cases. Mat helped with the reviews. Geliang the ability to dump addresses
used by the userspace path-manager, and did many clean-ups, and on how results
are presented in the selftests. Davide looked at fixing some issues.

A great community!

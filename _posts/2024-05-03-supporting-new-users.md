---
layout: post
title:  "Supporting new users"
---

During the last few months, a lot of work has been done to help other kernel
developers, and stabilize the current solution. It was really time to focus on
supporting new users and application developers. Read on to find out more about
what happened in April!

<!--more-->

## MPTCP case

Like many consequent projects modifying the Linux kernel, the development of the
MPTCP Linux Upstream project has been mainly supported by companies, for their
specific use-cases.

Because using multiple paths at the same time is still not common these days,
most OS's need to be configured to let MPTCP use different paths, and the apps
need to ask to use MPTCP. Companies working on the Linux implementation already
know about MPTCP, and how to configure it. Their clients have also certainly
been told what it is and how to use it in their product. But it has never been a
priority to have good documentation for the community. Of course, we were, and
still are, open to answer questions about MPTCP, but not everybody is curious
enough to find out what MPTCP is, how to configure it, and/or to ask questions
if something is not clear. That's a shame because attracting new users,
generally also means new contributors (bug reports, fixes, features, etc.), and
a better solution at the end.

This situation is now changing thanks to the [NLnet
project](https://nlnet.nl/project/MPTCP-deployability) I'm participating to,
funded through [NGI0 Core](https://nlnet.nl/core).

## New website

[mptcp.dev](https://www.mptcp.dev) has been completely rewritten and published
in April.

Before, the documentation about MPTCP in the Linux kernel was available in
different places:
- [mptcp.dev](https://www.mptcp.dev) was a summary of the wiki page, and assumed
  people already knew about MPTCP -- see the previous version
  [here](https://github.com/multipath-tcp/mptcp.dev/blob/531801e/README.md).
- The [wiki](https://github.com/multipath-tcp/mptcp_net-next/wiki) had some
  info, but it was also redirecting to info from different sources, including
  talks and articles, and out of date info -- see the previous version
  [here](https://github.com/multipath-tcp/mptcp_net-next/wiki/Home/34ba2883e2a30b15a149df94a941b446eb51e1d6).
- The documentation attached to the kernel only contained info about the
  [MPTCP sysfs variables](https://docs.kernel.org/networking/mptcp-sysctl.html),
  and recently about the
  [`mptcp_pm` Netlink API](https://docs.kernel.org/networking/netlink_spec/mptcp_pm.html).
- A [website](https://mptcp-apps.github.io/mptcp-doc/mptcp.html) has been
  initiated by the community, but never finished.
- RedHat has [documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/configuring_and_managing_networking/index#getting-started-with-multipath-tcp_configuring-and-managing-networking),
  but centred around RHEL.

When helping Maxime Durov, Dorian Craps, and Olivier Bonaventure to add native
MPTCP support to apps like
[lighttpd](https://github.com/lighttpd/lighttpd1.4/pull/132) and
[curl](https://github.com/curl/curl/pull/13278) as part of a project linked to
the UCLouvain, we realized it was urgent to have a website dedicated to new
users and apps developers, regrouping all required info about MPTCP, what it is,
how to configure it, use it, and more. With their great help, the website has
been updated, and announced on [social media](https://social.kernel.org/mptcp).
It even appeared at the top of [Hacker News](https://news.ycombinator.com/),
bringing thousands of visitors! So far, we got positive feedback: the new
website seems to be answering questions from new users, and app developers, and
even brought new contributors!

## What's next?

Various works have been started recently: adding missing features that are
important for some apps and libs which want to support MPTCP natively, a new lib
for MPTCP in Rust, a test service, adapting the packet scheduler API to support
new use-cases, and improving some corner cases, etc. That's quite a bit of work
in parallel, but that's coming "soon"!

But before, it is also time to wrap up all new but mature enough features to
send upstream for the inclusion in the future v6.10 version. It looks like we
have up to the 2nd week of May to have them accepted in the 'net-next' tree.
Check a [previous post]({% post_url 2024-03-04-backports %}) to know more about
the kernel development cycle.

## Team work

As always, it is important to note that what I presented here so far is mostly
what I was working on. But I'm not alone in this project. For example, Geliang
finished supporting the 'last time' fields in `MPTCP_INFO`, and did some
refactoring in the BPF selftests to reduce duplicated code ; Paolo fixed issues
reported by Christoph thanks to SyzKaller ; Mat helped with the code and website
reviews ; Gregory (new contributor!) is doing some experimentation with the
packet scheduler API, and fixing bugs along the way ; Dorian and Maxime
continued to add native MPTCP support in userspace apps.

A great community!

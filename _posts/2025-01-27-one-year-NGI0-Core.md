---
layout: post
title:  "One year of NGI0 Core support"
---

A bit more than one year ago, I started the [Angel Project]({% post_url
2024-01-01-Angel-Project %}): improving the "deployability" of Multipath TCP. A
project supported by [NGI0 Core](https://nlnet.nl/core), a fund established by
[NLnet](https://nlnet.nl). Before starting, I had good planning, something to
keep me busy for the following months. Not everything happened as expected, but
on the bright side, the impact of this project is probably higher than what I
expected! Read on to find out more about that!

<!--more-->

## Retrospective

When I'm thinking about what happened over the last year, I could say that I'm a
bit disappointed, simply because I didn't manage to finish all the tasks I
initially planned. But that feels wrong: yes, I didn't have time to finish some
of them, but I have been **really** busy on many other aspects, which, at the
end, had a very positive impact on Multipath TCP in general, but also in kernel
testing in the Linux Network subsystem.

As mentioned in a few of my previous blog posts, a lot of time has been spent
around the "maintenance"[^maintenance]. This time was and is still crucial for
the survival of the project, and to gain confidence of new users. For example,
**a lot** of various issues have been fixed. Without these fixes, and their
backports to stable versions, new features would have been nice to have, but
either tarnished by pre-existing bugs, or simply not implementable.

Note that a part of the "maintenance"[^maintenance] was dedicated to migrate and
stabilise the CI system. A part of this work was also beneficial for the Linux
Network subsystem with [NIPA](https://github.com/linux-netdev/nipa/). Thanks to
all of that, I was able to join and contribute to the last
[Netconf](https://netdev.bots.linux.dev/netconf/2024/index.html) with so many
brilliant and nice Linux developers! This part benefits a lot more people.

[^maintenance]: "Maintenance" is a generic term which for a kernel maintainer of
    an active subtree includes: communication with the community, organising
    regular meetings, answering questions, tracking, analysing and fixing bugs,
    fixing issues with anything related to the workflow like the CI and other
    tools and services, refactoring code to ease the inclusion of new features
    or fixes, reviewing and accepting work from others, sending modifications to
    be included in the official Linux kernel, helping with the backports, and
    doing the different follow-up.

Tasks that had probably the most important impact were not directly on the
kernel side, but around:

- The totally reworked [website](https://www.mptcp.dev) can now reply to most
  questions from new and existing users: what is MPTCP, how can it help, how to
  set it up, what apps devs need to do, how to contribute, etc.

- The documentation has been improved (or created) on different sides, e.g.
  [IPRoute](https://man.archlinux.org/man/core/iproute2/ip-mptcp.8.en), the
  [Linux Kernel](https://docs.kernel.org/networking/mptcp.html),
  [mptcpd](https://mptcpd.mptcp.dev/doc/html/), etc.

- MPTCP is natively supported in more programming languages, applications and
  Linux distributions, see this [page](https://www.mptcp.dev/apps.html). To name
  a few: Apache HTTP, cURL, HAProxy, Lighttpd, SystemD, GoLang (the next 1.24
  version will have all stream connections (TCP) supporting MPTCP by default on
  the server side), OpenWrt (enabled by default on the kernel side), etc.

- Various users and companies are then becoming interested in the technology,
  e.g. Marek at CloudFlare wrote a nice technical [blog
  post](https://blog.cloudflare.com/multi-path-tcp-revolutionizing-connectivity-one-path-at-a-time/)
  about the current situation using Ubuntu and a generic 6.8 kernel.

Looking from that side, I should be happy about the results, and force me to
take some (hopefully) deserved time off at some point to finally "_recharge my
batteries_" :)

## What's next?

In the short term, the activities around the second part of December were quite
calm, which allowed me to continue the work on a performance lab dedicated to
MPTCP and focused on tracking regressions linked to its packet scheduler. 2025
didn't continue like the previous year finished: various instabilities in the
tests -- showing real issues in the code, exposed by some recent fixes -- plus
new issues discovered by [Syzbot](https://syzkaller.appspot.com) -- which could
be used to cause DDoS... but requiring root access on the local machine! -- and
having to deal with tax-related admin tasks, forced me to move my focus
elsewhere. Hopefully I can go back to this task very soon!

Please note that I will be present at FOSDEM this weekend. There will be an
[MPTCP Community
meetup](https://fosdem.org/2025/schedule/event/fosdem-2025-6750-mptcp-community-meetup-bof/)
on Saturday at 17.30. Please join, so hopefully the room will have more than 2
people there :-D

In the longer term, quite a lot of work is planned to improve the situation in
many different corner cases. A part of this work would require long
uninterrupted sessions, which is hard to get when maintaining an active kernel
subtree. But I hope to be able to split the work in a way that such sub-tasks
can be interrupted by more urgent ones. I'm also looking at finding people to
help me to realise these tasks quicker.

## Team work

As always, it is important to note that what I presented here so far is mostly
what I was working on. But I'm not alone in this project. For example, Geliang
continued to look at BPF `iter` for the subflows, and having a path-manage
controlled with BPF ; Paolo helped fix some issues found by Syzbot, but also
some improvements on the received path ; Mat helped with the code reviews ;
Christoph continued to fix the SyzKaller infrastructure dedicated to MPTCP.

A great community!

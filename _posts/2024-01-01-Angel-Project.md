---
layout: post
title:  "Angel Project"
---

## Introduction

This is the first post on my new blog, an introduction is then mandatory!

<!--more-->

### Multipath TCP

If you are here, it is very likely you already know about MPTCP. In short,
Multipath TCP is an extension to TCP to improve connection redundancy and
performance by sharing a network data stream across multiple underlying TCP
sessions. It needs to be supported by both ends of the connection. The Linux
kernel supports the second version of the protocol,
[MPTCPv1](https://www.rfc-editor.org/rfc/rfc8684.html). The support started with
version v5.6, and it has progressively continued to get new features along the
new major [releases](https://github.com/multipath-tcp/mptcp_net-next/wiki/#changelog).
This effort is the result of hard work from many people (Mat, Peter, Paolo,
Florian, Christoph, Geliang, Davide, Ossama, Kishen, myself, and many more --
sorry, I couldn't mention everybody, but please check the Git commits),
supported by multiple companies (Intel, Apple, Tessares, RedHat, SUSE, etc.).

### Myself

I'm Matthieu Baerts, one maintainer of MPTCP in the upstream Linux kernel. I was
also one maintainer of a [previous implementation](https://github.com/multipath-tcp/mptcp),
a fork that didn't fit the Net maintainers requirements[^fork]. Lately, my work
around MPTCP was mainly focussed on the maintenance part -- communication with
the community, organising regular meetings, answering questions, tracking,
analysing and fixing bugs, refactoring code to ease the inclusion of new
features or fixes, reviewing and accepting work from others, sending
modifications to be included in the official Linux kernel and doing the
different follow-up -- and also around the tests area: selftests, packetdrill,
and mainly the CI infrastructure. I was doing that as part of my job at
[Tessares](https://www.tessares.net).

[^fork]: Mainly because the modifications were also impacting the "plain" TCP
connections (and more), and that would have impacted a lot of the overall
maintenance work.

### This Angel Project

Multipath TCP in the upstream Linux kernel started with version 5.6. Today, most
features mentioned in the [RFC 8684](https://www.rfc-editor.org/rfc/rfc8684.html)
are supported, with decent performances.

However, the adoption of this technology is slow. I have then decided to
dedicate more time to this project, hoping to lower the threshold for adoption.
My dream is to help to make the Internet more efficient: we can dream :)

To help to realise that, my work is now supported by the
[NGI0 Core](https://nlnet.nl/core/) fund, managed by the
[NLnet Foundation](https://nlnet.nl) (A big thank you :)). NGI0 Core is made
possible with [financial support](https://nlnet.nl/core/acknowledgement.pdf)
from the [European Commission](https://ec.europa.eu/)'s
[Next Generation Internet](https://ngi.eu/) programme, under the aegis of
[DG Communications Networks, Content and Technology](https://ec.europa.eu/info/departments/communications-networks-content-and-technology_en).
This project has received funding from the European Union's
[Horizon Europe](https://ec.europa.eu/programmes/horizoneurope/) research and
innovation programme under grant agreement No 101092990.

I have to admit, my new project didn't have the strong start I was expecting.
There are three explanations for that. The first one is personal, and the reason
behind this project's name: in memory of our Angel (December 2023). The second
one is linked to taxes: apparently I'm the lucky first Belgian individual
grantee getting such funding who wants to make sure there will be no issues with
the future tax declaration! The situation is so particular that all accountants
and financial experts I got in touch with didn't want to assist me because they
couldn't guarantee I would not have issues. I ended up getting some help from a
law firm, hopefully this will be sorted out soon. The last unexpected thing
related to this project is our CI tests that got time limited, meaning we could
not get results by the end of each month.

## November / December report

Most of my work during that period was around the maintenance area, including
a migration of a part of the CI work to validate tests. I admit, nothing
fascinating, but still very important work.

The visual changes are:

- Patches and reviews on Netdev Mailing list: [Lore](https://lore.kernel.org/netdev/?q=f%3Amatttbe%40kernel.org+d%3A20231101..20231231)
- Patches, reviews, meetings, applying patches: [Lore](https://lore.kernel.org/mptcp/?q=f%3Amatttbe%40kernel.org+d%3A20231101..20231231)
- GitHub Actions running tests: [First validations](https://github.com/matttbe/mptcp_net-next/actions/workflows/tests.yml)

### GitHub Actions

What is probably more interesting to describe is the use of GitHub Actions to
run the tests. Previously, we were exclusively using Cirrus CI, for three main
reasons:

- It supports KVM acceleration: it allows us to use debug kconfig like KASAN.
- The logs and artefacts are publicly available: before, tests were running
  exclusively on private instances, only visible by some people and sharing
  results and debug info files was difficult.
- There **was** no blocking limitations: OK, it was running on slow machines,
  with a time limit of 2h per job, with limited storage, etc. but that was OK
  for us.

But [recently](https://cirrus-ci.org/blog/2023/07/17/limiting-free-usage-of-cirrus-ci/),
for understandable -- yet annoying for us -- reasons, the usage is now limited
to a certain number of hours each  month. We tried to find quick workarounds:
reduce the number of builds by reducing the number of sync with netdev tree,
reduce the number of CPUs per job, quickly detect boot errors to stop the job
earlier, etc. But that was not enough. Recently, we were reaching the limit
after two-thirds of a month, leaving us almost 2 weeks without CI to run the
tests.

An easy solution would have been to pay for this service. That would be normal,
this is a service, it uses hardware and energy. But no, and the issue is often
seen in the Open-Source development: the CI is an important part of the project,
but, similar to the maintenance work, nobody wants to pay for it. Of course, it
is beneficial to users, and companies using the technology. But well, that's not
easy to explain to managers to unblock a budget for that, while there is already
a private CI for the company, plus the question: "_why just me/us paying while
there are other users/developers?_".

So we ended up looking at GitHub Actions: it looks like there is no usage
limitation for what we do, but it doesn't support KVM acceleration with our
current plan. Would it be OK without KVM support? We had to test to know. This
is what I did in December, adding a new action file. It runs a docker container
including all required dependences and a script which builds everything, start a
VM and run all tests as before. At the end, artefacts are saved.

When running the tests with a non-debug kernel config, it is fine: it takes a
bit less than one hour to do everything. Could be better, but that's OK for the
moment. But when doing the same with a debug kernel config -- which verifies
many assumptions all the time -- it took a bit less than... 6h! And many
failures in the tests, not supporting such slow environments. So we decided to
keep Cirrus CI for the moment, but just for the debug kernel. Still, we might
move everything to GitHub Actions later, using self-hosted runners, because it
looks like we will still be impacted by Cirrus CI's monthly limit when the
development is active.

The migration to Github Actions was not over in December: there is still some
work to do to get notifications, present test results, etc. But also, running
tests on an even slower environment than before showed new instabilities, and
even kernel crashes! Stay tuned to get more details about that in the future
report!

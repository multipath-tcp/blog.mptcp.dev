---
layout: post
title:  "Summer update"
---

A bit more than 3 months after the last update, I think it is really time to
share some updates about MPTCP! I have been quiet recently, not because I was on
holiday -- even if I should probably take some :) -- but because there were
always higher priority tasks than writing a new blog post which is something I'm
slow at! Bug-fixes, improvements, apps support, conferences, what's next: read
on to find out more about what happened recently!

<!--more-->

## More bug fixes

In my [previous post]({% post_url 2024-07-21-maintainers-responsibilities %}), I
mentioned I was working on fixing issues around the path-manager. A few more
fixes have been added since then, and a lot of backports have been done:
hundreds of commits have been added to the different stable kernels, up to v5.10
when needed. If you see odd behaviour with the MPTCP path-manager, please check
with an up-to-date kernel version first, maybe your issue has already been
fixed!

One important [issue](https://github.com/multipath-tcp/mptcp_net-next/issues/518)
has also been fixed recently: some (weird) middleboxes were causing MPTCP
connections to be reset, because MPTCP options were dropped later on during the
connection, but not at the beginning. When possible, such connections are now
forced to fall back to TCP. Middleboxes are still able to surprise us! :)

## New improvements

Some improvements have also been added during the last few months, e.g.

- If MPTCP is explicitly blocked by a firewall, after 3 unsuccessful connection
  requests, the kernel will fall back to TCP, and future connections will
  directly use TCP for a configurable period of time -- 1 hour by default, then
  it is exponential.

- `SO_KEEPALIVE` socket option -- to send keep-alive packets in case of
  connection inactivity -- has been supported by MPTCP for a few years now. But
  it was not possible to fine tune this behaviour using `TCP_KEEP*` socket
  options. This has been fixed, and exceptionally backported to stable versions,
  because it was not normal to support the main option, but not these other
  ones, this was blocking some deployments, and the code to support these socket
  options is simple and not impacting the rest.

- More `MIB` counters -- visible with `nstat` -- have been added, some of them
  have also been backported: this will help kernel developers and users to
  understand some behaviour like why an expected additional subflow has not been
  established, without having to instrument the kernel.

- The documentation about MPTCP endpoints configuration has been improved to
  avoid confusion. These modifications are visible in the IPRoute manual, and on
  our [website](https://www.mptcp.dev/pm.html).

- The CI now generates code coverage [stats](https://ci-results.mptcp.dev/lcov/)
  (also published on
  [Coveralls](https://coveralls.io/github/multipath-tcp/mptcp_net-next)). Code
  coverage is a valuable piece of information to know how much we can trust a
  test suite, and easily find out what needs to be improved. A majority of the
  code is covered (~90%), which is great! There are some areas that can be
  improved -- e.g.
  [diag](https://github.com/multipath-tcp/mptcp_net-next/issues/524) and [socket
  options](https://github.com/multipath-tcp/mptcp_net-next/issues/525) -- but
  not critical ones. We can also notice many error branches are not covered by
  the MPTCP test suite: that's because that would be a lot of work to cover most
  of them while most of the time the action that has to be taken in this case is
  very simple. That's why we rely on Syzkaller, a fuzzing tool, to cover most of
  these error branches.

- Still about the CI, the BPF MPTCP selftests are now also executed with a debug
  kernel config. More analytic tools will cover this area that is growing.

## More apps natively supporting MPTCP

Thanks to the help from [Anthony Doeraene](https://github.com/Aperence/),
even more [apps](https://www.mptcp.dev/apps.html) are now natively supporting
MPTCP:
- [Apache HTTP](https://httpd.apache.org/) ([v2.5.1](https://svn.apache.org/viewvc?view=revision&revision=1920586))
- [HAProxy](https://www.haproxy.org/) ([v3.1.0](https://git.haproxy.org/?p=haproxy.git;a=commit;h=20efb856e))
- [VLC in iOS](https://www.videolan.org/vlc/) ([v4.0](https://github.com/videolan/vlc-ios/commit/210c88b3e4))
- [Firefox in iOS](https://www.mozilla.org/en-US/firefox/browsers/mobile/ios/) ([v132](https://github.com/mozilla-mobile/firefox-ios/pull/21480))

[NGinx](https://github.com/nginx/nginx/pull/130) and
[FreeNGinx](https://freenginx.org/pipermail/nginx-devel/2024-August/000465.html)
might support it soon too!

An important change as well is on the Go Language: the next version will have
MPTCP enabled by default on the server side.
([src](https://go-review.googlesource.com/c/go/+/607715))

Additionally, Anthony added more languages in the [MPTCP
Hello](https://github.com/mptcp-apps/mptcp-hello) repository: Elixir, Erlang,
Swift (macOS), and Objective-C (macOS). Plus a dedicated page about how to use
MPTCP on [macOS](https://www.mptcp.dev/macOS.html).

Other apps and tools also gained MPTCP support recently:

- [cURL](https://curl.se) ([v8.9.0](https://github.com/curl/curl/pull/13278))
- [dae](https://github.com/daeuniverse/dae) ([v0.8.0](https://github.com/daeuniverse/dae/pull/601))
- [lighttpd](https://www.lighttpd.net/) ([v1.4.76](https://github.com/lighttpd/lighttpd1.4/pull/132))
- [SystemD](https://systemd.io) ([v257](https://github.com/systemd/systemd/pull/32958))
- [v2ray-core](https://github.com/v2fly/v2ray-core) ([v5.17.0](https://github.com/v2fly/v2ray-core/pull/3109))
- [ptcpdump](https://github.com/mozillazg/ptcpdump) ([v0.24.0](https://github.com/mozillazg/ptcpdump/pull/152))

OpenWrt v24 will also have MPTCP support available by default!
([src](https://github.com/openwrt/openwrt/pull/16786))

## Conferences

### Netconf 2024

[Netconf](https://netdev.bots.linux.dev/netconf/) is *a Linux community
conference, by-invitation-only. The agenda has a clear focus on kernel level
networking. Attendees are the main maintainers and developers of the Linux
networking subsystem*.

This year, I was grateful to have been invited to the
[2024](https://netdev.bots.linux.dev/netconf/2024/index.html) edition in Vienna,
and I learned a lot. Even if MPTCP has been invoked, I was mostly there to
discuss [CI aspects](https://netdev.bots.linux.dev/netconf/2024/mattbe.pdf). For
a while now, there is an MPTCP CI validating patches that are shared on the
mailing list. It took quite sometime to put that in place, and it regularly
needs tweaking. There is still some work to do, like having a proper performance
lab in place which is what I'm currently looking at that. On Netdev side, that's
only for less than one year the CI is validating functional tests. A good time
to discuss it, how useful it is, what can be improved, and what is important to
know about it.

We had very useful
[discussions](https://netdev.bots.linux.dev/netconf/2024/notes.html) there.
Amongst the topics that have been discussed around the CI, we have:
- How important it is to know the code coverage.
- A container image would be useful to easily reproduce issues locally.
- A debug kernel config causes a very slow environment, and results in tests to
  be more unstable, but that can maybe be ignored: what is important here is to
  look at new kernel warnings found by the debug tools.
- Other subsystems can re-use code from the MPTCP CI if they want a similar CI
  setup on their side. Still, it would be good to have an "official" service
  applying patches in a Git repository to ease the bootstrap of new CIs.
- Stable kernel are often being validated using the last version of the kernel
  [selftests](https://docs.kernel.org/dev-tools/kselftest.html). In theory, that
  makes sense, and looks good to have the biggest coverage and check for
  regressions, but most network related selftests expect to be executed on the
  kernel version it is linked to, and then fail if a feature is missing. That's
  an issue because older stable kernels have a lot of failed tests due to that,
  and probably nobody is monitoring the different results. Maybe there should be
  an exception for the network tests, because many tests are looking at the
  internals like how packets are created, not only at what is exposed to the
  userspace via the different simple APIs, like the socket one that hides all
  the complexity to the userspace.
- Managing the CI takes time, and resources, plus it might be good to have some
  hardware tests done in the same conditions, from a neutral lab. It might be
  good to create a dedicated entity to share costs.

### LPC 2024

The [Linux Plumbers Conference](https://lpc.events/) (or LPC) is *the premier
event for developers working at all levels of the plumbing layer and beyond*.

The [2024 edition](https://lpc.events/event/18/page/224-lpc-2024-overview) was
also in Vienna, just after Netconf.

There as well, I [talked](https://lpc.events/event/18/contributions/1961/) about
the CI. The Netdev CI is particular: it runs many different tests, while there
are many patches regularly posted on the list. That's why patches are validated
by batches. Due to the complexity -- but also to avoid increasing the traffic on
the Netdev mailing with people sending patches just to fix unworthy details
reported by the CI, instead of checking that upfront -- what is being done by
the CI is supposed to be checked by experienced reviewers and maintainers. It is
then important to agree on what should be advertised, and what to expect from
kernel developers, reviewers and maintainers.

(Note that my [presentation](https://lpc.events/event/18/contributions/1961/)
has been [recorded](https://www.youtube.com/watch?v=ApDDRrrXN9w), but there were
some issues with the microphone at the beginning, apparently.)

## What's next?

Good news, my work will continue to be supported by the [NGI0
Core](https://nlnet.nl/core/) fund -- managed by the [NLnet
Foundation](https://nlnet.nl) (a big thank you :)) -- for another
[round](https://nlnet.nl/project/MPTCP-deployability-II/). NGI0 Core is made
possible with [financial support](https://nlnet.nl/core/acknowledgement.pdf)
from the [European Commission](https://ec.europa.eu/)'s [Next Generation
Internet](https://ngi.eu/) programme, under the aegis of [DG Communications
Networks, Content and
Technology](https://ec.europa.eu/info/departments/communications-networks-content-and-technology_en).
This project has received funding from the European Union's [Horizon
Europe](https://ec.europa.eu/programmes/horizoneurope/) research and innovation
programme under grant agreement No 101092990.

<center>
  <a href="https://nlnet.nl">
    <img src="https://nlnet.nl/logo/banner.png" alt="NLnet foundation logo" width="20%" />
  </a>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <a href="https://nlnet.nl/core">
    <img src="https://nlnet.nl/image/logos/NGI0_tag.svg" alt="NGI Zero Logo" width="20%" />
  </a>
</center>

This support will help to cover the maintenance, and some new features I will
hopefully talk about soon!

## Team work

As always, it is important to note that what I presented here so far is mostly
what I was working on. But I'm not alone in this project. For example, Geliang
is looking at BPF `iter` for the subflows, and having a path-manage controlled
with BPF ; Paolo helped fix some issues found by Syzbot, but also by users
around the path-manager ; Davide switched to a better RST code in case of
middleboxes interference ; Paolo and Mat helped with the code reviews ;
Christoph continued the SyzKaller infrastructure maintenance.

A great community!

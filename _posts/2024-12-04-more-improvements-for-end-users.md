---
layout: post
title:  "More improvements for end users"
---

Here is a small update on what happened recently, like various small
improvements to help the end users, a new mptcpd version and more to come. Read
on to find out more about that!

<!--more-->

## Assisting users

Thanks to the [NLnet foundation](https://nlnet.nl)
[support](https://nlnet.nl/project/MPTCP-deployability-II/), a better focus on
the end user experience is being done. For example:

- The [mptcp.dev](https://www.mptcp.dev) website has been modified to:
  - [List Linux distributions](https://www.mptcp.dev/apps.html#linux-distributions)
    supporting MPTCP on the kernel side, and have`mptcpd` and `mptcpize`
    packages.
  - The [in-kernel path-manager](https://www.mptcp.dev/pm.html#in-kernel-path-manager)
    section has been improved to inform users about automatic configuration:
    [NetworkManager](https://networkmanager.dev/blog/networkmanager-1-40/#mptcp-support)
    1.40 or newer, present in most stable Linux distributions, automatically
    configures MPTCP endpoints with the `subflow` flag ("*client*" mode) by
    default, similar to what `mptcpd` does by default. The manual configuration
    is then likely not needed in most common cases.

- `mptcpd` [website](https://mptcpd.mptcp.dev) has been improved: instead of
  only describing `mptcpd` in a generic (and vague) way, mentioned now is how it
  behaves, how to install it, and where to find the documentation to write new
  plugins. The [README](https://mptcpd.mptcp.dev/README.html) file also lists
  which versions are available in the different Linux distributions.

- [check.mptcp.dev](https://check.mptcp.dev) service now has a clearer message:
  the browser might not support MPTCP, but maybe the system does.

- A new [form](https://github.com/multipath-tcp/mptcp_net-next/issues/new/choose)
  has been added to report issues related to MPTCP: this guides the user to
  provide useful info, and avoid a few back-and-forth.

## `mptcpd` 0.13

A new version has recently been released, including:
- Support for [ELL](https://git.kernel.org/pub/scm/libs/ell/ell.git/) >= 0.68.
- Documentation improvements for the `check_route` address notification flag.
- Optional listening socket creation for user space path manager plugins.
- Support for listener events.
- A new `/usr/libexec/mptcp-get-debug` script to help simplify information
  collection for MPTCP related bug reports, not just for `mptcpd` alone.
- A fix for a crash in the tests when `/etc/protocols` is not available.

This new version is already available in a few Linux distributions, e.g.
ArchLinux, Debian, Fedora, and Gentoo.

Please note that `mptcpd` is getting proposed for inclusion in
[NixOS](https://github.com/NixOS/nixpkgs/pull/355928) and
[AlpineLinux](https://gitlab.alpinelinux.org/alpine/aports/-/merge_requests/75529).
Do not hesitate to help with the review! Similarly, if `mptcpd` is not packaged
in your distribution or is outdated, do not hesitate to help with the packaging
:-)

## Bug-fixes and small improvements

Similar to the [previous post]({% post_url 2024-10-28-summer-update %}), a few
more issues have been fixed, and a few additions have been done on the kernel
side: `ip mptcp endpoint` was wrongly restricted to the net administrators, some
fixes have been added for issues coming from very uncommon paths and reported by
[Syzbot](https://syzkaller.appspot.com), the MPTCP endpoints are now dumped
while doing a lockless list traversal.

Please note that an external security audit of MPTCP code in the kernel was
completed a few months ago, thanks to the [NLnet foundation](https://nlnet.nl)
[support](https://nlnet.nl/project/MPTCP-deployability/). We are happy that
[RadicallyOpenSecurity](https://radicallyopensecurity.com/) performed this
audit. The report can be found [here](/assets/202406-mptcp-security-report.pdf).
Only low and unknown thread issues have been reported. After investigation, it
looks like no further action is required. The 'low' thread one would require
quite a few changes, something different from what is usually done on the kernel
side, and potentially introducing new issues. The other issues were marked as
false-positive ones. We would like to thank ROC, NLnet and the EU commission for
this report!

## What's next?

A performance lab is being prepared. Many regression tests are in place, and
automatically validated to make sure all the features continue to work as
expected. These tests are [covering](https://ci-results.mptcp.dev/lcov/export/)
a major part of the code, but only a very small part tries to cover the
performance side. Plus this is done from the same VM, with a high tolerance.
Proper performance tests are done manually so far. By not being tracked
regularly, it makes performance regressions harder to fix and easier to miss. It
is then required to automate the performance tests and track regressions.

A new lab is being setup, using multiple VMs to generate traffic. The goal is to
start with a fully virtual environment that can be setup on the same machine to
start, but reusable on bare metal later on.

Linked to that, a few improvements have been added on the `virtme-ng` side: it
is now possible to set multiple
[network](https://github.com/arighi/virtme-ng/pull/194)
[interfaces](https://github.com/arighi/virtme-ng/pull/195), but also a simple
way to get a [remote](https://github.com/arighi/virtme-ng/pull/197)
[console](https://github.com/arighi/virtme-ng/pull/198) to existing VM using a
[`vsock`](https://wiki.qemu.org/Features/VirtioVsock)!

## Team work

As always, it is important to note that what I presented here so far is mostly
what I was working on. But I'm not alone in this project. For example, Geliang
continued to look at BPF `iter` for the subflows, and having a path-manage
controlled with BPF ; Paolo helped fix some issues found by Syzbot, but also
some improvements on the received path ; Mat helped with the code reviews ;
Christoph continued to fix the SyzKaller infrastructure dedicated to MPTCP.

A great community!

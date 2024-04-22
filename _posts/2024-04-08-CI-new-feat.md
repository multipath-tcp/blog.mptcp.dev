---
layout: post
title:  "CI & new features"
---

The [previous post]({% post_url 2024-03-04-backports %}) mentioned that February
was still full of various "maintenance" tasks, mainly around the backports, and
the preparation of the future Linux 6.9. The beginning of March was similar to
that, then more time was finally available to look at fixing issues, and
preparing new features. Read on to find out more about what happened in March!

<!--more-->

## The future v6.9 and backports

Linux v6.8 was released on March 10th. As mentioned in my [previous post]({%
post_url 2024-03-04-backports %}), we had up to this date to suggest new
features, and refactoring to be included in `net-next` tree before being closed
for new submissions. We took this opportunity to send a last feature for the
future v6.9 (`TCP_NOTSENT_LOWAT` socket option support from Paolo) one week
before, and a bunch of refactoring in the selftests initiated by Geliang, a few
days before the limit. We usually don't like to rush things just before the
closure, but it generally helps to reduce the maintenance cost to send big
refactoring early, than having to carry it only in our tree for a bit of time.

This has been done while in parallel, I was also helping the stable team
[backporting]({% post_url 2024-03-04-backports %}) even more patches which could
not be applied without conflicts in stable versions. Pretty much the same as
what was done in February, indeed, not that interesting then :)


## CI: a big step forward

With more available time, this allows me to work on the long awaited tasks
linked to the CI:
- Using [runners with KVM support](https://github.com/multipath-tcp/mptcp_net-next/issues/474).
- Validating [MPTCP BPF tests](https://github.com/multipath-tcp/mptcp_net-next/issues/406).
- Switching to [`virtme-ng`](https://github.com/multipath-tcp/mptcp_net-next/issues/472).
- Tracking regressions by [publishing tests results](https://github.com/multipath-tcp/mptcp_net-next/issues/473).

### GitHub Actions and KVM support

Back in [December]({% post_url 2024-01-01-Angel-Project %}), when the switch to
GitHub Actions started, it was not possible to enable KVM support with public
runners. That was the main reason behind choosing [Cirrus CI](https://cirrus-ci.org/)
a few years ago, and keeping it for the tests with the debug kernel config a few
months ago. As described in the [previous post]({% post_url 2024-01-01-Angel-Project %}),
our workflow was impacted by Cirrus CI's monthly limit, and it was the reason
behind this partial switch to GitHub Actions. Moving only the tests with a
non-debug kernel config was not enough, we were still impacted by that: the
monthly limit was reached on the 31st of January, and on the 16th of February.
Another solution was then required.

I was then looking at adding a self-hosted runner. I managed to
[successfully](https://github.com/matttbe/mptcp_net-next/actions/runs/8194936484)
execute the tests on a self-hosted runner which was a refurbished mini PC at
home. I then realised that was not enough: KVM was still not used, because the
docker image is not executed with enough permissions (`--privileged`, or
`--cap-add` + `mount`).

I knew from a [GitHub blog post from last year](https://github.blog/changelog/2023-02-23-hardware-accelerated-android-virtualization-on-actions-windows-and-linux-larger-hosted-runners/)
that it was possible to have KVM support, so I tried to find a way to use it
with our "Docker container actions", like they do in
[reactivecircus/android-emulator-runner](https://github.com/reactivecircus/android-emulator-runner).
Then I found out that since [January this year](https://github.blog/2024-01-17-github-hosted-runners-double-the-power-for-open-source/),
it is possible to have KVM support with the Linux public GitHub runners! So no
need to host and maintain that at home with a limited Internet connection! Plus
it means there is no need to restrict these tests to patches sent on our mailing
list, people can have results from the CI simply by sending code to their GitHub
fork repo!

So I:
- [Enabled KVM support](https://github.com/multipath-tcp/mptcp_net-next/commit/677b5ecd223ca1a39e993dfd0138f32420521d26)
  with a "workaround" (Docker is launched manually)
- [Added the 'debug' mode support](https://github.com/multipath-tcp/mptcp_net-next/commit/6c0b56e647b611e902ffacb958eb7443009f0ef2)
- [Removed Cirrus-CI support](https://github.com/multipath-tcp/mptcp_net-next/commit/cc356e6ad19f66c50a97e7829e7031bbb5b7f199)
- (And did other [clean-ups](https://github.com/multipath-tcp/mptcp_net-next/commits/t/DO-NOT-MERGE-mptcp-add-CI-support/.github/workflows?author=matttbe&since=2024-03-01&until=2024-03-31)
  while at it)

With KVM support, the CPU usage is reduced and no longer near the 100% limit, so
our tests are more stable. Dropping Cirrus-CI support with a bunch of pretty
much duplicated code is helpful for the maintenance in the long term.

### BPF Tests
MPTCP BPF tests are present in the Linux kernel since 2022 (they were already in
our tree in August 2020, but the development got interrupted). Back then, the
tests were limited to the available features: being able to read fields from an
MPTCP socket and checking if a TCP socket is an MPTCP subflow. With this, it is
possible to monitor MPTCP connections, and even interact with them, e.g. by
changing socket options per subflow. Later,
[`mptcpify`](https://git.kernel.org/pub/scm/linux/kernel/git/bpf/bpf-next.git/commit/?id=ddba122428a7)
BPF program has been added to force the creation of MPTCP sockets instead of TCP
ones.

Until recently, these tests -- and the ones for the work-in-progress MPTCP BPF
packet schedulers -- were not validated by our CI. We didn't track regressions
in this area. With the help of Geliang, our CI scripts have been
[adapted](https://github.com/search?q=repo%3Amultipath-tcp%2Fmptcp-upstream-virtme-docker+bpf&type=commits)
to run these tests. Recently, I added a
["matrix" support](https://github.com/multipath-tcp/mptcp_net-next/commit/71a9e1d223e484148778e2549adbf18a6abecf8a)
on GitHub Action to be able to run these tests requiring more kernel config
options in a dedicated runner.

### Virtme NG
[Virtme](https://github.com/amluto/virtme/) is very useful to quickly run a VM
with a custom kernel, and using the file system of the host (or in our case, the
one of a container containing all required dependences). We have been using it
since 2019, and we were happy with it.

In 2020, it looks like this Virtme project started to get unmaintained. In
December 2022, we had to [patch it](https://github.com/amluto/virtme/pull/82) to
support kernels >= 6.2. More recently, another
[patch](https://github.com/amluto/virtme/pull/81) was required to support QEmu >=
7.2. Andrea Righi started to gather different fixes on
[his side](https://github.com/arighi/virtme/), before creating the
[`virtme-ng` project](https://github.com/arighi/virtme-ng/) in 2023.

`virtme-ng` brings interesting features introduced in this nice
[LWN article](https://lwn.net/Articles/951313/). Switching to it would reduce
the boot time, and reduce a lot the I/O thanks to
[`virtiofs`](https://virtio-fs.gitlab.io/). So that's what we did
[recently](https://github.com/multipath-tcp/mptcp-upstream-virtme-docker/commit/0c54a948e22669d265b4ef083080e0f0af3ffe6f). It should also help us for the long
term maintenance.

### Tracking regressions

Since we use a public CI, results are simply published on an IRC channel
([#mptcp-ci](https://web.libera.chat/?#mptcp-ci)). This is not really easy to
track regressions.

[Publish Test Results](https://github.com/marketplace/actions/publish-test-results)
GitHub Action has been added, but it doesn't keep a long history of results.

A new ["Flakes"](https://ci-results.mptcp.dev/flakes.html) has then been created
to help us to track unstable tests. It is similar to
[Netdev's Flakes](https://netdev.bots.linux.dev/flakes.html) page (with
[dark scheme support](https://github.com/linux-netdev/nipa/pull/17) :) ).

It is a shame such service is not better integrated in GitHub Actions. In a
perfect world where tests are all stable, it should not be needed. But here,
when hosts need to talk to each other, packets can be delayed for some reason,
causing retransmissions, etc. It is not easy to predict everything. The
[cURL](https://curl.se/) project is using
[TestClutch](https://github.com/dfandrich/testclutch/), but it is an external
service to deploy, and it doesn't support the TAP format yet.

## What's next?

Big work has been started to rewrite [mptcp.dev](https://www.mptcp.dev) website.
When working on adding native MPTCP support to apps like
[lighttpd](https://github.com/lighttpd/lighttpd1.4/pull/132) and
[curl](https://github.com/curl/curl/pull/13278), it was clear that a website
gathering all required info to know about MPTCP to set it up, and to add its
support in apps were missing. (*Note: our website was updated on the 18th of
April, it was looking like
[this](https://github.com/multipath-tcp/mptcp.dev/blob/531801e/README.md)
before.*)

Publishing a doc in the kernel official documentation will also help end-users
and app developers.

In terms of developments, the next priorities are adding
[missing features](https://github.com/golang/go/issues/56539#issuecomment-1940486340)
to have MPTCP enabled by default in Go.


## Team work

As always, it is important to note that what I presented here so far is mostly
what I was working on. But I'm not alone in this project. For example, Geliang
continued to do some clean-ups in the KSelfTests, looked at the MPTCP
support in [IPerf3](https://github.com/esnet/iperf/pull/1661), and started to
look at adding "last time" counters in `MPTCP_INFO`. Mat and Paolo helped with
the reviews, and Christoph looked at running fuzzing tests on top of the last
RHEL kernel.

A great community!

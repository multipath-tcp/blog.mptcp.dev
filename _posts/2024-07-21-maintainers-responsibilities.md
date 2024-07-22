---
layout: post
title:  "Maintainers responsibilities"
---

Last month, I didn't publish any new blog post here. Not because there was
nothing to say, but simply because I was busy. Sadly, not 100% focused on the
new tasks I wish to finish implementing, but mainly focused on resolving issues
discovered while working on these new features. Should I have closed my eyes and
carried on? Can maintainers do that? Read on to find out more about what
happened recently!

<!--more-->

## Maintainers responsibilities

"Maintenance" is a general term which, for a kernel maintainer of an active
subtree, includes: communication with the community, organizing regular
meetings, answering questions, tracking, analysing and fixing bugs, fixing
issues with anything related to the workflow like the CI and other tools and
services, refactoring code to ease the inclusion of new features or fixes,
reviewing and accepting work from others, sending modifications to be included
in the official Linux kernel, helping with the backports, doing the different
follow-up, and I probably missed other tasks. It might not look like it, but the
maintenance work in the kernel can be quite time-consuming. Some "small" tasks
can quickly take a few hours, e.g. reviewing non-straightforward code, or
analysing bug reports.

I already tried to demonstrate some of these aspects in my previous blog posts.
Here, I will focus on the responsibilities related to bugs discovered while
working on new features.

### Discovering new bugs

When bugs are discovered while working on something else, there are typically a
few possibilities: ignoring, documenting, or fixing them.

- I don't know if it is due to my personality, or because of maintainers' duty,
  but I would feel bad ignoring them without doing anything else. When someone
  is new to a project, it might not be clear if something looking strange is
  really a bug or not. But if it is someone who maintains the code, it is
  clearer when something is not right. It is then hard not to think about the
  consequences in the mid or long term, and ignore issues that will come back
  sooner or later, with possibly more pressure, or bad consequences.

- Documenting the issue can be a "quick" solution. Even if sometimes,
  documenting issues can take almost as long as resolving them: the focus will
  be on the issue, it is normal to already think about solutions, then why not
  trying to fix it while everything is still "fresh" in mind. But sometimes,
  there are some urgencies, the bug resolution can be long, or the priority
  can be too low.

- Fixing bugs would be ideal. But fixing bugs also means understanding them by
  analysing code, reproducing them by adding a regression test, and documenting
  them by providing all required details in commit messages. Often in such
  projects, this is not done in 5 minutes.

### Recent examples

Recently, I was working on documenting how the [MPTCP default
Path-Manager](https://www.mptcp.dev/pm.html) is working, and improving the user
experience. It is clear to me, I could not simply ignore issues I found while
working on that. But also, documenting that "_something is supposed to work like
that, but don't in some cases_" was feeling wrong.

I then look at the first bug, then, as often in these cases, it was like opening
Pandora's box: one bug after another. The result was the creation of 30+ kernel
patches, a better documentation, resolving a few issues reported by users but
not understood at that time with the provided info, etc. But also a clearer, and
more predictable software behaviour, which improves the user experience at the
end.

Here are two other examples with tests suites. The first one is with the MPTCP
CI which [reported](https://ci-results.mptcp.dev/flakes.html) a few unstable
tests over the last few months. They all used the same tools, and even if the
errors were quite rare, they were happening with different tests. Because of
that, developers started to lose faith in them: in case of error, it is no
longer a sign of an issue with the new code. After a bit of time, developers
might even not look at the errors any more, blaming the tests instead, and
possibly missing real problems. A short term solution was to re-launch the
tests, and consider them as problematic if they were failing twice in a row.
That can be OK to do that in some specific cases, but it also means, real
issues that only happened in some conditions might be missed as well. Fixing the
root cause seems more rewarding, and better in the long term. That's what has
been [done](https://github.com/multipath-tcp/packetdrill/pulls?q=is%3Apr+is%3Aclosed)
recently with MPTCP Packetdrill tests. It is good to have a trusted test suite!

In the second example, another CI, the [Netdev
one](https://netdev.bots.linux.dev/status.html), reported that some specific
subtests were unstable. They have been unstable only there, probably because too
many tests are being executed at the same time. The issues have been tracked and
documented, they still need further investigation, but it looks like it is
either an issue with the test itself, or the fixes seem more like non-trivial
optimizations. So either a debatable low priority, or an important work. In such
cases, it has been decided to clearly mark these tests as "unstable", and not as
"error". By doing that, it "_reduces the noise_", and helps new developers not
understanding why their modifications caused some unrelated issues. Yet, it is
still important to only mark the ones that had a first analysis, and track their
evolution. That's what has been [done](https://lore.kernel.org/mptcp/20240524-upstream-net-20240524-selftests-mptcp-flaky-v1-0-a352362f3f8e@kernel.org/)
recently with MPTCP selftests.

## Team work

As always, it is important to note that what I presented here so far is mostly
what I was working on. But I'm not alone in this project. For example, Geliang
helped to reduce duplicated code in BPF selftests, including the MPTCP ones ;
Davide replaced a few unintentionally discriminated words from the comments in
the code ; Yonglong Li fixed bugs with MIB counters ; Gregory continued his
experimentations with the packet scheduler API ; Paolo and Mat helped with the
code reviews ; Christoph continued the SyzKaller infrastructure maintenance.

A great community!

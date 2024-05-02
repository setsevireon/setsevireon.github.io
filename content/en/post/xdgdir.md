---
title: A Lawless Land Ruled by Dots
date: 2021-07-28T09:13:32-03:00
tags: [opinion, xdg, homedir, dotfiles]
author: Karlmarx
description: About XDG Base Directory and Directory Standards
---

Everybody (maybe not) loves standards. It makes life much easier as you don't
have to make unnecessary choices and helps to keep compatibility between
software. The problem is a paradox, we like standards, we just don't like the
ones that already exist, what we really want, is to [create a new, better and
universal][2] one, and if others do not like, they are just wrong.

When it comes to directory hierarchy, we are well-served. I know, [FHS][3] is
not equally implemented across all distros, nonetheless, you can, mostly,
expect things to be where they should. At least system wide, where it is really
important.

The user home directory, on the other hand, is a lawless land, ruled by dots.
It is difficult to distinguish what is configuration, data, local binaries and
libraries or temporary files. I must admit, It bugs me to have a bunch of
dotfiles scattered over my homedir to the point of writing a blog post about it.

[**Enters XDG Base Directory Specification**][4].

Using `$XDG_DATA_HOME`, `$XDG_CONFIG_HOME` and `$XDG_CACHE_HOME` has become
somewhat common. Many applications now support it, to other it is the default.

So, problem solved, right?

Not quite. There is, still, resistance from project maintainers to add support
to it, some of then have even ruled out this possibility. Mostly they are
fossilized developers, with fossilized ideas, and *no youngster should
tell them what to do* (a bit too harsh?).

There is a good reason not to hard-code user configuration, while the system
should be as agnostic to user's preferences as possible, user's home should be
adaptable to each ones workflow and organizational system. I don't think XDG
solution is the ideal one, but is the best attempt so far.

[2]: https://xkcd.com/927/ "How Standars Proliferate"
[3]: https://www.pathname.com/fhs/ "Filesystem Hierarchy Standard"
[4]: https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html "XDG Base Directory Specification"

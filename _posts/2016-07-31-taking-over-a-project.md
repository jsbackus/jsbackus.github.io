---
layout: post
title:  "Taking Over A Project"
date:   2016-07-31 21:00:00
categories: FOSS Fedora
---

These days life doesn't leave me time for much outside of work and family. It
is funny how one's priorities can shift so suddenly and drastically, all within
the blink of an eye. But, hey, that's life.

An interesting thing happened, though. In my almost-hiatus from FOSS and
Fedora, the upstream for one of the packages I maintain ... disappeared.
I don't mean became abandoned or unresponsive. I mean actually and completely
disappeared.

See, this project was hosted on GitHub as a personal project and the
original developer was the sole maintainer of this project. Which was fine,
particularly since this individual was more responsive than even some large
projects with a headcount that rivals most mid-sized companies.

However, one day this individual had had enough and deleted the whole repository.
I won't go into the reasons, and they actually had little to do with the
project, but they were sufficient to cause this individual to wash their
hands of the whole matter and be done with it. This is understandable. I'm
sure most of us have been pushed to the point of throwing our hands up in the
air and saying "Fine. Screw it. I'm moving on."

The problem is that with GitHub, when you delete a repository, it all goes.
The code, the wiki, the issues -- *all* of it. It doesn't matter if you are
the origin of 70+ forks. When you delete a repo, it all goes.

This just goes
to show how important it is that Fedora maintains source RPMs as well as the
binary ones. It also goes to show how important it is to maintain your own
repo of any projects you deem critical. Luckily, Git makes doing so ridiculously
easy and GitHub makes it easy to do so for the code and for the wiki. However, for
the issues you are SOL.

Believe it or not, the torching of a moderately popular project isn't the
most interesting part of this story. People noticed when this project
disappeared. And some of these people raised an issue on one of the other
projects this developer had on GitHub. Others of us noticed and joined
the conversation, even long after the developer stopped responding.

Several of us had fairly recent copies of the repo. Mine was current up through
the last official release. But we were still missing quite a few commits. (As
I said, this individual was very active). A handful of us decided to take what
we had and set up an official fork as an organization. As we worked, more
people came forward with their forks. We were able to rebuild the repository
with all but two commits.

... And then the ridiculous happened. One individual was able to resurrect
those two commits by downloading the patches using the URLs in the developer's
activity stream. ... And then someone else was able to track down a cached
version of the wiki before it was purged.

And there we were - a completely resurrected repository and wiki, all because
of the efforts of a community that was heartbroken because a program they
used frequently had disappeared.

And thus, we soldiered on. I'm proud to say that we just
[released](https://github.com/AntiMicro/antimicro/releases) our first
official version as an organization. It includes a few bug fixes and updated
links, but, hey, it is a start.

This is why open source is so awesome. When a
closed source developer decides "Fine. Screw it. We're moving on," there is
little that the community can do but send e-mail upon e-mail to an already
haggard worker-bee, who inevitably ships those e-mails off to the junk
folder. But with open source, a community can rise up out of the rubble
of an abandoned program and thrive.

Assuming, someone else kept the source that is. So, the moral of the story
is: keep copies of all the materials you can. Be it code, wiki pages, or even
the issue-related e-mail stream.

Happy Hacking!

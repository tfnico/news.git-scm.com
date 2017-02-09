---
layout: default
title: Git Rev News Edition 1 (March 25th, 2015)
navbar: false
---

## Git Rev News: Edition 1 (March 25th, 2015)


Welcome to the first edition of [Git Rev News](https://git.github.io/rev_news/rev_news/),
a digest of all things Git, written collaboratively 
on [GitHub](https://github.com/git/git.github.io) by volunteers. 

Our goal is to aggregate and communicate
some of the activities on the [Git mailing list](mailto:git@vger.kernel.org)
in a format that the wider tech community can follow
and understand. In addition, we'll link to some of the interesting Git-related
articles, tools and projects we come across.

This edition covers what happened during the month of March 2015.

You can contribute to the upcoming edition by sending [pull
requests](https://github.com/git/git.github.io/pulls) or opening
[issues](https://github.com/git/git.github.io/issues).

## Discussions

### General

* [Promoting Git developers](http://thread.gmane.org/gmane.comp.version-control.git/265165)  (*written by Junio C Hamano*)

Following up to a message from David Kastrup, a discussion was initiated
by Christian Couder where Michael J Gruber noted
that there are three classes of Git developers, and it is hard for
people who are not already sponsored by corporations to work on Git
not as a hobbyist, but as a means to gain either monetarily or build
reputation.

Later in the thread, Christian volunteered to start a newsletter (this
newsletter!) to summarize and publicise works by participants and
Thomas Ferris Nicolaisen, who runs a popular podcast GitMinutes,
offered to help.

Also, it was decided that release notes will list new and returning
contributors and thank for their help for each release.

### Reviews

* [make git-pull a builtin](http://thread.gmane.org/gmane.comp.version-control.git/265628/)

Paul Tan, who will probably apply to be a [Google Summer of Code
(GSoC)](https://developers.google.com/open-source/soc/) student for
Git this year, anticipated the start of the GSoC and sent a patch to
rewrite git-pull.sh as a built-in.

Indeed as stated in [the GSoC idea
page](https://git.github.io/SoC-2015-Ideas.html):

```
Many components of Git are still in the form of shell and Perl
scripts. While this is an excellent choice as long as the
functionality is improved, it causes problems in production code – in
particular on multiple platforms, e.g. Windows (think:
POSIX-to-Windows path conversion issues).
```

This is why the "Convert scripts to builtins" GSoC project idea is to:

```
... dive into the Git source code and convert a couple of shell
and/or Perl scripts into portable and performant C code, making it a
so-called "built-in".
```

It appeared that two developers, Duy Nguyen and Stephen Robin, had
already worked on converting git-pull.sh into a built-in in the
past. This happens quite often, so it is a good idea before starting
to develop something in Git, to search and ask around.

Johannes "Dscho" Schindelin, one of the Git for Windows developers, 
thanked Paul for the very detailed and well researched comments in 
his patch and commented a bit on the benefits of this kind of work:

```
> on Windows the runtime fell from 8m 25s to 1m 3s.

This is *exactly* the type of benefit that makes this project so important! Nice one.

> A simpler, but less perfect strategy might be to just convert the shell
> scripts directly statement by statement to C, using the run_command*()
> functions as Duy Nguyen[2] suggested, before changing the code to use
> the internal API.

Yeah, the idea is to have a straight-forward strategy to convert the scripts in as efficient manner as
possible. It also makes reviewing easier if the first step is an almost one-to-one translation to
`run_command*()`-based builtins.

Plus, it is rewarding to have concise steps that can be completed in a timely manner.
```

Regarding the test suite, Matthieu Moy suggested:

```
> Ideally, I think the solution is to
> improve the test suite and make it as comprehensive as possible, but
> writing a comprehensive test suite may be too time consuming.

time-consuming, but also very beneficial since the code would end up
being better tested. For sure, a rewrite is a good way to break stuff,
but anything untested can also be broken by mistake rather easily at any
time.

I'd suggest doing a bit of manual mutation testing: take your C code,
comment-out a few lines of code, see if the tests still pass, and if
they do, add a failing test that passes again once you uncomment the
code.
```

* [Forbid "log --graph --no-walk"](http://thread.gmane.org/gmane.comp.version-control.git/264899/)

Dongcan Jiang, who will also probably apply to be a GSoC student for
Git this year, sent a patch to prevent `git log` from being used with both
the `--graph` and the `--no-walk` option. He sent this patch because
the Git community asks potential students to work on a
[microproject](https://git.github.io/SoC-2015-Microprojects.html) to
make sure that they can work properly with the community.

Eric Sunshine made some good general suggestions that are often made
on incoming patches:

```
> Forbid "log --graph --no-walk

Style: drop capitalization in the Subject: line. Also prefix with the
command or module being modified, followed by a colon. So:

    log: forbid combining --graph and --no-walk

or:

    revision: forbid combining --graph and --no-walk

> Because --graph is about connected history while --no-walk is about discrete points.

Okay. You might also want to cite the wider discussion[1].

[1]: http://article.gmane.org/gmane.comp.version-control.git/216083

> revision.c: Judge whether --graph and --no-walk come together when running git-log.
> buildin/log.c: Set git-log cmd flag.
> Documentation/rev-list-options.txt: Add specification on the forbidden usage.

No need to repeat in prose what the patch itself states more clearly
and concisely.

Also, such a change should be accompanied by new test(s).
```

René Scharfe and Junio also suggested some improvements especially in the code.

Junio also explained the interesting behavior of `git show` depending
on the arguments it is given:

```
When "git show" is given a range, it turns no-walk off and becomes a
command about a connected history.  Otherwise, it is a command about
discrete point(s).
```

* [War on broken &&-chain](http://thread.gmane.org/gmane.comp.version-control.git/265874) (*written by Junio C Hamano*)

We often see review comments on test scripts that points out a
breakage in the &&-chain, but what does that mean?

When you have a test that looks like this:

```
test_expect_success 'frotz and nitfol work' '
	git frotz
	git nitfol
'
```

the test framework will not detect even if the `frotz` command
exits with a non-zero error status.  A test must be written like
this instead:

```
test_expect_success 'frotz and nitfol work' '
	git frotz &&
	git nitfol
'
```

Jeff resurrected a clever idea, which was first floated by Jonathan
Nieder in October 2013, to automate detection of such an issue.  The
idea is to try running the test body with this magic string prefixed:
`(exit 117) &&`.  If everything is properly chained together with `&&`
in the test, running such a test will exit with error code 117 (and
the important assumption is that error code is unused elsewhere). If
on the other hand, if there is an breakage, the test prefixed with the
magic string would either succeed or fail with a code different from
117.  His work caught many real test breakages but fortunately it was
found that no command being tested was failing ;-)

As the variety of platforms each developer has access to and the time
each developer has to test Git on them is inevitably limited, broken
&&-chains in some tests that Jeff didn't run were expected to be
caught by others.  Torsten Bögershausen did find one in a test that
runs only on a case insensitive filesystem a few days later.


### Support

* [Git with Lader logic](http://thread.gmane.org/gmane.comp.version-control.git/265679/)
 
People often ask the Git mailing list whether they can use Git to manage
special content. Fortunately, the list is monitored by many experts in different
domains who can often provide specific answers.

This time Bharat Suvarna asked about [PLC
programs](http://en.wikipedia.org/wiki/Programmable_logic_controller)

Kevin D gave the usual non-specific answer:

```
Although git is not very picky about the contents, it is optimized to
track text files. Things like showing diffs and merging files only works
on text files.

Git can track binary files, but there are some disadvantages:

- Diff / merge doesn't work well
- Compression is often difficult, so the repository size may grow
  depending on the size of the things stored
```

Then Randall S. Becker gave a more specific answer:

```
Many PLC programs either store their project code in XML, L5K or L5X (for
example), TXT, CSV, or some other text format or can import and export to
text forms. If you have a directory structure that represents your project,
and the file formats have reasonable line separators so that diffs can be
done easily, git very likely would work out for you. You do not have to have
the local .git repository in the same directory as your working area if your
tool has issues with that or .gitignore. You may want to use a GUI client to
manage your local repository and handle the commit/push/pull/merge/rebase
functions as I expect whatever PLC system you are using does not have git
built-in.
```

Doug Kelly also gave some interesting specific information and pointed to a
web site more information.

Thank you all for these helpful answers!

## Misc Git News

### Developers

* David Kastrup (dak at gnu.org) as of version 2.1.0 reimplemented significant
parts of "git blame" for a vast gain in performance with complex
histories and large files. As working on free software is his sole
source of income, please consider contributing to his remuneration if
you find this kind of improvements useful.

### Events

* [Git Merge 2015](http://git-merge.com/), The Conference for the Git
Community, will take place on April 8th & 9th in Paris, France, thanks
to GitHub and the sponsors. If you are a Git developer and need travel 
or other assistance to go there, please contact Peff, aka
Jeff King &lt;<peff@peff.net>&gt;.

### Releases

* [Git v.2.3.2](http://permalink.gmane.org/gmane.linux.kernel/1902789), March 6th.

```
The latest maintenance release Git v2.3.2 is now available at
the usual places.
```

* [Commandline GUI-tool *tig* version 2.1](http://article.gmane.org/gmane.comp.version-control.git/265298), March 11th:

```
I just released version 2.1 of Tig which brings a lot of improvements to speed
up usage in large repositories such as the Linux kernel repo (see improvements
related to #310, #324, #350, and #368). Else this release brings minor
improvements across the board plus a fair amount of bug fixes.
```

* [NodeGit 0.3.0](https://github.com/nodegit/nodegit/releases/tag/v0.3.0), March 13th:

```
Updated to libgit2 v0.22.1. This release contains breaking API changes. Most 
noteworthy is the change to how certificate errors are handled during authentication.

For more details check out the change log: http://www.nodegit.org/changelog/#v0-3-0
```

* [Git v.2.3.3](http://permalink.gmane.org/gmane.linux.kernel/1908634), March 14th:

```
The latest maintenance release Git v2.3.3 is now available at
the usual places.  It is comprised of 26 non-merge commits since
v2.3.2, contributed by 11 people, 1 of which is a new contributor.
```

* [Git v2.3.4](http://permalink.gmane.org/gmane.linux.kernel/1915412), March 23rd:

```
The latest maintenance release Git v2.3.4 is now available at the
usual places.  It is comprised of 22 non-merge commits since v2.3.3,
contributed by 9 people, 1 of which is a new face.  All these fixes
have already been in the 'master' branch for some time.
```

### From outside the list

* Google Code [announced their shutdown](http://google-opensource.blogspot.de/2015/03/farewell-to-google-code.html), offering all remaining projects easy migration to Git/GitHub. 
* Nava Whiteford wrote a popular article on the [most common Git screwups/questions and solutions](http://41j.com/blog/2015/02/common-git-screwupsquestions-solutions/).
* An older but related resource: [Flight rules for Git](https://github.com/k88hudson/git-flight-rules). 
* An [interview with Git book "Git in Practice" author, Mike McQuaid](http://www.infoq.com/articles/interview-Mike-McQuaid-git-practice).
* Speaking of Git books, there's  a new one on the way: [Git Essentials](https://www.packtpub.com/application-development/git-essentials).
* And there's [a new video course out on mastering Git](https://www.packtpub.com/application-development/mastering-git-video). The author was just interviewed on....
* The [latest episode of the podcast  GitMinutes](http://episodes.gitminutes.com/2015/03/gitminutes-33-thom-parkin-on-mastering.html), about mastering Git.
* Here's a [Git hook](https://github.com/naholyr/github-todos/wiki/Full-presentation) which will automatically turn new TODOs into tasks in your issue tracker (if you use GitHub issues).
* Both [GitLab](https://about.gitlab.com/2015/03/19/omnibus-gitlab-openssl-security-release/) and [BitBucket](https://blog.bitbucket.org/2015/03/19/openssl-security-advisory/) recently posted advisories on recent OpenSSL vulnerabilities.
Make sure that your Git or otherwise OpenSSL-based services are not affected!

## Credits

This edition of Git Rev News was curated by Christian Couder &lt;<christian.couder@gmail.com>&gt; and Thomas Ferris Nicolaisen &lt;<tfnico@gmail.com>&gt; with help from Junio Hamano, Matthieu Moy, Jeff King, David Kastrup, Stefan Beller and Alex Vandiver.


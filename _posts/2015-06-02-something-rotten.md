--- 
title: "Something is rotten in the state of Denmark"
status: publish
layout: post
published: true
type: post
tags:
- R
- CRAN
- packages
active: blog
category: R
alert: "I have modified <i>slightly</i> the juxtaposition of Oliver Keyes' tweet and the subsequent comments about broken vignettes (and expanded it to packages to further distance this from the tweet). It was not my intention to link Oliver, via his tweet, with poor packagers. I only used the tweet to exemplify a common complaint I hear."
---

On Twitter and elsewhere there has been much wailing and gnashing of teeth for some time over one particular aspect of the R ecosphere: [CRAN](http://cran.r-project.org/). I'm not here to argue that everything is peachy --- far from it in fact --- but I am going to argue that the problems we face *do not* begin and end with CRAN or one or more of it's maintainers.

Before I let rip, in writing this I am not attempting to gloss over or otherwise dismiss the real complaints from those that feel that they have been harassed by responses from a CRAN maintainer. It's not my place to address those issues, but rather something that the R Foundation should be handling. If true, and I have no reason to doubt the claims, there is no place for such treatment of individuals, no matter their transgression. Did you hear me? Ok, with that said, here goes.

For all the good that there is in the R community, one part of the rot that exists is with package authors. Not all package authors, mind. Just a few package authors. Some of those same people seem very vocal on Twitter and elsewhere about the perceived problems and question why CRAN has the temerity to uphold them to some quality standards. The rot, or at least a not-insignificant part of it, is those package authors that don't give a crap about the quality of their submissions or those that don't think the rules apply to them.

There is nothing mystical or random about getting a package on to CRAN. You create your package following the guidelines & advice in [Writing R Extensions](http://cran.r-project.org/doc/manuals/r-patched/R-exts.html) (WRE), which, whilst verbose in places it doesn't need to be, at least includes most of the relevant information if people would just both to read it. I hear complaints about it being some hundred and odd pages and that people don't have time to read it. Wait, you don't have time to read the documentation that is provided but then get all bent out of shape when a *volunteer* CRAN Maintainer calls you on your lack of effort?

Big chunks of WRE don't apply to most packages; not including C, C++, or FORTRAN code in your package? Great, ignore the 60% of the manual that doesn't apply to you. By my reckoning there are on the order of 70 sparse pages that cover all you need to know about writing an R package, conveniently listed in the first 2 chapters of WRE. Add 2 more pages if you want to write new generic functions and methods. How many of those complaining read the 100-odd pages of Hadley Wickham's [R Packages book](http://shop.oreilly.com/product/0636920034421.do) (or the equivalent [web/HTML version](http://r-pkgs.had.co.nz/))?

That information, those 70 pages, is what most package authors need. Yes, OK, some people will be proficient programmers writing interfaces to compiled code who'll need to read the other 60%, but I sure as hell hope they do read it because I'd really appreciate it if their compiled code didn't segfault my R session just because I had the nerve to use their package.

If you've gotten your code this far, you should have a reasonably functioning package. Next step is to do what WRE tells you and run `R CMD check --as-cran` and `R CMD INSTALL` on the **tarball** (i.e. on the thing produced by `R CMD build`, **not** your source tree). If there are **any** issues here, fix them or make a note to tell CRAN about the issue and why it is either a false positive or nothing to worry about. This is important! A lot of what CRAN does is manual; help them by telling them why they shouldn't worry that your package generated 3 NOTEs. You probably want to check this on at least two OSes (Linux and Windows would be ideal) and under the current R release and a recent build of R-devel. The latter may be a bit of a pain but you only really need to do this when you are doing pre-flight checks for CRAN, not at all stages of development. Using the [Win-builder service](http://win-builder.r-project.org/) run by Uwe Ligges will cover Windows and R-devel on that OS to boot. Using a continuous integration service like Travis CI or Appveyor can help with testing on Linux/OS X and Windows respectively. Using these fancy new tools isn't *that* technical, difficult, or insurmountable; if you are building a package in the first place you already have access to one test system and Win-builder gives you another, for free and you get the R-devel ribbon on top! 

Having done all of that, you need to read the CRAN policy for submissions. And re-read it. And read it *each* and *every* time you submit a package to CRAN, not just on the first occasion. It changes from time to time to reflect tightening of the policy or to accommodate changes in R and the checking system.

That done, you should be good to go. But, yes sometimes you'll have overlooked something, or your test systems weren't configured in the same way as CRAN's were. Or something else. If you've read the policy and followed WRE, then this is the one place where some error might creep in. But you know what, if CRAN tells you to single-quote some words, or title case your `Title` tag, or put a period on the end of the `Description` tag, or something else, just fix the damn problem and get on with life. You might think this is petty, and from some points of view it probably is petty, but CRAN don't and if you want your package on CRAN then you have to follow their rules! It really is that simple. You don't like it? You're welcome to a refund and can always set up a competing repository yourself.

Some package authors complain that CRAN is sweating the insignificant details at the expense of letting through compliant-but-pointless or crap-or-broken packages. Oliver Keyes recently commented on Twitter

<blockquote class="twitter-tweet" lang="en" align="center"><p lang="en" dir="ltr">Let me say a big thank you to BDR for shouting at me for not using single quotes but not noticing fundamentally broken vignettes</p>&mdash; Oliver Keyes (@quominus) <a href="https://twitter.com/quominus/status/605025216222265344">May 31, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

complaining about being asked to single-quote something or other whilst broken vignettes seemingly languish on CRAN (it's not immediately clear whether Oliver was referring to his own vignettes or something from another package). The implication here seems to be that BDR is somehow being remiss in pointing out one transgression of the rules whilst simultaneously allowing other, more serious, transgressions. This is invariably a false argument of course. If BDR #sweat[s]theshitthatdoesntmatter (as @recology_ succinctly put it), he sure as hell isn't letting an obviously --- visibly --- broken vignette through the pearly gates now is he? Of course not!

If there are broken vignettes/packages on CRAN there are two reasons

 1. the author of the package doesn't care about fixing the vignette/package but it is broken in a non-obvious-to-CRAN way, or
 2. the package author's vignette/package has broken due to recent changes in dependencies or R, (OK I guess there's a 3rd option...
 3. the package author doesn't know and, you know what, perhaps a friendly note to tell them would suffice )

If the reality is 2. and the package is still on CRAN, then be thankful that CRAN is probably allowing a period of grace for the package author to fix the problem. Can you imagine the cacophony of wailing from the twitteRati if CRAN pulled their packages as soon as `R CMD check` threw an error? You might mistake such an event for the rapture...

If the problem is 1. then what do you want BDR or CRAN to do about it? They get lambasted for too much reliance on manual checks and then when their automated checks fail to catch a problem they're damned again! If the problem is 1. then the ire should be directed at the respective package author, not CRAN. The problem of broken vignettes etc is not something CRAN can do much about; that contribution to the rottenness lies squarely at the feet of R package authors.

I don't know for sure, but I can see reasons for CRAN wanting to improve the way packages are presented and described on CRAN's website. That they sweat these seemingly trivial details because if package authors get those things right, they're probably conscientious enough to make sure there aren't other, undetectable-to-CRAN problems with their package.  We should be lauding this attention to detail if the effort in quoting a few words and changing the case of some title or other is what stops idiots from throwing-up whatever it was they ate for lunch into a tarball destined for CRAN.

If you follow the [CRANberries package feed](http://dirk.eddelbuettel.com/cranberries/) you'll be amazed at the number of packages that get yanked from CRAN; invariably because some problem *was* found later in their package or changes to R broke the package and the author failed to sort the problem. This all has to be handled responsibly by CRAN because they invariably have a legal obligation to continue to make the sources for those removed packages available for download. This is not a trivial exercise to dump this garbage in a responsible manner, with a human-negotiated time interval within which a problem will be fixed (note the cacophony point above). Raising the barrier to entry for R packages shipped via CRAN is, in my not so humble opinion, a good thing if it weeds out those that can't be arsed with the effort involved in jumping through the ever-shifting hoops of WRE and CRAN's policy.

So, yes there is a problem in the R community. It's just not the entity that you all thought was the problem, at least not entirely. If the rot has set in, if the sickness has infected the community, package authors are very much partly to blame. There is no secret sauce to getting a package on to CRAN, despite what some people might think or claim. The only cure to the sickness is to sweat the detail, read the documentation, do what CRAN says. If you don't like that, then go play somewhere else. There are hundreds, if not thousands, of package authors that have successfully navigated the treacherous waters that lie before CRAN's safe harbour. You know what each of these package authors has in common? They (eventually) read the documentation and played by the rules. What makes you so special that you should get a free pass on that?

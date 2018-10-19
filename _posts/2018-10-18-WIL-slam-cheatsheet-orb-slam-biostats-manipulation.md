---
layout: post
title: SLAM Cheatsheet; Is ORB SLAM Misnamed?; Biostats Study Manipulation
description: Alex thompson is a Stallion!!!
image: https://s3-us-west-1.amazonaws.com/co-directory-images/alexcthompson.jpg
mathjax: true
---

Three topics, here's the TL;DR!

1. I created a [cheatsheet for SLAM terms, resources, and important papers]({{ site.baseurl}}{% link slam-cheatsheet.md %}).  (SLAM stands for Simultaneous Localization and Mapping, see the glossary!)  It's a work in progress.
2. Working through ORB SLAM, I realize it is misnamed.  While ORB features are important part of the technique, they could be swapped out for another feature-detector-descriptor.  The greater innovation seems to be in the thoughtful weaving of many algorithms through four threaded processes making map updates.
3. [Alarming survey results from consulting biostatisticians were released (Annals of Internal Medicine)](http://annals.org/aim/article-abstract/2706170/researcher-requests-inappropriate-analysis-reporting-u-s-survey-consulting-biostatisticians) showing that researchers frequently make inappropriate requests of biostatisticians to alter results.  One stat: 24% of respondents had been asked to **"remove or alter some data records to better support the research hypothesis"**.  Mark Twain comes to mind ...
   ![Lies, damn lies, and statistics](/images/lies_statistics.png){:class="img-responsive"}

I go in depth on each below the fold ...

<!--excerpt-->

# SLAM Cheatsheet

I mentioned last week that I thought there was a need for a glossary of terms for SLAM learners.  I complained, but I also have the capacity to help fix it, so I started a [SLAM cheatsheet]({{ site.baseurl}}{% link slam-cheatsheet.md %}).  For now I've defined about 30 terms, and another roughly 30 terms TBD (To Be Defined!), and more as I go.  I also started a short list of resources and papers I have found valuable.  It's a work in progress, so please comment or email me with any suggestions or revisions.

Also, a hat-tip is deserved for Github user `kanster`, aka robotics researcher Kanzhi Wu, who has a [similar **Awesome SLAM** cheatsheet](https://github.com/kanster/awesome-slam) up that I have found useful.

# Is ORB SLAM misnamed?

As I wrote last week, I've been working on understanding ORB SLAM2 from soup to nuts.  Since I don't have a deep robotics background, this has meant numerous stoppages to learn foundational knowledge, or dig up references I don't get.  It's slow going but it's fun to note a gradual acceleration as more and more often, the terms I encounter in this and other papers are terms I've freshly learned.

Now that I'm deep into ORB SLAM, I think it is misnamed.  The ORB feature-detector-descriptor is essential to establishing keypoints in images and matching them across images to triangulate map points.  But there are numerous other featured-detector-descriptors that could potentially take its place.  For example, the paper [A Comparative Analysis of SIFT, SURF, KAZE, AKAZE, ORB, and BRISK (2018)](https://ieeexplore.ieee.org/document/8346440), by Shaharyar Ahmed Khan Tareen and Zahra Saleem, compares nine of these detectors on four factors: (1) quantity of detected features, (2) computational efficiency per feature point, (3) efficiency of matching per feature point, and (4) speed of total image matching.

The ORB SLAM authors might have been able to use another feature-detector-descriptor, and we'd have AKAZE SLAM instead.  And I can see that happening ultimately: as compute powers up, or people optimize chips for specific detectors, it may be the case that the ORB SLAM architecture continues to be relevant, but more compute intensive feature-detector-descriptors replace ORB to improve performance.

What is truly impressive, to me, about this algorithm is the way that they have fit many different algorithmic approaches to SLAM and computer vision together, each piece tailor fit to its role, creating a cohesive system.  It's not simple, it's very layered, and it takes time to understand, but it's a great example of good engineering: a fusion of strong technique with pragmatism.

On the other hand, if you don't name it ORB SLAM, what are you going to name it instead: *"multifaceted integrated system for SLAM that runs real fast"*?  It doesn't roll off the tongue.

By the way, here's Tareen and Saleem's summary of feature-detector-descriptor performance:

> The quantitative comparison has shown that the generic order of feature-detector-descriptors for their ability to detect high quantity of features is:
> 
> $$\begin{equation*} \mathbf{ORB} > \mathbf{BRISK} > \mathbf{SURF} > \mathbf{SIFT} > \mathbf{AKAZE} > \mathbf{KAZE} \end{equation*}$$
> 
> The sequence of algorithms for computational efficiency of feature-detection-description per feature-point is:
> 
> $$\begin{align*} \mathbf{ORB}& > \mathbf{ORB}(\pmb{1000}) > \mathbf{BRISK} > \mathbf{BRISK}(\pmb{1000}) > \mathbf{SURF}(\pmb{64}\mathbf{D})\\ & > \mathbf{SURF}(\pmb{128}\mathbf{D}) > \mathbf{AKAZE} > \mathbf{SIFT} > \mathbf{KAZE} \end{align*}$$
> 
> The order of efficient feature-matching per feature-point is:
> 
> $$\begin{align*} \mathbf{ORB}(\pmb{1000})& > \mathbf{BRISK}(\pmb{1000}) > \mathbf{AKAZE} > \mathbf{KAZE} > \mathbf{SURF}(\pmb{64}\mathbf{D})\\ & > \mathbf{ORB} > \mathbf{BRISK} > \mathbf{SIFT} > \mathbf{SURF}(\pmb{128}\mathbf{D}) \end{align*}$$
> 
> The feature-detector-descriptors can be rated for the speed of total image matching as:
> 
> $$\begin{align*} \mathbf{ORB}(\pmb{1000})& > \mathbf{BRISK}(\pmb{1000}) > \mathbf{AKAZE} > \mathbf{KAZE} > \mathbf{SURF}(\pmb{64}\mathbf{D})\\ & > \mathbf{SIFT} > \mathbf{ORB} > \mathbf{BRISK} > \mathbf{SURF}(\pmb{128}\mathbf{D}) \end{align*}$$

# Lies, damn lies, and biostatistics

I was actually surprised by how alarming the results were from [Researcher Requests for Inappropriate Analysis and Reporting: A U.S. Survey of Consulting Biostatisticians (2018)](http://annals.org/aim/article-abstract/2706170/researcher-requests-inappropriate-analysis-reporting-u-s-survey-consulting-biostatisticians).  There's no PDF publicly available but [there might be one on Reddit](https://www.reddit.com/r/sciences/comments/9myvyp/biostatisticians_report_that_they_receive_an/) if you look hard enough.

390 consulting biostatisticians rated the frequency with which they received 18 different types of inappropriate requests in the last 5 years.  Researchers also rated how severe they considered the request.  Here's the results, sorted by severity, with the most severe at the top.  The right three columns show the % of respondents receiving these kinds of request 0 times, 1-9 times, or 10+ times:

![Researcher Requests for Inappropriate Analysis (Table 1)](/images/Researcher Requests for Inappropriate Analysis (Table 1).png)

As you would expect, the two most severe requests were seen the least often.  However, it's still pretty bad, as 7% of respondents received the request to *"Change data to achive the desired outcome"* 1-9 times in the last 5 years.  That's just a tremendously unethical; literally an attempt to falsify results.

30% of respondents reported being asked to *"interpret the statistical findings on the basis of expectations, not the actual results"*.  This has been asked of me many times in my career ... in business settings by people in non-scientific outward facing functions who are concerned about the image of the organization.  I don't like being asked that, and I resent that it's probably better for one's career to go along with those requests on occasion.  That said, I expect those requests in those circumstances because there's not a set of professional ethics governing the situation that all are agreed on.  But what person in a scientific setting can ask that and maintain a positive reputation?

For some of the lesser offenses, which in a way are still really bad, the rates are way higher.  Researchers requested *"Stress only the significant findings, but underreport nonsignificant ones"* 1-9 times from 48% of respondents, and 10+ times from 7% of respondents.

Some of these are more due to sloppy execution - *"Do not fully describe the treatment under study because protocol was not exactly followed"* for example - but I cannot find any violation on here that I can attribute to simple statistical ignorance.  Regardless, given the [Replication Crisis](https://en.wikipedia.org/wiki/Replication_crisis) occuring throughout science right now, we have to take these survey results very seriously.

This caught my eye in the study discussion:

> Before the pilot study of our survey (28), the only published report we could identify that quantified researcher requests for inappropriate analysis and reporting was a 1998 international survey of biostatisticians who were members of the International Society for Clinical Biostatistics (29). Although the response rate was only 37%, the authors felt that “ . . . the high proportion of respondents knowing about fraudulent projects [51%] provided the primary motivation for [publishing their] report” (29).

This needs to be an annual survey.

In the [Reddit discussion of the survey](https://www.reddit.com/r/sciences/comments/9myvyp/biostatisticians_report_that_they_receive_an/) there's interesting commentary there from (self-identified) scientists.  For example:

> I've seen both sides of the issue they list as a limitation (not knowing whether ignorance or malfeasance is the causal factor in bad statistical analysis)- rotated in a lab where the following conversation happened:
> 
> Post-doc: the effect isn't significant, but it's close.
> 
> PI: okay, running a few more animals should get it over the top.
> 
> Post-doc: sounds good.
> 
> I myself have made a number of mistakes in my analyses as well. Fortunately these we're caught before publication, but it doesn't feel great.

and:

> Some times this is just honest misunderstanding of statistics, which is, unfortunately, extremely common. Sometimes this is just people who are zealots about their work and are convinced that someone else should be able to show something that they're convinced is there - but isn't coming up in the data for them.

Both anecdotes are concerning if true, and consistent with the survey results.

However, this attitude:

> Can’t trust “science” readily anymore, unfortunately.

is textbook nihilism. We should ignore it.

Editorializing, I have never met a scientist (or, more broadly, scholar) who enjoys peer review; IMO, most resent it.  Scientists are ego driven too, and they want to prove their hypotheses true.  In that sense, I believe **there are no scientists**.  Scientists don't exist *except* in the context of a scientific community, where everyone grudgingly submits to peer review and does their best, despite it being unnatural, to follow scientific method.  This survey confirms my point of view, big time. \#confirmationbias.  I'm glad there's some sharp eyed biostatisticians out there spotting and stopping these kinds of practices.

Which makes a relevant point for data science teams: the moment you have two or more data scientists at your company, start a process for peer reviewing all work before deployment or publication.

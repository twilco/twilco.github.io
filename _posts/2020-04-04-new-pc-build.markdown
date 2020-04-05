---
layout: post
title:  "New PC build: Temperature and compilation benchmarks"
date:   2020-04-04 12:42:53
categories: posts
description: An overview of my new 3900x PC build.  Compilation benchmarks of WebKit are performed for my new system vs. my old system.  System temperatures under RealBench and Prime95 load are tested with and without a 120mm Noctua NF-F12 rear exhaust fan.
---

{: .no_toc}
#### Table of contents
1. TOC
{:toc}

### Introduction

I have spent a lot of time lately working on large C++ and Rust projects — for example, a [failed attempt](https://github.com/servo/servo/pull/24462) at correctly handling pending request state for HTML `<img>.complete` in Servo, and [an attempt](https://bugs.webkit.org/show_bug.cgi?id=195180) at implementing [`lh` and `rlh` units](https://www.w3.org/TR/css-values-4/#font-relative-lengths) in WebKit.  Unsurprisingly, even the smallest changes to either of these projects takes a lot of compute to recompile, making development cycles slower than ideal.  Any change to Servo was an ~8 minute recompile (the `script` module seems to be one of the more compute-intensive parts), and any change to WebKit was a 3-10 minute recompile depending on what I had touched.

Compilation is a function of computation over time.  Since I can't yet manipulate time, I'll have to increase my compute — hence, a new PC build.  Here's what you can expect from the remainder of this post:

1. An overview of the new specs vs. the old
2. Temperature benchmarks before and after the introduction of a rear exhaust fan
3. A comparison of WebKit compilation times in the new vs. the old

### The build

In my old build, I was running an i5-6600k (4 cores, 4 hardware threads), 16GB of 2400mhz CL15 RAM (12.5ns memory access latency), and an Nvidia GTX 970.  You can check out my new build [here](https://pcpartpicker.com/user/twilco/saved/#view=6jsVcf).  If you don't want to click, the important points as far as compute goes are the introduction of a 3900x (12 cores, 24 hardware threads) and 32gb of 3600mhz CL16 RAM (8.88ns memory access latency).  The 3900x is [much faster](https://cpu.userbenchmark.com/Compare/Intel-Core-i5-6600K-vs-AMD-Ryzen-9-3900X/3503vs4044) than the i5-6600k.

I don't play graphically intensive games, so I transplanted the GTX 970 from the old build to the new rather than upgrading.

### What difference does one case fan make?

I am running 5 case fans in this build — two X2 GP-12 120mm static voltage fans came with the Meshify C and are run as top exhausts, two 140mm Noctua PWM NF-A14s are run as front intake fans, and one 120mm PWM Noctua NF-F12 as a rear exhaust.  This many fans is likely overkill...but at least I can be reasonably confident things will stay cool and quiet.

All of the parts in this build arrived within a week of when I ordered them, minus the 120mm rear exhaust NF-F12 which took roughly three weeks.  I decided to initially build without it.  It has finally arrived, so let's do some temperature benchmarks before and after the introduction of this fan.

For these tests I'll be using [RealBench](https://rog.asus.com/tag/realbench/), since that stresses both the GPU and CPU in a "realistic" fashion, and [Prime95](https://www.mersenne.org/download/), which focuses on CPU stress-testing.  All of these tests were run with every PWM fan set to the "Silent" fan profile, as that is closer to what I would actually use (though I will be setting up custom fan profiles in the end).  These tests were run within hours of each other, so ambient temperature outside the case should be roughly the same between each test.

{:refdef: style="text-align: center;"}
<a href="/assets/img/new-pc-build/RealBenchInAction.png">![Image showing RealBench running.](/assets/img/new-pc-build/RealBenchInAction.png)</a>
{:refdef}
<div style="margin-top: -10px; margin-bottom: 10px; text-align: center; font-style: italic; font-size: .85rem">RealBench in action.  Note I ended up running these tests for a total of an hour.</div>

{:refdef: style="text-align: center;"}
<a href="/assets/img/new-pc-build/Prime95.png">![Image showing Prime95 about to run.](/assets/img/new-pc-build/Prime95.png)</a>
{:refdef}
<div style="margin-top: -10px; margin-bottom: 10px; text-align: center; font-style: italic; font-size: .85rem">Prime95.</div>

{: .no_toc}
#### Results

<div style="margin-bottom: 15px;">
<b>Without rear exhaust 120mm</b><br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench CPU peak: 70°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench CPU ambient: 68°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench GPU peak: 67°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench GPU ambient: 65°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; Prime95 CPU peak: 79°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; Prime95 CPU ambient: 66°C<br/>

<br/><b>With rear exhaust 120mm</b><br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench CPU peak: 69°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench CPU ambient:  67-68°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench GPU peak:  67°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; RealBench GPU ambient:  65°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; Prime95 CPU peak:  76°C<br/>
&nbsp;&nbsp;&nbsp;&nbsp; Prime95 CPU ambient:  65-66°C<br/>
</div>

Given a 1-2°C margin of error, the results are roughly the same.  In case you needed more confirmation that there are diminishing returns in adding more case fans, let me prove it again.  I don't think it's a complete waste, as more fans should be able to cool a non-peak load system more quietly by all spinning at an inaudible base-level RPM.

### WebKit compilation comparison

For the following tests, I ran a clean build of WebKit with an empty `ccache`:

{% highlight bash %}
ccache --clear && \
Tools/Scripts/build-webkit --gtk --debug --clean && \
Tools/Scripts/build-webkit --gtk --debug
{% endhighlight %}

I'm sure there are lower-level caches that aren't cleared running these back-to-back, but I'm okay with this not being an exact science.

<div style="margin-bottom: 15px">
<b>Old build (i5-6600k, 16gb 2400mhz CL15 RAM):</b><br/>

&nbsp;&nbsp;&nbsp;&nbsp;Attempt 1: 56min 0sec<br/>
&nbsp;&nbsp;&nbsp;&nbsp;Attempt 2: 53min 50sec<br/>
&nbsp;&nbsp;&nbsp;&nbsp;Attempt 3: 53min 42sec<br/>

<br/><b>New build (3900x, 32gb 3600mhz CL16 RAM):</b><br/>

&nbsp;&nbsp;&nbsp;&nbsp;Attempt 1: 14min 59sec<br/>
&nbsp;&nbsp;&nbsp;&nbsp;Attempt 2: 14min 22sec<br/>
&nbsp;&nbsp;&nbsp;&nbsp;Attempt 3: 14min 23sec<br/>
</div>

About 374% faster...I'll take it!

WebKit recently [gained the ability to effortlessly integrate with IceCC](https://lists.webkit.org/pipermail/webkit-dev/2020-March/031147.html), a distributed compilation tool.  Renting some cloud compute and taking IceCC out for a test run sounds like a fun experiment.  Stay tuned!

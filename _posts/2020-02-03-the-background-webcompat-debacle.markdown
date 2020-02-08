---
layout: post
title:  "The `background` debacle — a case study on web compatibility"
date:   2020-02-03 12:42:53
categories: posts
description: A case study on web compatibility featuring the serialization of the `background` CSS shorthand property.  Learn about the CSS standards process and web platform tests (WPTs) along the way.
published: false
---

{: .no_toc}
#### Table of contents
1. TOC
{:toc}

### Introduction

A few days ago, I was browsing the [CSS Backgrounds 3 spec](https://www.w3.org/TR/2017/CR-css-backgrounds-3-20171017) trying to determine the browser-default `background-color` of the viewport for use in [Kosmonaut](https://github.com/twilco/kosmonaut) (hint: it's system and browser dependent).

In doing so, I stumbled across this note in the changelog:

{:refdef: style="text-align: center;"}
<a href="/assets/img/background_serialization/bg_serialization_change.png">![Image saying: (Header) Changes since the 9 September 2014 Candidate Recommendation (Bullet list item) Moved background-color (hyperlink) component of final-bg-layer (hyperlink) to the front for serialization because authors seem to expect this even though it doesn't make sense?](/assets/img/background_serialization/bg_serialization_change.png)</a>
{:refdef}

Clicking into the <code>&lt;final-bg-layer&gt;</code> hyperlink, we discover this revision changed the serialization order of the `background` shorthand property:

{:refdef: style="text-align: center;"}
<a href="/assets/img/background_serialization/background_shorthand_definition.png">![Image showing the spec definition of the background shorthand property.  Highlighted in red is the value for the property: bg-layer#, final-bg-layer.  Further down in the image, the definition of final-bg-layer is also highlighted in red: background-color || bg-image || bg-position](/assets/img/background_serialization/background_shorthand_definition.png)</a>
{:refdef}
<div style="margin-top: -20px; margin-bottom: 10px; text-align: center; font-style: italic; font-size: .85rem">The <span style="font-style: normal"><code>#</code></span> next to <span style="font-style: italic;"><code>&lt;bg-layer&gt;</code></span> means "0 or more" non-final background layers</div>

This intrigued me.  Why do authors expect the color to be first, and why does it make less sense for the color to be first?  What does this change mean in practicality?  How was this decision to change the specification made?

Read on for a glimpse into [how the CSS sausage is made](https://en.wiktionary.org/wiki/how_the_sausage_gets_made), from CSS specification to (sometimes differing) browser implementation.  You can expect to learn about the CSS standards process, CSS serialization, web platform tests, and more, hopefully gaining some appreciation for the difficult task that web compatibility is.

### First, a detour: how CSS comes to be

Having an understanding of how the CSS standards process works will help in understanding the `background` shorthand awkwardness, so let's go over it.

In summary: CSS syntax is standardized by the [CSSWG (CSS working group)](https://www.w3.org/Style/CSS/members), which is comprised of representatives from browser vendors, universities, various companies, and some independent experts.  New CSS syntax passes through six stages:

<ol class="bold slightly-spaced-list">
    <li>
      Editor's draft (ED) 
      <br />
      <span class="non-bold">
        New syntax goes from idea to "paper" in this phase, during which it is fleshed out internally by the CSSWG.  You can find WIP editor drafts here: <a href="https://drafts.csswg.org/">https://drafts.csswg.org/</a>
      </span>
    </li>
    <li>
      Working draft (WD) 
      <br />
      <span class="non-bold">
        If an ED is accepted internally by the CSSWG, it moves onto this phase for review by the community (technical organizations, W3C members, the public).  Changes to the specification continue to be made during this phase.  You can find the current working drafts (and more) here: <a href="https://www.w3.org/Style/CSS/current-work.en.html">https://www.w3.org/Style/CSS/current-work.en.html</a>
      </span>
    </li>
    <li>
      Last call working draft (LCWD) 
      <br />
      <span class="non-bold">
        The LCWD provides a hard deadline for any final changes to a WD before it moves on to the CR phase.
      </span>
    </li>
    <li>
      Candidate recommendation (CR)
      <br />
      <span class="non-bold">
        Browsers typically implement the specification in this phase in order to gain <a href="https://www.w3.org/2018/Process-20180201/#implementation-experience">implementation experience</a>, thus determining the worthiness of this specification for the final phases.  We see this phase mentioned in the above screenshot, meaning the change to <code>background</code> serialization we will dive into shortly was an revision to the original CR. 
      </span>
    </li>
    <li>
      Proposed recommendation (PR)
      <br />
      <span class="non-bold">
        The <a href="https://www.w3.org/2005/10/Process-20051014/organization#AC">W3C Advisory Committee</a> decides if the specification should move to the final phase.
      </span>
    </li>
    <li>
      Recommendation (REC)
      <br />
      <span class="non-bold">
        The specification is considered complete and ready to implement.  Only small maintenance work happens at this phase.  In reality, most browsers implement specifications at the CR phase, so specifications that make here are often "dead", laden with errors discovered in implementation that are too difficult to fix in these later phases.
      </span>
    </li>
</ol>

The above is largely a summary of [this article](https://css-tricks.com/css-standards-process/), so head there if you're looking for more detail.

### Authors?  Serialization?

Before we continue, allow me to clarify some terms you might not be familiar with.

> Moved &lt;background-color&gt; component of &lt;final-bg-layer&gt; to the front for serialization because some authors seem to expect this even though it makes less sense? 

In the context of CSS specifications, "authors" are the authors of webpages — web developers.  You may also see the terminology "implementor" — these are the people implementing the specification, so browser (which is also known as a _user agent_) developers.

Serialization is the process of converting from a data structure (e.g. Rust struct, C++ class) representation of a CSS object to a string representation of that CSS object.  To see this more concretely, check out [this portion of the CSSOM (CSS Object Model) spec](https://drafts.csswg.org/cssom/#serializing-css-values) that details step-by-step how to serialize various CSS values and components.  Here's an excerpt showing serialization of a ratio value:

{:refdef: style="text-align: center;"}
<a href="/assets/img/background_serialization/serialize_ratio_component.png">![Serializing ratio: The numerator serialized as per number, followed by the literal string " / ", followed by the denominator serialized as per number](/assets/img/background_serialization/serialize_ratio_component.png)</a>
{:refdef}

Whenever you ask the browser for state on styles, such as through the [HTMLElement.style](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style) or [Window.getComputedStyle](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle) APIs, the value it returns to you is the result of the serialization of its data structures to a string.

It's important to note that how you input styles for an element may not be how they are later serialized back to you.  In fact, this point is the crux of the issue we're exploring — the serialization of the `background` shorthand property differed from author's expectations based on previous browser behavior and the spec.

For example, given this specified style:

{% highlight css %}
#target {
  background: url("https://bit.ly/") 1px 2px / 3px 4px
                space round local padding-box content-box,
              rgb(5, 6, 7) url("https://bit.ly/") 1px 2px / 3px 4px
                space round local padding-box content-box;
}
{% endhighlight%}

You might get back this (or something completely different, depending on your browser) post-serialization:

{% highlight javascript %}
const target = document.getElementById('target')
console.log(target.style['background'])

// Logs the following.
// Note the difference in order vs. what was specified above.

// url("https://bit.ly/") space round local
//  1px 2px / 3px 4px padding-box content-box,
// rgb(5, 6, 7) url("https://bit.ly/") space round local
//  1px 2px / 3px 4px padding-box content-box
{% endhighlight %}

The order in which properties are serialized could be very important.  For example, perhaps you are writing a JavaScript framework that parses the serialized styles of HTML elements in order to compute some new ones.  Different serializations in different browsers makes this much, much more difficult to do.

### So why was this change made?  

Development of these specifications [happens on Github](https://github.com/w3c/csswg-drafts), so let's look at [the commit that introduced this change](https://github.com/w3c/csswg-drafts/commit/02fe11230e02279b495e4c5931be6ed5bab61c5c):

{:refdef: style="text-align: center;"}
<a href="/assets/img/background_serialization/background_serialization_commit.png">![Image of commit on GitHub making the revision to the background serialization spec.  Commit message is: Move bg-color to beginning of final layer per (hyperlink to meeting notes) even though it is silly (as leaverou mentions in the minutes)](/assets/img/background_serialization/background_serialization_commit.png)</a>
{:refdef}

Clicking on the [pictured hyperlink](https://lists.w3.org/Archives/Public/www-style/2015Jan/0406.html) takes us to a set of CSSWG telecon meeting notes from 2015.  Search for "background serialization" to find the bit we're interested in.

Feel free to read the full conversation yourself — for the sake of brevity, here is a summary of points:

<ol class="bold slightly-spaced-list">
    <li>
      <span class="non-bold">
      The <a href="https://www.w3.org/TR/CSS21/colors.html#background-properties">CSS 2.1 background spec</a> placed the <code>&lt;background-color&gt;</code> first in the <code>&lt;final-background-layer&gt;</code> value list.
      </span>
    </li>
    <li>
      <span class="non-bold">
      For CSS3, the <code>&lt;background-color&gt;</code> was moved to the end of the <code>&lt;final-background-layer&gt;</code> because it is painted underneath all the other components of this layer (e.g. the background image and all its modifiers).
      </span>
    </li>
    <li>
      <span class="non-bold">
        <a href="https://bugzilla.mozilla.org/show_bug.cgi?id=743392">Authors complained</a> about this change in serialization, as tutorials had codified the CSS 2.1 serialization order (<code>&lt;background-color&gt;</code> first).
      </span>
    </li>
    <li>
      <span class="non-bold">
        It is discovered Chrome's <code>background</code> shorthand serialization is "100% broken".  While not made as explicit in the conversation, WebKit (Safari's browser engine) and Gecko (Firefox's browser engine) likely had, and may still have problems relative to the spec — more on that later.
      </span>
    </li>
</ol>

### But wait, there's more!

The 2016 spec revision was not the only confusion surrounding `background` serialization — the topic [was also raised as an issue in the CSSWG repository](https://github.com/w3c/csswg-drafts/issues/418):

{:refdef: style="text-align: center;"}
<a href="/assets/img/background_serialization/cssom_define_ser.png">![GitHub issue with the title: cssom define serialization for background shorthand](/assets/img/background_serialization/cssom_define_ser.png)</a>
{:refdef}

{:refdef: style="text-align: center;"}
<a href="/assets/img/background_serialization/serialization_questions.png">![Part of the text of the GitHub issue described above.  Text is: Firefox seems to only serialize when all set longhands are of the same length(and when it does, it serializes to all default values).  Chrome always serializes, and just prints out whatever was set.  Paragraph break.  This should probably be specced.  Important questions to answer.  Bullet list item: Should we truncate everything to the length of the background-image longhand?  Bullet list item: For underspecified longhands, should we cycle through them to make them the same length as background-image?  Bullet list item: Should we be explicit (like Firefox) or implicit (like Chrome)?  Bullet list item: Is serialization (of a shorthand) allowed to fail if parsing hasn't?](/assets/img/background_serialization/serialization_questions.png)</a>
{:refdef}

And roughly two years after this issue was opened, [it was noted in a comment](https://github.com/w3c/csswg-drafts/issues/418#issuecomment-380951618) that this inconsistency across browsers still existed.  In fact, the linked comment shows all browsers serialized `background` wrong relative to the spec, each in different ways.

### Web platform tests: the ultimate equalizer

There is a somewhat happy ending to this story, and it comes thanks to something called a _web platform test_ (WPT).  Quoting the [WPT project readme](https://github.com/web-platform-tests/wpt#the-web-platform-tests-project):

> The web-platform-tests project is a cross-browser test suite for the Web-platform stack. Writing tests in a way that allows them to be run in all browsers gives browser projects confidence that they are shipping software that is compatible with other implementations, and that later implementations will be compatible with their implementations.

Most browsers automatically sync these tests to their own repositories, and use them to help prevent any set of changes from accidentally regressing web-compatibility.  At the time of this post, there are a whopping 41,492 WPTs, each of which made up of one or more subtests, for a grand total of _1,711,211 subtests_.  That's a lot!

The `web-platform-tests` group also maintains a website called [wpt.fyi](https://wpt.fyi/results/?label=experimental&label=master&aligned), which provides a dashboard to view the results of any and all WPTs for the included browsers.

As you might now be guessing, [a WPT was created](https://github.com/web-platform-tests/wpt/pull/10462/files) to more rigidly enforce all the various ways the `background` shorthand property may be serialized.  This is still not ideal, as the test considers five different serializations (spec, Edge, Firefox, Blink, and WebKit) valid, but it's a great step forward.  With this test in place, it's at least made clear to authors what serializations they could and should expect if they decide to rely on this API. 

Making use of the aforementioned dashboard, we can view the [results of this WPT](https://wpt.fyi/results/css/css-backgrounds/parsing/background-valid.html?label=master&label=experimental&aligned&q=background-valid) for each browser — as is expected, all pass!

### Summary

In summary, web compatibility is hard.  Thank your local browser developer when you next get a chance :)

If you have any questions, comments, or corrections, feel free to [open up an issue](https://github.com/twilco/twilco.github.io/issues) or leave a comment below via [utterances](https://github.com/utterance/utterances).

---
<div class="paragraph-sized-spacer"></div>

For some extra credit reading, [check out this discussion](https://github.com/w3c/csswg-drafts/issues/1033) concerning the serialization of computed CSS styles returned from [Window.getComputedStyle()](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle).  There's some interesting back-and-forth on the validity of serializing the object to an empty string vs. something "meaningful", and the technical difficulties that would come with that.

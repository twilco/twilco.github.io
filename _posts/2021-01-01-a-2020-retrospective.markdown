---
layout: post
title:  "My 2020 browser retrospective (and 2021 prospective)"
categories: posts
description: "A quick retrospective on the browser-related things I worked on in 2020, and what I have planned for 2021."
---

{: .no_toc}
#### Table of contents
1. TOC
{:toc}

<hr style="margin-bottom: 15px;" />

I spent much of 2020 working on web browsers (mainly [Kosmonaut](https://github.com/twilco/kosmonaut) and WebKit).  Here's a little write-up about how that was, and what I have planned for
the future.

### Kosmonaut in 2020

One year ago today, I was finishing the first implementation of [block layout](https://github.com/twilco/kosmonaut/commit/f3041f0b7e986b7a0be26b34870489cfaeb3e830) and [display-list generation](https://github.com/twilco/kosmonaut/commit/fedf68029e4ad9ea8bf4b96611b8738a347402f2).  In terms of LoC, Kosmonaut hasn't grown much — from 7.5k then to
20k now.  However, there are some pretty neat improvements wrapped up in that 12.5k LoC increase:

* Layout-tree dump[^1] snapshot based testing
* Partial support[^2] for [abstract box layout](https://drafts.csswg.org/css-writing-modes-4/#abstract-layout) with the `writing-mode` and `direction` properties
* Support for arbitrary scale factors, e.g. for HiDPI screens
* OpenGL-based box painting and text rendering (though text rendering is not yet hooked into layout)

Three months ago, I ventured out to rewrite Kosmonaut's layout engine, as I had made a mess of it in my first pass at
implementing abstract box layout.  The result is [this PR](https://github.com/twilco/kosmonaut/pull/14), merged today,
that ended up being a rewrite of...a lot of things.  Some highlights from that PR include:

* Proper representation of many spec-level concepts, such as [text runs](https://drafts.csswg.org/css-display/#text-run), [formatting contexts](https://drafts.csswg.org/css-display/#formatting-context), various types of boxes, and more.
* Much cleaner layout tree dump output.  [Before](https://github.com/twilco/kosmonaut/blob/a6b7138b2c95714af8ce3a446e2f4b40b2d67950/tests/layout/snapshots/lib__layout__tests__rainbow_divs_baseline.snap) vs. [after](https://github.com/twilco/kosmonaut/blob/82cf7ef53e67e0c33ea8812e486fe8d867a82da9/tests/layout/snapshots/lib__layout__tests__rainbow_divs_baseline.snap)
* A _much_ cleaner HiDPI scaling implementation.  The TLDR of this is that scaling used to be done in layout, which required passing a `scale_factor` everywhere and applying it in exactly the right places.  The [new implementation](https://github.com/twilco/kosmonaut/commit/15f76d68617bf3e846b8405d000b1b9e17dafd72) scales the viewport size down before layout and scales everything else up just before painting, making things far more simple and less prone to bugs.

There's a fair amount of cleanup I'd like to do following the landing of this PR, but it's a big step forward and
I'm really happy with how it turned out.

One of the reasons this took so long is because I spent quite a long time trying out various designs and subsequently
throwing them out.  One thing I really wanted to do with this PR was to enforce certain spec-level constraints with
the type system.  To give a concrete example, [take this snippet from the CSS Display spec](https://drafts.csswg.org/css-display/#block-container) regarding block containers:

> A block container either contains only inline-level boxes participating in an inline formatting context, or contains only block-level boxes participating in a block formatting context. 

I had experimented with a design that looked like this:

{% highlight rust %}
pub struct BlockContainer {}
pub struct BlockLevelBlockContainer {
   children: Vec<BlockLevelBox>
}
pub struct InlineLevelBlockContainer {
   children: Vec<InlineLevelBox>
}

impl BlockContainer {
   pub fn new(...) -> BlockContainer {}

   pub fn add_block_level_child(
      blb: BlockLevelBox
   ) -> BlockLevelBlockContainer {}

   pub fn add_inline_level_child(
      ilb: InlineLevelBox
   ) -> InlineLevelBlockContainer {}
}
{% endhighlight %}

The idea is that you could only get a `BlockLevelBlockContainer` or `InlineLevelBlockContainer` by calling one of
these `add_{block, inline}_level_child` methods (i.e. neither of these structs have `new` methods), ensuring at compile-time
that the quoted spec invariant is upheld.

However, this proved to to be awkward for a number of reasons, and I scrapped the idea in favor of a less restrictive API
that technically allows these rules to be broken.  I would like to revisit this someday, as allowing users to encode
rules via the type system (and making it ergonomic to do so) is something Rust is good at.

### WebKit in 2020

Kosmonaut was not the only browser I worked on in 2020 — this year also saw the beginning of my contributions to WebKit.
I landed [13 patches](https://github.com/WebKit/WebKit/search?q=author%3Atwilco&type=commits), and worked on quite a few
others I didn't push across the finish line.  Here are some highlights:

* Various bug fixes and improvements to WebKit's CSS variables implementation.  I have a blog post in the works describing these in more detail, so stay tuned.
* Fixed a bug that caused `::selection` pseudoelement styles to not be applied to elements with direct anonymous parents, such as text nodes. [[commit]](9d06f52c2c04b24605af1bb19eed43ad03a6d9c4)
* Per spec, ignore order when parsing `<inset>` and `<color>` values for the `box-shadow` property.  This makes values that should've been valid for `box-shadow` actually work.  [[commit]](https://github.com/WebKit/WebKit/commit/b1876da3f1ce3bed2a8c779e5716927ce9d85438) 

In the second half of the year I really felt like I was getting into a groove in working on WebKit, so I'm excited to get
back to it.

### My plans for 2021

2021 has arrived!  Here are some things I'd like to complete in Kosmonaut soon:

* Parse and expand the most common CSS shorthands — `margin`, `border`, `padding`, etc.  Kosmonaut currently only understands longhands, which is quickly becoming inconvenient.
* Add the ability to load styles from `<link href="...">` and `<style></style>` tags.  Currently, loading HTML and CSS in Kosmonaut requires passing all files individually via the `—files` flag, which is inconvenient (are you noticing a trend?).
* [Handle `display: none` boxes](https://github.com/twilco/kosmonaut/blob/82cf7ef53e67e0c33ea8812e486fe8d867a82da9/src/layout/flow/block.rs#L358)
* Further improve support for abstract box layout. Concretely, this means making `writing-mode: {sideways-lr, sideways-rl, vertical-rl}` and permutations with `direction: {ltr, rtl}` work.  I don't think this will be all that hard.  Calculation of [block start coordinates](https://github.com/twilco/kosmonaut/blob/82cf7ef53e67e0c33ea8812e486fe8d867a82da9/src/layout/flow/block.rs#L528) and [inline start coordinates](https://github.com/twilco/kosmonaut/blob/82cf7ef53e67e0c33ea8812e486fe8d867a82da9/src/layout/flow/block.rs#L533) will have to change, and maybe a few other things.
* A basic implementation of [inline layout](https://drafts.csswg.org/css-inline-3/), followed by some polish of Kosmonaut's existing OpenGL text rendering.

After this list, I'm not really sure what will come next for Kosmonaut.  I think it would be a fun minigame to start tackling the [Acid2 test](https://en.wikipedia.org/wiki/Acid2) and/or [some web platform tests](https://github.com/web-platform-tests/wpt).

I've taken a break from WebKit recently to push Kosmonaut's layout rewrite across the finish line, but I want to get back to it soon.  I have been pondering about an experiment I'd like to try regarding my work in WebKit, so stay tuned for information on that — I'll be posting about it here.

---

[^1]: <a href="https://github.com/twilco/kosmonaut/blob/82cf7ef53e67e0c33ea8812e486fe8d867a82da9/tests/layout/directional/snapshots/lib__layout__directional__ltr_vertical_lr_block_boxes_top_left_right_mbp_applied_physically.snap">Here's what a layout dump looks like. </a>

[^2]: The specifics of what "partial support" means is documented <a href="https://github.com/twilco/kosmonaut/blob/e50b640e467a630776a3a3c910839176da98f868/README.md#f1">here</a>.


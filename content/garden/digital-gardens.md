+++
title = "digital gardens"
author = ["Nilay Kumar"]
date = 2025-02-17T00:00:00-05:00
lastmod = 2025-02-18T10:37:00-05:00
tags = ["meta", "web"]
draft = false
progress = "in-progress"
+++

I don't remember exactly when it was that I stumbled upon [gwern.net](https://gwern.net/), perhaps
high school or early college. It's tempting to make the comparison to Alice
tumbling down a rabbit hole, but my hazy recollection is that I found the
content less interesting than the shape and form of the website itself. Less of
a rabbit hole and more of a garden, albeit one somewhat mysterious in its
monochromatic darkness. To find a hypertext garden in the age of Facebook's
explosive popularity felt like a breath of fresh air.

Now, many years later, I find myself drawn again to the concept, with an urge to
start tending to a garden of my own. It's hard to say why... an interest in
small-scale digital spaces in a time of big tech monopolization and climate
breakdown, perhaps? Or maybe just the need to separate from the thoughtless
doomscrolling and actually engage critically with the ideas in my head? A less
[alienating]({{< relref "alienation" >}}) way to sustain a digital presence?

This note explores digital gardening in the hopes of developing my own approach.


## what is a digital garden? {#what-is-a-digital-garden}


### garden vs. stream {#garden-vs-dot-stream}

Mike Caulfield [argues](https://hapgood.us/2015/10/17/the-garden-and-the-stream-a-technopastoral/) that today (note that this was written in 2015) the
internet is dominated by the "stream":

> The “conversational web”. A web obsessed with arguing points. A web seen as a
> tool for self-expression rather than a tool for thought. A web where you weld
> information and data into your arguments so that it can never be repurposed
> against you. The web not as a reconfigurable model of understanding but of
> sealed shut presentations.

In the stream, information comes and goes, and is forgotten or tossed aside as
is convenient. The garden, however, accumulates.

The way we access information in the garden is fundamentally different as well.
There need not be a clear ordering on the pages -- unlike in a blog or a feed,
timestamps are less important than the completion state of a note. Was the
thought _just_ planted? Or is it an old one, tended to regularly, and quite
stable? This makes the garden immediately interesting to explore, allowing
for visitors to wander along their own paths, following links at their lesiure.

Given the garden's inherently unordered, non-hierarchical nature, how do we
expect to find a particular thought? A system of tags and categories might be a
first-order solution. Does it even make sense to have search functionality?


### public learning, continuous growth {#public-learning-continuous-growth}

Maggie Appleton [points out](https://maggieappleton.com/garden-history) aspects of digital gardening that I find particularly
interesting: learning in public, and continuous growth.

> Gardens are imperfect by design. They don’t hide their rough edges or claim to
> be a permanent source of truth. Putting anything imperfect and half-written on
> an “official website” may feel strange. [We seem to reserve all our imperfect
> declarations and poorly-worded announcements for platforms that other people own
> and control.] We have all been trained to behave like tiny, performative
> corporations when it comes to presenting ourselves in digital space.

Learning in public helps dispel both the myth of my expertise (in the off-chance
that it existed to begin with) as well as the myth of knowledge being settled.
For instance, while we might present a fact as settled, it may continue to
produce new and interesting connections to other thoughts long afterwards. So
although I'm not necessarily sold on the idea of associating to each thought an
epistemic status (I don't have the confidence for that), I do like the idea of
categorizing thoughts as seeds/buds/evergreen according to how solid their shape
appears to me. Perhaps I'd even prefer an analogy with the weather: foggy (🌫) /
taking shape (🌤) / clear (☼). Understanding changes -- it may clear up for a
moment, only to be hit by unexpected stormy weather.


### playful and personal {#playful-and-personal}


### ownership and freedom {#ownership-and-freedom}

100 Rabbits build software along a few simple [rules](https://100r.co/site/philosophy.html) that I find useful to keep
in mind for my projects, and are especially relevant to choosing the tooling for
a digital garden.


#### offline first {#offline-first}

Gardening should be possible entirely offline.


#### past-proofing {#past-proofing}

Prefer plaintext tools whenever possible. Use Javascript and CSS sparingly. Keep
images small. Consider principles of [low-tech web design](https://solar.lowtechmagazine.com/about/the-solar-website/).


#### freedom {#freedom}

Content and history should be accessible to visitors through version control
software. Care about accessibility (carefully written alt-text, simple
navigation, good contrast...). Oppose oppression in all forms, e.g. free as in
Palestine. Garden towards a better future: another world is possible.


## my gardening tools {#my-gardening-tools}


### backlinks {#backlinks}

Here is a resource on using hugo to export org-roam backlinks
<https://seds.nl/notes/export_org_roam_backlinks_with_gohugo/>
Actually this might be independent of org-roam, so we might be able to manually
add a backlinks section to each note manually, via hugo

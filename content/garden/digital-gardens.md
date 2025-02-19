+++
title = "digital gardens"
author = ["Nilay Kumar"]
date = 2025-02-17T00:00:00-05:00
lastmod = 2025-02-19T01:24:20-05:00
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

A blog feels formal and rigid -- I'd better get things right the first time
around, otherwise risk being wrong on the internet. There's no room to be a
normal human being. As Appleton puts it,

> Putting anything imperfect and half-written on an “official website” may feel
> strange. We have all been trained to behave like tiny, performative corporations
> when it comes to presenting ourselves in digital space.

The hope is that a garden can be explorative and tentative. Maybe even a bit
playful, though I'm not sure I have any of the writing chops necessary to
express that over text.

Appleton points out that the medium of html+css itself is a potential avenue of
exploration and experimentation. A bit of interior (exterior?) design, done
digital. I do think it makes sense to keep things as simple and straightforward
-- static site, javascript to a minimum, accessible and performant -- as
possible, but maybe it's worth adding: no simpler. Even a zen garden in its
simplicity has its distinct style. I think this will take some time to learn and
discover.


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

High level overview: org-mode + ox-hugo + hugo.


### marking external links with css {#marking-external-links-with-css}

I've marked external links with a slightly subtle dotted underline, with
internal links kept solid. I find this a little less visually distracting that
:after arrow symbols. The css selector is above my paygrade and lifted from
[stackoverflow](https://stackoverflow.com/questions/5379752/css-style-external-links):

```css
div#content.garden a[href]:not(:where(
  /* exclude hash only links */
  [href^="#"],
  /* exclude javascript only links */
  [href^="javascript:" i],
  /* exclude relative but not double slash only links */
  [href^="/"]:not([href^="//"]),
  /* domains to exclude */
  [href*="//nilaykumar.github.com"],
  [href*="//localhost"],
  /* subdomains to exclude */
  [href*="//nilaykumar.github.com"],
  [href*="localhost"],
)) {
  text-decoration: underline dotted;
}
```

Note the exclusion of `localhost` so it works when testing locally via `hugo
server`.


### marking internal links that don't exist {#marking-internal-links-that-don-t-exist}

Here's a broken internal [link]({{< relref "dne" >}}). In my `hugo.toml` I've set broken relative links
(those generated by org-links to non-existent org files) to direct us straight
to a custom 404 page,

```toml
refLinksErrorLevel = 'warning'
refLinksNotFoundURL = '/404.html'
```

and then styled links to the 404 page with a wavy underline:

```css
div#content.garden a[href^="/404.html"] { text-decoration: underline wavy; }
```


### backlinks {#backlinks}

Here is the [resource](https://seds.nl/notes/export_org_roam_backlinks_with_gohugo/) I'm following for generating (using hugo) to generate
backlinks.

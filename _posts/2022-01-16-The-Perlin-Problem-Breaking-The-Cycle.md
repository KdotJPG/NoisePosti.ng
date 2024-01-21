---
layout: post
title: "The Perlin Problem: Breaking the Cycle"
image: /assets/images/the-perlin-problem-moving-past-square-noise/cycle.png
date: 2022-01-16 1
---

In [The Perlin Problem: Moving Past Square Noise](/posts/2022-01-16-The-Perlin-Problem-Moving-Past-Square-Noise.html), I shed light on the axis alignment problems Perlin and related noises carry in their unmitigated forms. I then covered two key tools we have to address them: Simplex-type noise and Domain Rotation. Here, I will recommend a set of best practices we can employ to improve the information we contribute to the field, keeping our content as helpful as possible.

### Overview

If Perlin has the problems we discuss, and solutions are so readily available now, why does the unmitigated noise still see such far-reaching use? The full explanation likely depends on many factors: software patents, old documents, existing libraries, bias to tradition, past shortcomings of alternatives, etc. No single matter is to blame, but what's clear is the room for improvement we have in what we teach.

Whether it be tutorials or forum thread answers, articles or videos, libraries or tech demos, the messages we convey through the media we create play formative roles in our audience's understanding. Developers, as autonomous as they are, are influenced by the resources available to them. The details therein inform the decisions they make, and the material they themselves may pass on. It's on us as contributors to the space to ensure we present ideas accurately, relay up-to-date knowledge, and carefully consider the practices and defaults we're promoting.

I will lead off with the most central point, and expand upon it in the sections that follow.

### Teach Modern Options (avoid "Perlin in a Vacuum")

As driven home in the previous article, Perlin noise carries faults that newer tools address. It's clear the wider set of options we have now, but an overwhelming amount of media still concentrates on bare Perlin.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/article_ridges_perlinframed.png" max_height="372" caption="Mockup of an article framing a topic around Perlin without context." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_perlin.png" max_height="206" caption="Mockup of a forum post recommending uncontextualized Perlin." %}

This path may seem reasonable as an author. Perlin is recognizable, many libraries support it, and it makes it quick to move onto the next point. Perhaps further, as I alluded to above, one isn't familiar with the issues. However, it's important to see through this to the ways this holds us back. A surplus of sources reinforce this gap, when we need more that help us move us forward.

There are several ways we can revise our approaches with this goal in mind.

- We can replace our focus algorithm with Simplex-type, adding any further context as necessary.
  - We may later discover an even better tool, but well-implemented Simplex is one of the best standards to adopt within our current knowledge base.
- We can teach Perlin in clear light of effective measures such as Domain Rotation, preserving its role but promoting revised modes of its use.
  - This doesn't communicate the breadth of the option space, but it's a stronger lens through which to cover the older technique.
- We can also address the subject using general language, deferring the endorsement of any particular tool.
  - Examples include *gradient noise*, *coherent noise*, *procedural noise*, *smooth noise*, and simply *noise*.
  - Specifics can be left to a dedicated section or external links.

<div>
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/article_ridges_general.png" max_height="372" caption="Mockup of an article discussing noise generally." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/presentation_noise.png" max_height="353" caption="Mockup of a presentation slide teaching modern noise information." %}
</div>
<div>
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/article_animation_domainrotated.png" max_height="300" caption="Mockup of an article teaching domain-rotated noise." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_simplex.png" max_height="206" caption="Mockup of a forum post recommending Simplex." %}
</div>

Each of these paths lays a foundation for more constructive modern-day discussions on this topic, but there are a few more considerations to make from here.

### Place Proper Weight

When teaching multiple options for noise, we should strive to convey them in productive fashions.

Some sources teach the availability of multiple noises, but relegate them to sidenotes while continuing to spotlight unmitigated Perlin. This does inform users that the choices exist, but it continues to reinforce poor defaults and standards. Perlin best serves as conceptual stepping stone that leads ultimately into fixes and alternatives.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_backseatedsimplex.png" max_height="280" caption="Mockup of an article mentioning Simplex in passing." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_notbackseatedsimplex.png" max_height="280" caption="Mockup of an article introducing Simplex productively." %}

Relatedly, some material also starts off strong, then claims that the choice doesn't matter. It's true that some examples can be taught using any noise, but there are ways to communicate this without downplaying their differences. It's better to lead with constructive pointers, then clarify the flexibility in a follow-up point.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_diminishchoice.png" max_height="234" caption="Mockup of an article discounting the differences between noises." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_notdiminishchoice.png" max_height="234" caption="Mockup conveying the value with minimal change in message." %}

### Present the Right Reasons (Simplex)

When characterizing something as advantageous over another, it's important to filter through to the relevant factors.

Sometimes media will mention Simplex, but fixate on performance as the reason to use it. Simplex itself does scale better to higher dimensions. It's absolutely faster than Perlin in 5D, but the actual performance of noise in the category will depend on a multitude of factors. It differs between the original and related algorithms, and may not be greater in just two or three dimensions. The biggest reason to use Simplex-type noises is for their improvements in visual isotropy, which lend them more credibility as nature emulators. (See Stefan Gustavson's note [here](https://stackoverflow.com/a/19998734/).)

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_simplexspeed.png" max_height="234" caption="Mockup of a StackExchange answer suggesting Simplex for speed." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_simplexisotropy.png" max_height="234" caption="Mockup suggesting Simplex for its visual isotropy." %}

### Promote Effective Fixes

The previous article demonstrated Domain Rotation as a means of moving the squarest slices of noise outside of the observer's view. It turns out to be a compelling solution for the bulk of Perlin's issues, but not all purported approaches are so effective.

A prevailing misconception is that Ken Perlin's *Improved Noise* removes directional bias from the noise, obviating the need for Simplex or mitigation measures. Perhaps it's rational to draw this conclusion from [the 2002 paper](https://web.archive.org/web/20210902092236id_/https://mrl.cs.nyu.edu/~perlin/paper445.pdf) and note it in one's material. The paper and noise do improve the distribution and smoothness; however, the grid-following tendency remains in the sense that matters. The majority of the Perlin renders shown in these articles use the improvements the paper presents, as do most implementations in libraries. They can all be observed to note such patterns.

A notion also arises that fractal Brownian motion (fBm) resolves the bias, but this isn't so accurate either. Fractal noise is widely useful in its own regard, and can rectify straightness in individual noise layers. However, as reflected in the figures under [Perlin: A Square Noise](2022-01-16-The-Perlin-Problem-Moving-Past-Square-Noise.html#perlin-a-square-noise), its effects are only local, so it can't do much against global trends in the noise.

It's up to us to carefully evaluate claims to this end, and further the ones that hold up.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_suggestfbmfixperlin.png" max_height="328" caption="Mockup of a StackExchange answer suggesting fBm to fix Perlin." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_suggestdomainrotate.png" max_height="328" caption="Mockup of a StackExchange answer suggesting domain rotation." %}

### Clarify Imperfect References
 
When contributing content to a space, we often direct our audience to external supporting resources. This is another area worthy of our consideration. Just as the points in this article find bearing in our own wording, they can also function as model decision factors between one source and another -- or as clarifications when one isn't quite up to par. It's important to avoid passing on substandard information at face value, whether in this field or any other.

Prefacing imperfect links helps us make the most out of more of our resources, especially those that present something worthwhile past a few rough spots.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/reddit_clarifyarticle.png" max_height="115" caption="Mockup of a reddit reply supplementing a link with caveats." %}

### Answer Outside the Box

Thus far, I've placed thread answers and more formal media on equal footing. But if forum posts indeed fall within the broader realm of influential noise content, how might this article's concerns apply to help-topic replies? Algorithm-agnostic questions framed around unmitigated Perlin are quite common matters on forums. Since noise choice presupposition is often a product of the shortcomings of popular resources, it's worth analyzing the larger context of a question to determine what the most productive guidance might be.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/reddit_notoutofbox.png" max_height="304" caption="Mockup of a /r/proceduralgeneration thread with a straight-track answer." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/reddit_outofbox.png" max_height="304" caption="Mockup showing a noise-general-style answer moving into a holistic suggestion." %}

### Avoid the Fractal Mixup

In the [Confusion Clear-up](2022-01-16-The-Perlin-Problem-Moving-Past-Square-Noise.html#confusion-clear-up) section at the beginning the previous article, I remarked on the conflation of Perlin with fractal noise. This is vital to overcome as well, for three primary reasons. First, it suggests that noise is inherently interwoven with octaves and parameters, when countless other ways exist to combine layers. Second, it obfuscates the interchangeability of the underlying algorithm, concealing the availability of options. Finally, it invokes the term Perlin without regard for its relevance. This inhibits phasing it out of conversations where it doesn't apply, and seeds misunderstanding in those where it does.

By properly disentangling *Fractal noise* from *Perlin*, we can communicate more effectively and avoid steering our audience down unintended paths.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/libnoise_falseperlin.png" max_height="300" caption="A screenshot of the <i>libnoise</i> docs which misdefine Perlin. (adjusted)" %}

### Reconsider Dependent Terminology

To wrap up this article, I will call attention to two algorithm-pointed terms whose cause for concern is in line with the above. I'll note both alongside workable alternatives to offer a direction forward.

- ~~**Perlin Worms**~~ -> **Noise Worms** / **Noisy Worms** / **Cave Worms**: An iterative process wherein noise decides the travel directions of "worms" which can carve out tunnels. The technique presents a unique extension beyond the pattern classes typically afforded by noise; however, this Perlin-centric label carries an implicit recommendation for a tool that isn't the only choice. Clarifying this flexibility can mitigate most misunderstanding, but it's also time for a more neutral name.

- ~~**Hello Perlin**~~ -> **Hello Worldgen** / **Hello Noise**: A play on "Hello World", occasional uses of this term have found market share in reference to an introductory exercise on noise generation. We can debate the best title for such a project, but this one certainly does us no good. It spotlights old noise while also directly targeting beginners. We should encourage newcomers to adopt sensible practices based on modern information. This label impedes that message, only to assign one where not truly necessary.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/libnoise_worms.png" max_height="372" caption="Worms libnoise page, seemingly its first mention. (adjusted)" %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/hellonoise.png" max_height="372" caption="\"Hello Noise!\" example." %}

---

### Conclusion

An abundance of media presents noise in ways that trap the field in a cycle. Escaping this pattern starts with what we teach -- indeed, the true *Perlin Problem*, so named in the title, is the state of information. We've enabled one tool so long to dominate our focus, neglecting to properly address its shortcomings. We're caught in a cycle of rehashing old tricks that miss the mark, without the needed sense of progression.

The points I covered on what we communicate are strides we can make to move forward. But if you leave with just one takeaway, let it be this: it's time we moved past counting unmitigated Perlin as default, and teaching it without caveats to others. We have more tools now than we did before, and we're due for a refresh in direction.

---

### Extras

<div markdown="1" style="font-size: 0.85em">

##### Does *Perlin* work as a general term for noise?

Some authors do make efforts to use the term generally, but the [overwhelming](https://adrianb.io/2014/08/09/perlinnoise.html) [consensus](https://en.wikipedia.org/wiki/Perlin_noise) is that it refers to the specific square/cube-based algorithm. Most searches done on the term also lead to existing sources that falter on this article's guidelines. Calling all noise *Perlin*, while perhaps good in spirit, doesn't impart modern understanding of the material. It's better to leave it to its true definition, and focus on noise-general language for noise-general concepts.

##### Doesn't *gradient noise* often also point to Perlin?

It's true that many sources directly refer to Perlin when discussing gradient noise, such as [the libnoise documentation](https://libnoise.sourceforge.net/noisegen/index.html#gradientnoise) and [Unity Shader Graph](https://docs.unity3d.com/Packages/com.unity.shadergraph@6.9/manual/Gradient-Noise-Node.html). However, these are moreso resource correctness issues than problems with the term itself. [Gradient noise](https://en.wikipedia.org/wiki/Gradient_noise) is defined as any generation scheme that draws on slope direction extrapolations as its primary influence for curvature. Plenty of sources, such as [Catlike Coding's tutorials](https://catlikecoding.com/unity/tutorials/pseudorandom-noise/simplex-noise/)\* and [Brian Sharpe's blog](https://briansharpe.wordpress.com/2012/01/13/simplex-noise/), do correctly represent this generality. We can draw upon such sources for accuracy to model, while clarifying the shortcomings of the former where needed.

\* The *Catlike Coding* article contains a dropdown titled *Isn't simplex noise patented?*, which includes a note that reads "There is no compelling reason to use simplex noise instead of Perlin noise," in reference to Ken Perlin's 2002 *Improved Noise*. This is incorrect, for the reasons described in [Promote Effective Fixes](#promote-effective-fixes). 2002 *Improved Noise* does not address the axis alignment characteristics of Perlin noise in the way that Simplex does. Past that, the article presents a lot of helpful insight on the subject.

##### What about Domain Warping (re: *Promote Effective Fixes*)?

I showed Domain Rotation as a method to address visible alignment, and fBm as a tool that doesn't -- but has other uses. Domain Warping, the displacement of coordinates with noise, has been called up as another solution to noise bias, but it isn't such a cut-and-dry fix. Perhaps, for similar reasons that a sufficiently-jittered square/cube grid can produce solid results in cellular noise, Domain Warping might do the same in Perlin. It certainly raises a stronger case than fBm, but one fact to note is that [its effectiveness would be tied to the warping amplitude](/assets/images/the-perlin-problem-moving-past-square-noise/domain_warp_perlin.png). It's also difficult to separate from its other behavior or runtime cost, though it's incredibly useful for its own category of effects. In any case, it doesn't preclude the use of Simplex-type noise or Domain Rotation in tandem. A more compelling fix in this vein would be Brian Sharpe's [Perlin-Surflet-Offset Noise](https://briansharpe.wordpress.com/2012/03/09/modifications-to-classic-perlin-noise/).

Going further, it's also worth noting that different approaches to domain warping exist, varying in both directional uniformity and performance. The traditional technique calls for individual noise instances for each axis to displace, while another draws upon a modified implementation that directly outputs a more spherically-distributed vector. A take on this *vector-valued gradient noise* is in use by FastNoiseLite, and will be the subject of a future article.

</div>

---

{% include the-perlin-problem-moving-past-square-noise/footer.md %}

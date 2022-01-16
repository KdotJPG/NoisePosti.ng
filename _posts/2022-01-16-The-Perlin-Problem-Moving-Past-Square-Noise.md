---
layout: post
title: "The Perlin Problem: Moving Past Square Noise"
image: /assets/images/the-perlin-problem-moving-past-square-noise/title.png
---

Noise plays an elemental role in many genres of procedural generation. One algorithm in particular, Perlin noise, has dominated the conversational spotlight, but it suffers a critical flaw: visible axis alignment. Newer approaches exist which address this problem, but such biased noise still sees widespread use where may not be the right choice. In this article, I will dive deep into this issue, cover two of our most practical options now, and suggest ways we can work towards improving the state of information on this topic.

---

### Confusion Clearup

First, it is important I clarify some terminology, because there are occasional discrepancies between sources.

- **(Coherent) noise**: A type of procedural generation primitive which consists of continuous (typically also smooth) randomized bumps and valleys in an N-dimensional coordinate space. Multiple layers of noise can be combined in different ways to create a wide variety of effects.

- **Perlin noise**: A specific algorithm for generating such noise, which smoothly interpolates between the ramps of pseudorandomly-chosen slope directions (gradient vectors) assigned to each vertex on a square/cube grid.

- **Fractal Brownian motion**: A technique in which multiple layers ("octaves") of some noise are combined to yield a new result with richer detail. Each layer is assigned a different frequency and amplitude. It's often abbreviated *fBm* or simply called *fractal noise*.

Most sources use these terms accordingly, regardless of what their actual content is. However a handful use *Perlin noise* to refer in reality to *fBm*, or the combination of some algorithm with fBm. I'll get back to this later on in the article; for now all you need to know is what I mean by them.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/fBm_OS2S_3D_iXY.png" max_height="384" caption="Fractal Brownian motion performed on a non-Perlin noise." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/perlin_single.png" max_height="384" caption="A single layer of Perlin without fBm." %}

Two additional terms I will use throughout the article are defined as follows:

- **(Noise) algorithm**: A scheme for generating noise which includes certain defining steps.
- **(Noise) implementation**: A specific embodiment of a noise algorithm in code or other media.

---

### Perlin: A Square Noise

Perlin noise, so defined in the previous section, is an algorithm for generating smooth randomness. Tools like this have countless applications in procedural generation, most notably in terrain generation and texture synthesis. Noise creates randomized wave patterns over multiple dimensions, which we can pass through various mathematical formulas to achieve particular results. Perlin has seen wide popularity in the noise space for quite a long time, to the point that it's become a frequent focal point in tutorials and libraries. This noise, however, stumbles in a significant way that needs to be addressed.

Perlin noise is visibly square-biased. It's smooth and random, as formulas call for, but the features and shapes the noise creates fall predominantly in alignment 45 or 90 degrees to the input coordinates. This phenomenon persists globally throughout the noise, offering little break or gap inbetween. Such behavior is ill-fit for applications of this category, especially if a solution is to be upheld as a standard.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/coastlines_perlin.png" max_height="384" caption="FBm Perlin terrain rendered in WorldPainter." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/coastlines_perlin_highlighted.png" max_height="384" caption="Previous image with haphazard segments drawn to highlight bias." %}

#### Why Avoid Squareness?

Whether the goal is realism or fantasy, the purpose of noise is generally the same: to mimic patterns in nature. One of nature's fundamental properties is isotropy, or the even distribution of patterns regardless of angle. Nature doesn't have a grid, and it doesn't pick any directions as favorites. It's worth being able to explore one's terrain, for example, to find cliffs, mountains, valleys, and coastlines that follow many directions rather than primarily those on a compass rose. Procedural primitives that capture this effectively fulfill more of our visual expectations, and immerse us better in the end result. Directional uniformity is central to the role noise plays in procedural generation, just as the curvature itself is.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/Ngerukewid-2016-aerial-view-Luka-Peternel.jpg" max_height="384" caption="Ngerukewid islands showing natural angular variation. <a href='https://en.wikipedia.org/wiki/Palau#/media/File:Ngerukewid-2016-aerial-view-Luka-Peternel.jpg'>Credit</a> (cropped)" %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/Ocean_water_surface_texture_2_-_Martha's_Vineyard.jpg" max_height="384" caption="Ocean water surface showing no particular grid affinity. <a href='https://en.wikipedia.org/wiki/File:Ocean_water_surface_texture_2_-_Martha%27s_Vineyard.JPG'>Credit</a> (cropped)" %}

#### What Causes It?

The Perlin noise algorithm is based on a square, cubic, or in general [hypercubic](https://en.wikipedia.org/wiki/Hypercubic_honeycomb) grid. On each vertex of this grid, it chooses slope directions (gradient vectors), computes their extrapolations, then mixes them together with their neighbors. Its bias is a straightaway consequence of this grid structure, and how the gradients interact on it. New gradients only come in at grid vertices, and their bumps only join together across vertices of the same cell. Features with grid-favored angles merge with no effort, while those with the rest aren't often so successful. The resulting patterns adhere to the grid, thereby following the coordinate directions which form its basis.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/pcontribution.png" max_height="384" caption="Square contribution range of a gradient in Perlin noise." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/pgrads_random_2002_set_slice.png" max_height="384" caption="
    Perlin noise rendering with gradient directions visualized.<br />
    Uses 2D slice of 2002 \"Improved Noise\" gradients.<br />
    Click <a href=\"/assets/images/the-perlin-problem-moving-past-square-noise/pgrads_random_24_normalized_set.png\">here</a> to see one using a more-uniform 2D set.<br />
" %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/pgrads_directional.png" max_height="384" caption="
		Attempts to induce various angular trends in Perlin.<br />
		Horizontal (&#9712;), 45-degree (&#9715;), 22.5-degree (&#9713;), circular (&#9714;).
" %}

#### Not Just Perlin

Perlin has found itself at the forefront of the noise conversation for a while now, so it makes sense to address the problems it has specifically in context of it. That doesn't mean there aren't others that share its issues, though. **Value** and **Value-Cubic** noises, found in many libraries themselves, falter through their use of the same grid structure. While they exhibit different characteristics which lend them to different use cases, their related shortcomings warrant similar critical light.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/value_noise.png" max_height="384" caption="Value noise with fade-curve interpolation." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/value_cubic_noise.png" max_height="384" caption="Value noise with cubic spline interpolation (Value-Cubic)." %}

Value noise displays the most prominent artifacts, resembling a blurred pixel grid with clearly recognizable cells. It's certain, at least, what Perlin has in looks over Value; nonetheless they both look square. Value-Cubic is better, producing more-varied features, but it still struggles to decouple them from the grid layout. Later in the article, I'll circle back to how we can improve the looks of all of these noises. First, though, I'll cover an alternative that some of you might already know.

---

### Simplex(-type) Noise

If you're familiar with any of the noises above, you may have also heard of **Simplex noise**. Simplex noise is a second algorithm by Perlin noise's eponymous author, that was put forth as a successor to his original algorithm. One of its primary selling points is in reducing visible directional bias through the use of a new grid structure. In this section I'll examine this noise, tie in some of my own contributions to the space, and illustrate why noises like this have certain core advantages over Perlin and Value that mark them as better standards.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/simplex_noise_2D.png" max_height="384" caption="2D Simplex noise, single layer." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/simplex_noise_2D_fBm.png" max_height="384" caption="2D Simplex noise with fractal summation." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/coastlines_simplex.png" max_height="384" caption="FBm Simplex noise terrain." %}

#### (Expired?) Patent, Related Noises

Those of you familiar with Simplex may also know of its US patent (US6867776B2), which covered certain implementations and use cases of the noise in 3D+. While thankfully somewhat more refined in scope than other patents, it surely played a role in curbing the noise's popularity. In efforts to alleviate some of these concerns, three contributions I made to the field are **OpenSimplex** (2014), then **OpenSimplex2** (2019) and **OpenSimplex2S** (2017/2019). These noises have gained some traction in applications that would otherwise use Simplex. All four of these *Simplex-type* noises share key details, both inside and out, that relate them to one another.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/os2s_noise_2D.png" max_height="384" caption="2D OpenSimplex2S noise (SuperSimplex)." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/os2_noise_3D.png" max_height="384" caption="2D slice of 3D OpenSimplex2 noise." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/os2s_noise_3D.png" max_height="384" caption="2D slice of 3D OpenSimplex2S noise." %}

#### Inner-Workings

Simplex, the namesake of the noise, refers to the shape in a given number of dimensions with the lowest possible number of vertices. In 2D it's a triangle, in 3D it's a tetrahedron (or triangular pyramid), and in general it has just one more vertex than the number of dimensions. These noises are characterized by their use of grid schemes based on these rather than squares or cubes. Grids like this provide more basis directions, decouple themselves from the coordinate axes, reduce the maximum possible misalignment of any given observation angle, and generally pack the vertices together more tightly. This enables them to better align to the principles founding noise generation, and attunes them better to what we use them for.

Simplex-type noises attach gradients to each grid vertex in much the same manner as Perlin, but combine them differently. Instead of interpolation, they use spherically-bound functions for contribution falloff, which are more portable to arbitrary point layouts. When the gradients are added to the final noise value, each is multiplied by a weight that depends only on the (squared Euclidean) distance to the vertex it comes from. At the sphere boundary, that weight becomes zero, and the gradient no longer contributes. This scheme is shared across these algorithms, whose main differences become the grid constructions they use and how they determine which points are in range.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/contributions_simplex.png" max_height="384" caption="Circular vertex contributions in 2D Simplex noise." %}

#### Do I need to understand all that?

No, not generally. Understanding how noise works internally can be instrumental towards modifying an algorithm, debugging one, or implementing one yourself. It can also be rewarding as a programming and mathematical exercise, and can enable you to create specialized variants where you need. However it isn't necessary as part of simply using the noise. There are numerous freely-available open-source libraries that are well-documented, straightforward to import, and support good quality Simplex-type noise. Libraries get the heavy lifting out of the way, so you only need to worry about what you actually do with the noise.

#### Algorithms & Implementations

All of these noises excel in key areas, however not all variants are created equal. Gradient sets and contribution radii both serve to set one adaptation apart from another.

Gradient sets are the space of possible slope directions that can be assigned to each vertex on the grid. Good sets promote visual isotropy and avoid random tall bumps that throw off the noise's value range. Some older Simplex implementations used less-tuned gradient sets which allow some squareness to sneak back in, so it's worth looking out for their artifacts.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/gradient_difference_simplex_2D.png" max_height="384" caption="
    Gradient set differences in 2D Simplex noise.<br />
    Noise with original gradients (&#9712;); with 24-set (&#9715;).<br />
    Contours around zero with original (&#9713;); with 24-set (&#9714;).
" %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/gradient_difference_os2_3D_stock.png" max_height="384" caption="
		OpenSimplex2 simulating original 3D Simplex gradients (left).<br />
		Same noise with improved, re-oriented gradients (right).
" %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/gradient_difference_os2_3D_iXY.png" max_height="384" caption="
		Less difference if you use domain rotation.<br />
		(We'll get to this shortly!)
" %}

Contribution radius, the primary difference that sets OpenSimplex2S and 2014 OpenSimplex apart from Simplex and OpenSimplex2, affects the contour behavior and overall shape of the noise. Ridged fractal noise, in particular, benefits from softer curvature yielded by a larger radius. However these two algorithms aren't as fast, as they require adding together more vertices.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/ridged_simplex.png" max_height="384" caption="2D Ridged fractal Simplex noise." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/ridged_os2s.png" max_height="384" caption="2D Ridged fractal OpenSimplex2S noise." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/contributions_os2s.png" max_height="384" caption="Larger vertex contributions in 2D OpenSimplex2S noise." %}

Smaller differences in contribution radius can also be found between some 3D+ implementations of Simplex and OpenSimplex2. [The original Simplex paper](https://www.csee.umbc.edu/~olano/s2002c36/ch02.pdf) called for `R² = 0.6` and dimension-specific tuning, while the technically-correct constant that avoids discontinuities is always `R² = 0.5`. Nonetheless, the small-order jumps in value are undetectable in all but the most extreme use cases.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/os2_variation.png" max_height="384" caption="<code class=\"language-plaintext highlighter-rouge\">R² = 0.6</code> vs <code class=\"language-plaintext highlighter-rouge\">R² = 0.5</code> in 3D OpenSimplex2 noise." %}

---

### Domain Rotation

In the section above, I discussed the ways Simplex and its relatives improve visual isotropy in comparison to common cube-grid-based noise algorithms. Here, we'll find out that switching the algorithm isn't necessarily the only route to a dramatic improvement. A surprisingly effective technique, which was seemingly first touched on in [a 2014 article](https://catlikecoding.com/unity/tutorials/simplex-noise/) (bottom), is to rotate the sampling space of the noise. Not to be confused with Domain Warping, this **Domain Rotation** involves transforming a noise's input coordinate via a linear rotation transform. The proper rotation used in the right circumstances can cause the griddiest parts of a noise to disappear from the cross-sections observed. The article didn't dive far into detail or reasoning, but it's a powerful tool and I'll cover it more comprehensively here.

#### Dimensions and Rotation

The first thing to know when using domain rotation to improve the look of noise is that you need at least 3D noise. Indeed, when filling a 2D plane with 2D noise, there is nowhere for the artifacts to go. The noise will just rotate within the plane, leaving its patterns as exposed they were as before. But by using 3D noise, it becomes possible to define rotations that altogether move its internal square planes out of alignment with our view.

The process goes as follows: Choose one coordinate that will point up the main diagonal of the noise grid. Typically in 3D, this will be Y or Z, but it could be any we choose. By deriving a rotation that satisfies this mapping, our rendering plane takes a much nicer slice through the noise.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/grid_unrotated.png" max_height="304" caption="Noise slice visualization through an un-rotated grid." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/grid_rotated.png" max_height="304" caption="Noise slice visualization through a rotated grid." %}

Now it would seem, in general, that we always need one dimension higher (N+1) to create good N-dimensional noise. This is true for 2D, and might be proper in many 3D situations such as texturing spherical planets. In such examples, I would tend toward Simplex-type noise anyway. However in certain cases, we can attain the full benefit using only the same dimensionality of noise. 2D-focused use-cases of 3D noise, such as overhanging voxel terrain and time-varied animations, are prime examples of this. In these scenarios, the third dimension plays more of a supportive role, and can be assigned to the "main diagonal" direction without issue. This is where Domain Rotation truly shines, because we can realize its value with virtually no performance penalty.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/nightfall_rotonoise_terrain.png" max_height="384" caption="
		In-dev terrain in Nightfall, using domain-rotated 3D noise.<br />
		(note: incomplete lighting engine)
" %}

#### Results

Illustrating domain rotation on the three cube-grid algorithms, its effects become clear. Square bias disappears from all three noises, preserving varying other aspects of their character. Perlin sees only modest change in overall appearance, in exchange for a substantial reduction in discernible alignment.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/rotated_perlin.png" max_height="384" caption="2D slice of 3D Perlin noise, unrotated/rotated." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/coastlines_rotoperlin.png" max_height="384" caption="FBm Domain-Rotated Perlin terrain." %}
	
Value noise experiences an extravagant transformation, though it still isn't the cleanest contender. Value-Cubic retains much of its composition, but sheds its grid-founded perforations.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/rotated_value.png" max_height="384" caption="2D slice of 3D Value noise, unrotated/rotated." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/rotated_value_cubic.png" max_height="384" caption="2D slice of 3D Value-Cubic noise, unrotated/rotated." %}

It even works on Simplex and co. to clear up some slight probabilistic asymmetry, though the difference is a little hard to tell here. In truth, I rendered most of the 3D Simplex-type noise figures in this article using this rotation.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/rotated_os2.png" max_height="384" caption="2D slice of 3D OpenSimplex2 noise, unrotated/rotated." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/rotated_os2s.png" max_height="384" caption="2D slice of 3D OpenSimplex2S noise, unrotated/rotated." %}

#### Formula

Here, I will go into the derivation of the formula for the 3D rotation. You can skip over this if you prefer. I'll link a library that offers an implementation of this at the end of the article, and will likely write an article in the future that covers more of the odds and ends of this topic.

As hinted at the beginning of this section, rotations are linear transforms. This means we can express them using matrices, or equivalently by assigning new vectors to each old coordinate direction. For example, `Y` without rotation moves us along `<0, 1, 0>` in the noise. We want to pick new vectors for each coordinate that move them in the directions we would like.

Noise domain rotation will typically single out one coordinate to be treated differently than the rest, e.g. as the vertical, time, or unused direction. Then the rest will, ideally, behave symmetrically to each other w.r.t. the noise grid, so as to simplify convention. The first step is to send this "special" coordinate (e.g. Z `<0, 0, 1>`) up the main diagonal of the noise grid. The vector `<1, 1, 1>` moves in the direction we want, but it's not unit-length. Using a different magnitude changes the frequency of the noise, so to counter that we will divide by its magnitude to get the unit vector we can assign to `Z`. `<1, 1, 1>/length(<1, 1, 1>) = <1, 1, 1>/sqrt(3) = <g, g, g>` where `g ≈ 0.577350269189626`. Note that we don't need to compute any square roots at runtime, just now during development to determine what these vectors should be.

Then to assign the right vectors to X and Z, we want to pick two that are (1) 90 degrees to the vector we just created, (2) 90 degrees to each other, (3) also unit length, and ideally (4) symmetric to each other on the noise grid. It turns out that, because the cross-sectional point layouts of an N-dimensional hypercubic grid where `x+y+z` is an integer are equivalent to the grid structure used in 2014 OpenSimplex, we can just borrow its skew constant `s = (1/sqrt(N+1)-1)/N` (`≈ -0.21132486540518713` for `N=2`). Then we can choose `h` in `X: <1+s, s, h>` and `Y: <s, 1+s, h>` to make them 90 degrees to `<g, g, g>` (or equivalently to `<1, 1, 1>`). To our luck, it will turn out that `g = -h` and also that the resulting vectors themselves are unit length.

Writing this out: `x*<1+s, s, -g> + y*<s, 1+s, -g> + z*<g, g, g>`, we see some opportunity for simplification. `xr = (1+s)*x + s*y + g*z`, `yr = s*x + (1+s)*y + g*z`, `zr = -g*x + -g*y + g*z`. `xr = x + (x+y)*s + g*z`, `yr = y + (x+y)*s + g*z`, `zr = g*z - (x+y)*g`. Then we can write `A = g*z`, `B = x+y`, `C = B*s + A` so that `xr = x + C`, `yr = y + C`, `zr = A - B*g`, and that's what we can use to transform our noise coordinates.

```java
public static void noise3_ImproveXZ(double x, double y, double z) {
	double xz = x + z;
	double s2 = xz * -0.211324865405187;
	double yy = y * 0.577350269189626;
	double xr = x + (s2 + yy);
	double zr = z + (s2 + yy);
	double yr = xz * -0.577350269189626 + yy;

	// Generate noise on coordinate xr, yr, zr
}
```

Here's the same thing implemented for 4D choosing the W coordinate.

```java
public static void noise4_ImproveXYZ(double x, double y, double z, double w) {
	double xyz = x + y + z;
	double s3 = xyz * (-1.0 / 6.0);
	double ww = w * 0.5;
	double xr = x + s3 + ww, yr = y + s3 + ww, zr = z + s3 + ww;
	double wr = xyz * -0.5 + ww;

	// Generate noise on coordinate xr, yr, zr, wr
}
```

You can also chain rotations for successively lower dimensions. Here's an example of that, combined and simplified.

```java
public double noise4_ImproveXYZ_ImproveXZ(double x, double y, double z, double w) {
	double xz = x + z;
	double s2 = xz * -0.21132486540518699998;
	double yy = y * 0.28867513459481294226;
	double ww = w * 0.5;
	double xr = x + (yy + ww + s2), zr = z + (yy + ww + s2);
	double yr = xz * -0.57735026918962599998 + (yy + ww);
	double wr = y * -0.866025403784439 + ww;

	// Generate noise on coordinate xr, yr, zr, wr
}
```

---

### Breaking the Cycle

Above, we discussed some of the tools we have to address squareness and improve visual isotropy in our noise creations. Having such options is paramount in our efforts to create quality work, however it's only one part of the picture.

If Perlin has the problems we speak of, and solutions are so readily available now, why does the unmitigated noise still see such far-reaching use? The full answer likely depends on many factors: software patents, old documents, existing libraries, bias to tradition, past shortcomings of alternatives, etc. No single matter is to blame, but what's clear is the room for improvement we have in what we teach on this subject.

Whether it be tutorials or forum thread answers, articles or videos, libraries or tech demos, the messages we convey through the media we create play formative roles in our audience's understanding. Developers, as autonomous as they are, are influenced by the resources available to them. The details of that information contribute to the decisions they make, and eventually the material they themselves might pass on. It's on us as contributors to the space to ensure we present things accurately, relay up-to-date knowledge, and consider carefully the practices and defaults we're promoting.

In this section, I will recommend a handful of best practices we can employ to improve the information we put out into the field, as well as keep our content as relevant as possible.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/cycle.png" max_height="354" nolink=1 %}

#### Teach Modern Options (avoid "Perlin in a Vacuum")

Thus far in this article, I've discussed the faults Perlin noise carries in its unmitigated form and demonstrated practical tools we have to address them. It's clear now the options we have, but an overwhelming amount of media still concentrates on bare Perlin.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/article_ridges_perlinframed.png" max_height="372" caption="Mockup of an article framing a topic around Perlin without context." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_perlin.png" max_height="206" caption="Mockup of a forum post recommending uncontextualized Perlin." %}

There may be some understandable reasoning to take this path as an author. Perlin is recognizable, it's in a lot of libraries, and makes it quick to move onto the next point. Perhaps further, as I alluded to above, they aren't familiar with the issues. However it's important to see through this to the ways it holds us back. A surplus of sources reinforce this knowledge gap, when we need more that actually help us move forward from it.

There are several ways we can reshape our content to better empower it in this regard.

- We can replace our focus noise with Simplex-type, adding in additional context we deem necessary.
  - It's possible we'll discover an even better tool in the future, but Simplex-type noise is one of the best standards we can adopt and promote given our current knowledge.
- We can teach Perlin in clear light of effective measures such as Domain Rotation, preserving its role but promoting revised modes of its use.
- In certain situations, we can also refer to noise using general language, deferring the endorsement of any particular approach.

<div>
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/article_ridges_general.png" max_height="372" caption="Mockup of an article discussing noise generally." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/presentation_noise.png" max_height="353" caption="Mockup of a presentation slide teaching modern noise information." %}
</div>
<div>
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/article_animation_domainrotated.png" max_height="300" caption="Mockup of an article teaching domain-rotated noise." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_simplex.png" max_height="206" caption="Mockup of a forum post recommending Simplex." %}
</div>

Each of these choices lays a foundation for more constructive modern-day discussions on noise, but there are a few more considerations worth making from here.

#### Place Proper Weight

When we teach multiple options for noise, it's worth also paying mind to how we present them in relation to one another.

Some sources today teach the availability of multiple noises, but relegate them to sidenotes while continuing to place unmitigated Perlin in the spotlight. This does inform users that the choices exist, but it continues to reinforce poor defaults and standards. Perlin in plain form is an excellent conceptual stepping stone, but instilling it as default only keeps us where we are.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_backseatedsimplex.png" max_height="280" caption="Mockup of an article mentioning Simplex in passing." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_notbackseatedsimplex.png" max_height="280" caption="Mockup of an article introducing Simplex productively." %}

Relatedly, some media I've encountered also discusses multiple algorithms, but then claims the choice doesn't matter. Perhaps, in the grand scheme of things, it won't matter what you used in one experiment. This is an understandable position, however certainly it makes a difference what we teach along the way. We have better options now, whose improvements are rooted in key criteria. We should strive to characterize them accurately and with the appropriate supporting information, rather than downplay their differences.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_diminishchoice.png" max_height="234" caption="Mockup of an article discounting the differences between noises." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/blog_notdiminishchoice.png" max_height="234" caption="Mockup conveying the value with minimal change in message." %}

#### Present the Right Reasons (Simplex)

When teaching users about something as an upgrade, it's important to filter through to the differences that actually give it its credence.

Sometimes media will mention Simplex, but fixate on performance as the reason to use it. It's true that Simplex noise has better dimensional computational scaling, and requires fewer vertex computations per evaluation. Simplex is absolutely faster than Perlin in 5D, but the actual performance of noise in the category can depend on a multitude of factors. It varies between algorithms, and may not be greater in just two or three dimensions. The biggest reason to use Simplex-type noises is for their improvements in visual isotropy, which lend them more credibility as nature emulators. Stefan Gustavson made a great note on this [here](https://stackoverflow.com/a/19998734/).

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_simplexspeed.png" max_height="234" caption="Mockup of a StackExchange answer suggesting Simplex for speed." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_simplexisotropy.png" max_height="234" caption="Mockup suggesting Simplex for its visual isotropy." %}

#### Promote Effective Fixes

This article demonstrates domain rotation as a viable fix for the bulk of Perlin's issues. However, not all purported methods for removing noise bias are so effective.

A prevailing idea is that Ken Perlin's 2002 *Improved Noise* removes directional bias from the noise, obviating the need for Simplex or any mitigation measures. Perhaps it's rational to come away from [the 2002 paper](https://web.archive.org/web/20210902092236/https://mrl.cs.nyu.edu/~perlin/paper445.pdf) with this conclusion, and note it in one's material. The paper and noise do make improvements in gradients and smoothness, but the grid-following tendency remains in the sense that matters. The majority of the Perlin figures shown in this article use the improvements the paper presents, as do most implementations in libraries. They can all be observed to note such patterns.

A notion that fBm resolves the bias also arises at times, but isn't so accurate either. Fractal noise has extensive utility in its own right, and can rectify straightness in single noise layers. Its effects are only local, though, so it can't do much against global trends in the noise.

It's up to us to carefully evaluate the claims we hear on this, and further the ones that are accurate.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_suggestfbmfixperlin.png" max_height="328" caption="Mockup of a StackExchange answer suggesting fBm to fix Perlin." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/forum_suggestdomainrotate.png" max_height="328" caption="Mockup of a StackExchange answer suggesting domain rotation." %}

#### Clarify Imperfect Links
 
Often when writing our own articles or replying to questions on forums, we look for outside resources to link. This is another area worthy of our consideration. Just as these points can find bearing in our own wordage, they can also function as model decision factors between one external source and another, or as appropriate clarifications to make when one isn't quite up to par. Where sources contain information substandard at face value, it's important to avoid passing it on as such -- whether it's noise context or any other factor. Including such prefaces enables us make the most out of more of our resources, especially those that teach something worthwhile past a few rough spots.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/reddit_clarifyarticle.png" max_height="115" caption="Mockup of a reddit reply supplementing a link with caveats." %}

#### Answer Outside the Box

Thus far, I've discussed thread answers and more formal media on equal footing. But if forum posts indeed fall within the broader picture of noise-related content, how might these ideas apply to help-topics that are algorithm-agnostic in essence, but framed around using unmitigated Perlin? Surely, development obstacles arise from a vast assortment of scenarios, and furthermore everybody is going to have different preferred styles of engaging and answering on forums. Nonetheless, questions posed in this manner encompass significant land share in the space, and often come from a place of relative unfamiliarity with noise issues rather than thorough research and deliberation. It's worth analyzing the larger context of a question to determine, on the whole, what the most productive guidance to offer might be.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/reddit_notoutofbox.png" max_height="304" caption="Mockup of a /r/proceduralgeneration thread with a straight-track answer." %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/reddit_outofbox.png" max_height="304" caption="Mockup showing a noise-general-style answer moving into a holistic suggestion." %}

#### Avoid the Fractal Mixup

Circling back to the beginning clarification, there has been some confusion between Perlin and fractal noise. This is a vital hurdle to overcome as well for two main reasons. First, it shrouds the understanding of noise as a standalone concept. It teaches it as inherently interwoven with octaves and parameters, robbing focus from the underlying algorithm's interchangeability. Second, it invokes the term Perlin without regard for whether the algorithm is relevant. This inhibits phasing it out of conversations where it doesn't apply, and seeds misunderstanding in those where it does. Through properly disentangling these concepts, in our heads as well as our material, we can communicate more effectively and avoid steering users down unintended paths.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/libnoise_falseperlin.png" max_height="300" caption="A screenshot of the <i>libnoise</i> docs which misdefine Perlin. (adjusted)" %}

#### Reconsider Dependent Terminology

To finish off this section, I want to remark on two algorithm-pointed terms whose cause for concern is in line with the above. This point is less about particular writing dilemmas, and more about revisiting certain terminology in a general sense. I'll note both terms alongside some workable alternatives to offer a direction forward.

- **Perlin Worms** -> **Noisy Worms** or **The Worms Method**: An iterative procedural process wherein noise is used to decide the travel directions of "worms", which can then be tasked with carving out caves. The technique constitutes a unique and interesting extension beyond the patterns typically afforded by noise, however this Perlin-oriented label comes in spite of its versatile noise compatibility. It carries through it an implicit recommendation for something that may not be appropriate -- at least not without some tune-ups. We can all do our parts to clarify such on subjects like this. But the way I see it, it's also time for a more neutral name.

- **Hello Perlin** -> **Hello Worldgen** or **Hello Noise**: A play on "Hello World", I've found this term used on occasion in reference to an introductory exercise on noise generation. We can debate on the best title for such a project, but certainly this one does us no good. It spotlights old noise while also directly targeting beginners. We should encourage newcomers to adopt sensible practices based on modern information. This label impedes that message, only to assign one where we may not genuinely need it.

{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/libnoise_worms.png" max_height="372" caption="Worms libnoise page, seemingly its first mention. (adjusted)" %}
{% include figure.html url="/assets/images/the-perlin-problem-moving-past-square-noise/hellonoise.png" max_height="372" caption="\"Hello Noise!\" example." %}

---

### Conclusion

Perlin noise has problems with squareness which run at odds with its purpose. Simplex and its relatives represent better standards, and Domain Rotation addresses its issues in-place. But the true Perlin Problem, so named in the title, is an informational one. We've enabled one tool so long to dominate our focus, often neglecting to address its shortcomings or truly weigh our options. We're caught in a cycle of rehashing old tricks that miss the mark, without the needed sense of progression.

The points I covered on what we communicate are strides we can make to move forward. But if you leave this article with just one takeaway, let it be this: it's time we moved past counting unmitigated Perlin as default, and teaching it without caveats to others. We have more tools now than we did before, and we're due for a refresh in direction.

---

### Extras

<div markdown="1" style="font-size: 0.85em">

##### Don't these solutions just replace square bias with hex bias?

Sort of, yes and no. The resulting grids (or planar cross-sections) do follow some hexagonal trends, but the additional basis directions combined with lack of cardinal axis alignment function to make them less obvious. Simplex-type noise and domain rotation aren't perfect, but they're among our best options today for visual isotropy within the performance bracket we expect.

##### What about Domain Warping?

I showed Domain Rotation as a method to address visible alignment, and fBm as a tool that doesn't (but has other uses). Domain Warping, the displacement of noise coordinates with more noise, has been called up as a potential solution to noise bias, but it isn't such a cut-and-dry fix. Perhaps, for similar reasons that Cellular/Voronoi noise on a sufficiently-jittered square grid can look fine, Domain Warping is potentially able to move the noise's components around enough that the grid is no longer so obvious. While it's surely more capable than fBm in this regard, one certain fact is that [its effectiveness would depend on the warping amplitude](/assets/images/the-perlin-problem-moving-past-square-noise/domain_warp_perlin.png). It's also difficult to separate from its other behavior or runtime cost, but is incredibly useful for its own category of effects. Plus, there's little reason you couldn't combine it with Simplex-type noise or Domain Rotation anyway.

As a complimentary note, it's worth that I mention there are different approaches to domain warping that vary in both directional uniformity and performance. The traditional technique involves spawning individual new instances of noise for each axis to warp, while another involves modifying a single noise to directly output a more unbiased vector.

##### What if I'm using biased noise for stylistic reasons?

The first question I would ask is if the bias itself is part of the style you're going for, or if there are other parts of its character that serve as truer reasons. If the latter is the case, my general principle is to try and isolate just the favorable parts in the solution. Domain Rotation, as covered here, is a great tool for this in noise. If you're using the bias itself for style, then my only question becomes how your creation sets itself apart from the sea of those where it's more evidently a side effect.

##### What if my world is made of blocks or tiles?

A tempting thought might be: If a game is based on a grid of blocks or tiles, would it not make sense for the terrain to follow that grid? It sounds like a logical parallel, but it can also serve as an intuition trap. The primary reason I tend not to subscribe to this relates to differentiation between scales.

At what *scales* should should the block/tile layout influence the world's form, and in what ways? Should the shape of a vast continent be a consequential product of its small parts? To me, block/tile world generation is generally less about extrapolating their underlying geometric structure into larger trends, and more about making great artistic/visual use of the pieces -- especially where nature-inspired curvature is already in play. Like an image made of pixels doesn't have to follow its own grid, worlds made of blocks or tiles need not remind us of any coordinate-space math under the hood. The aesthetic charm of the tiles/blocks is self-evident on its own, leaving worlds built upon them free to follow more independent and/or natural rulesets that put them to the best overall use.

##### What if the player never sees an aerial view?

The vantage points from which your player observes the world, and what they reveal about it, are worthy and valid considerations when setting priorities during development. It's possible a player will notice different issues at ground-level versus flying high above. However visual anisotropy can manifest in many perspectives, and it may not be just the player camera's point-of-view to account for. In-game maps, promotional material, and possible community-made mods/tools all have potential to showcase the world in more ways, so it can be worth considering how it looks from many possible angles. It's a straightforward win on attention to detail, where the solutions are already readily available.

##### Could Perlin be a general term for noise?

The reformalization of the term Perlin to refer to all manner of smooth coherent noise isn't an alien concept to this world, but I wouldn't count it particularly productive. Most uses of the term today (that aren't the fractal mixup) are pointed at the specific cube-grid algorithm, but perhaps more importantly is the path it leads people down. The term typically guides users to media about the specific algorithm, or to their own pre-existing knowledge about it, rather than towards a greater overall understanding of noise choice.

##### What if my framework already offers just a Perlin/Value function?

Unity and P5.js, for example, offer conveniently-accessible noise options that are poor in quality and directional variety. For Unity, it's `Mathf.PerlinNoise()`, and for P5.js it's the fractal-Value-mislabeled-Perlin `noise()` module. My first step when using platforms like this, whether for my own development or as teaching utilities, is always to import an external noise library. Hopefully, in the future, the teams and leaders behind tools like this will come to understand the ways this skews the product space, and push updates to improve the situations. For now, though, we at least (usually) have the ability to pull in alternatives ourselves.

##### How does the Diamond-Square algorithm play into this?

Diamond-Square is a terrain/texture method early in origin, which involves recursive midpoint value displacement on a pixel grid. The finer-grained control it offers has been praised as a strong point, but it loses out in significant ways on quality and directional variety. The effective calculations it performs aren't unlike fBm Value noise under plain linear interpolation, and its visual results turn out likewise with grid-aligned creases.

##### What about one-dimensional noise?

Visual isotropy doesn't really become a proper concern until you hit at least 2D, so from that standpoint all the choices are essentially equal. But perhaps from a different perspective, it might be worth finding a noise that doesn't cross zero on regular intervals of the number line the way the 1D generalizations of Simplex or Perlin do. 1D Value-Cubic or a grid-point-avoiding line/path through some 2D noise would work nicely to avoid that.

</div>

---

### Libraries

- [FastNoiseLite](https://github.com/Auburn/FastNoiseLite) - Straightforward library for common use cases or for getting started, that supports OpenSimplex2(S) noise as well as 3D domain rotation through a config option.
- [FastNoise2](https://github.com/Auburn/FastNoise2) - Hyper-optimized SIMD-enabled library with support for OpenSimplex2 and an angle-based rotation node.
- [OpenSimplex2](https://github.com/KdotJPG/OpenSimplex2) - My own repository that serves as a home base for OpenSimplex2(S) noise.
- [WebGL noise](https://github.com/ashima/webgl-noise) - GLSL implementations of noise functions including Simplex.

### Resources

- [Perlin vs Simplex](https://www.bit-101.com/blog/2021/07/perlin-vs-simplex/) - BIT-101
- [Making Terrain from Noise](https://www.redblobgames.com/maps/terrain-from-noise/) - Red Blob Games (comprehensive, recurrently-updated noise fundamentals article with primarily-modern information)
- [Simplex Noise Demystified](https://weber.itn.liu.se/~stegu/simplexnoise/simplexnoise.pdf) - Stefan Gustavson
- [Noise Hardware](https://www.csee.umbc.edu/~olano/s2002c36/ch02.pdf) (original Simplex paper) - Ken Perlin

---

###### Thanks goes out to the following who helped proofread this article prior to publishing!
- YUNGNICKYOUNG
- Amit Patel (Red Blob Games)






---
layout: post
title: "Evaluating Multiple Noise Seeds at Once"
image: /assets/images/multi-seedable-noise/title.png
---

Applications built on procedural noise often rely on a network of interconnected noise channels. In certain situations, some of these layers may share a common frequency and differ only in their seeds. This is notably prevalent in cave tunnel generation, biome map sampling, and multi-color blending, where the target patterns can depend fundamentally on the interplay between multiple individual values. As the complexity of a generator grows, so too can the importance of optimization. In this article, I will demonstrate an approach through which multiple seeds of noise can be evaluated using a single efficient function call.

----

### Overview

The execution of most coordinate-evaluated noise algorithms can be divided into three interwoven steps: *vertex discovery*, *vertex computations*, and *result combination*. The function accepts an input coordinate to identify which vertices are in range. The coordinates of the vertices are then hashed based on the seed to derive specific data such as gradient directions, scalar values, or displacements. Finally, the results are combined, accounting for the positions of each vertex relative to the evaluation point.

While the majority of the generated vertex data relies on the seed of the noise layer, certain procedures are influenced solely by the input coordinate, and remain consistent across all layers of a given frequency. This typically includes the entire discovery phase and part of the vertex computation stage. We can leverage this to evaluate multiple seeds at once, duplicating only the seed-dependent operations.

Each noise type examined will include code snippets that demonstrate the refactoring. The full implementations can be found [on GitHub](https://github.com/KdotJPG/Noise-Extras/tree/master/Multi-Seedable Noise).

---

### Type: Simplex

{% include figure.html url="/assets/images/multi-seedable-noise/simplex-plain.png" max_height="384" caption="" caption="Plain 2D slice of 3D Simplex noise." %}

{% include figure.html url="/assets/images/multi-seedable-noise/simplex-triple-channel-ridged-terrain.png" max_height="384" caption="" caption="Ridged terrain using three Simplex channels per octave. Note the trifurcated crests." %}

In Simplex noise, the *vertex discovery* step involves a skew transform to address the internal grid layout, floor/modulo operations to identify the current unit cell, and an ordered edge traversal to isolate the specific contributing vertices. The *vertex computations* include evaluating distance-based falloff curves, selecting gradient vectors, extrapolating them, and multiplying them by the falloff curve values. The final step, *result combination*, is a simple summation of the contributions of each vertex. A more detailed explanation can be found in the [Simplex Noise Demystified](https://web.archive.org/web/20230310204121id_/https://weber.itn.liu.se/~stegu/simplexnoise/simplexnoise.pdf) paper.

While the specifics of the discovery step may differ in [stand-in algorithms](https://github.com/KdotJPG/OpenSimplex2), the subsequent steps remain analogous. All algorithms falling into this *simplex-type* category can be modified to evaluate multiple seeds simultaneously. By sharing the *vertex discovery* and *falloff curve computation* elements across the multiple evaluations, it is only necessary to repeat the *gradient vector selection* and subsequent dependent operations.

See the following implementations before and after applying this change. Note that all of the seed-dependent operations take place inside the *Contribution* method.

<div class="accordion" id="accordionSimplex">
  <div class="accordion-item">
    <h2 class="accordion-header" id="headingTwo">
      <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#accordionSimplex-1" aria-expanded="false" aria-controls="accordionSimplex-1">
        3D Simplex Noise – Single Seed
      </button>
    </h2>
    <div id="accordionSimplex-1" class="accordion-collapse collapse" aria-labelledby="headingTwo">
      <div class="accordion-body">
<div markdown="1">
```cs
public static double Noise3_ImproveXZ(long seed, double x, double y, double z) {

    // Combined skew + rotation to tune noise for XZ horizontal and Y vertical (or time).
    // In place of `(xSkewed, ySkewed, zSkewed) = (x, y, z) + (x + y + z) * Skew3D`
    double xPlusZ = x + z;
    double yRescaled = y * (Root3Over3 * 2.0); // This *2.0 performs the skew.
    double xzOffset = xPlusZ * RotationOrthogonalizer3D + yRescaled;
    double xSkewed = x + xzOffset;
    double ySkewed = yRescaled - xPlusZ * Root3Over3;
    double zSkewed = z + xzOffset;

    int xSkewedBase = FastFloor(xSkewed), ySkewedBase = FastFloor(ySkewed), zSkewedBase = FastFloor(zSkewed);
    long xBasePrimed = xSkewedBase * PRIME_X, yBasePrimed = ySkewedBase * PRIME_Y, zBasePrimed = zSkewedBase * PRIME_Z;

    double xDeltaSkewed = xSkewed - xSkewedBase, yDeltaSkewed = ySkewed - ySkewedBase, zDeltaSkewed = zSkewed - zSkewedBase;
    double unskewOffset = (xDeltaSkewed + yDeltaSkewed + zDeltaSkewed) * Unskew3D;
    double xDelta000 = xDeltaSkewed + unskewOffset, yDelta000 = yDeltaSkewed + unskewOffset, zDelta000 = zDeltaSkewed + unskewOffset;
    double distanceSquared000 = xDelta000 * xDelta000 + yDelta000 * yDelta000 + zDelta000 * zDelta000;
    double xDelta111 = xDelta000 - (1 + 3 * Unskew3D), yDelta111 = yDelta000 - (1 + 3 * Unskew3D), zDelta111 = zDelta000 - (1 + 3 * Unskew3D);
    double distanceSquared111 = xDelta111 * xDelta111 + yDelta111 * yDelta111 + zDelta111 * zDelta111;

    double value = 0;
    if (distanceSquared000 < FalloffRadiusSquared) {
        value += Contribution(seed, distanceSquared000, xBasePrimed, yBasePrimed, zBasePrimed, xDelta000, yDelta000, zDelta000);
    }
    if (distanceSquared111 < FalloffRadiusSquared) {
        value += Contribution(seed, distanceSquared111, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta111, yDelta111, zDelta111);
    }

    bool xGreaterThanY = (xDeltaSkewed > yDeltaSkewed);
    bool xGreaterThanZ = (xDeltaSkewed > zDeltaSkewed);
    bool yGreaterThanZ = (yDeltaSkewed > zDeltaSkewed);

    if (xGreaterThanY && xGreaterThanZ) {
        double xDelta100 = xDelta000 - (1 + Unskew3D), yDelta100 = yDelta000 - Unskew3D, zDelta100 = zDelta000 - Unskew3D;
        double distanceSquared100 = xDelta100 * xDelta100 + yDelta100 * yDelta100 + zDelta100 * zDelta100;
        if (distanceSquared100 < FalloffRadiusSquared) {
            value += Contribution(seed, distanceSquared100, xBasePrimed + PRIME_X, yBasePrimed, zBasePrimed, xDelta100, yDelta100, zDelta100);
        }
    } else if (!xGreaterThanY && yGreaterThanZ) {
        double xDelta010 = xDelta000 - Unskew3D, yDelta010 = yDelta000 - (1 + Unskew3D), zDelta010 = zDelta000 - Unskew3D;
        double distanceSquared010 = xDelta010 * xDelta010 + yDelta010 * yDelta010 + zDelta010 * zDelta010;
        if (distanceSquared010 < FalloffRadiusSquared) {
            value += Contribution(seed, distanceSquared010, xBasePrimed, yBasePrimed + PRIME_Y, zBasePrimed, xDelta010, yDelta010, zDelta010);
        }
    } else {
        double xDelta001 = xDelta000 - Unskew3D, yDelta001 = yDelta000 - Unskew3D, zDelta001 = zDelta000 - (1 + Unskew3D);
        double distanceSquared001 = xDelta001 * xDelta001 + yDelta001 * yDelta001 + zDelta001 * zDelta001;
        if (distanceSquared001 < FalloffRadiusSquared) {
            value += Contribution(seed, distanceSquared001, xBasePrimed, yBasePrimed, zBasePrimed + PRIME_Z, xDelta001, yDelta001, zDelta001);
        }
    }

    if (xGreaterThanZ && yGreaterThanZ) {
        double xDelta110 = xDelta000 - (1 + 2 * Unskew3D), yDelta110 = yDelta000 - (1 + 2 * Unskew3D), zDelta110 = zDelta000 - 2 * Unskew3D;
        double distanceSquared110 = xDelta110 * xDelta110 + yDelta110 * yDelta110 + zDelta110 * zDelta110;
        if (distanceSquared110 < FalloffRadiusSquared) {
            value += Contribution(seed, distanceSquared110, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed, xDelta110, yDelta110, zDelta110);
        }
    } else if (xGreaterThanY && !yGreaterThanZ) {
        double xDelta101 = xDelta000 - (1 + 2 * Unskew3D), yDelta101 = yDelta000 - 2 * Unskew3D, zDelta101 = zDelta000 - (1 + 2 * Unskew3D);
        double distanceSquared101 = xDelta101 * xDelta101 + yDelta101 * yDelta101 + zDelta101 * zDelta101;
        if (distanceSquared101 < FalloffRadiusSquared) {
            value += Contribution(seed, distanceSquared101, xBasePrimed + PRIME_X, yBasePrimed, zBasePrimed + PRIME_Z, xDelta101, yDelta101, zDelta101);
        }
    } else {
        double xDelta011 = xDelta000 - 2 * Unskew3D, yDelta011 = yDelta000 - (1 + 2 * Unskew3D), zDelta011 = zDelta000 - (1 + 2 * Unskew3D);
        double distanceSquared011 = xDelta011 * xDelta011 + yDelta011 * yDelta011 + zDelta011 * zDelta011;
        if (distanceSquared011 < FalloffRadiusSquared) {
            value += Contribution(seed, distanceSquared011, xBasePrimed, yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta011, yDelta011, zDelta011);
        }
    }

    return value;
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static double Contribution(
        long seed, double distanceSquared, long xPrimed, long yPrimed, long zPrimed, double xDelta, double yDelta, double zDelta) {
    long hash = seed ^ xPrimed ^ yPrimed ^ zPrimed;
    hash *= HashMultiplier;
    hash ^= hash >> (64 - Gradients3DBitsConsidered);

    int index = (int)hash & Gradients3DHashMask;
    index = (index * NGradients3DMultiplier) >> Gradients3DBitShift;
    index &= Gradients3DSelectionMask;

    double falloff = Pow4(distanceSquared - FalloffRadiusSquared);
    var gradient = Gradients3D[index];
    return falloff * (gradient.X * xDelta + gradient.Y * yDelta + gradient.Z * zDelta);
}
```
</div>
      </div>
    </div>
  </div>
  <div class="accordion-item">
    <h2 class="accordion-header" id="headingTwo">
      <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#accordionSimplex-2" aria-expanded="false" aria-controls="accordionSimplex-2">
        3D Simplex Noise – Multi-Seed
      </button>
    </h2>
    <div id="accordionSimplex-2" class="accordion-collapse collapse" aria-labelledby="headingTwo">
      <div class="accordion-body">
<div markdown="1">
```cs
public static void Noise3_ImproveXZ(Span<long> seeds, Span<double> destination, double x, double y, double z) {

    // Combined skew + rotation to tune noise for XZ horizontal and Y vertical (or time).
    // In place of `(xSkewed, ySkewed, zSkewed) = (x, y, z) + (x + y + z) * Skew3D`
    double xPlusZ = x + z;
    double yRescaled = y * (Root3Over3 * 2.0); // This *2.0 performs the skew.
    double xzOffset = xPlusZ * RotationOrthogonalizer3D + yRescaled;
    double xSkewed = x + xzOffset;
    double ySkewed = yRescaled - xPlusZ * Root3Over3;
    double zSkewed = z + xzOffset;

    int xSkewedBase = FastFloor(xSkewed), ySkewedBase = FastFloor(ySkewed), zSkewedBase = FastFloor(zSkewed);
    long xBasePrimed = xSkewedBase * PRIME_X, yBasePrimed = ySkewedBase * PRIME_Y, zBasePrimed = zSkewedBase * PRIME_Z;

    for (int i = 0; i < seeds.Length; i++) destination[i] = 0;
    double xDeltaSkewed = xSkewed - xSkewedBase, yDeltaSkewed = ySkewed - ySkewedBase, zDeltaSkewed = zSkewed - zSkewedBase;
    double unskewOffset = (xDeltaSkewed + yDeltaSkewed + zDeltaSkewed) * Unskew3D;
    double xDelta000 = xDeltaSkewed + unskewOffset, yDelta000 = yDeltaSkewed + unskewOffset, zDelta000 = zDeltaSkewed + unskewOffset;
    double distanceSquared000 = xDelta000 * xDelta000 + yDelta000 * yDelta000 + zDelta000 * zDelta000;
    double xDelta111 = xDelta000 - (1 + 3 * Unskew3D), yDelta111 = yDelta000 - (1 + 3 * Unskew3D), zDelta111 = zDelta000 - (1 + 3 * Unskew3D);
    double distanceSquared111 = xDelta111 * xDelta111 + yDelta111 * yDelta111 + zDelta111 * zDelta111;

    if (distanceSquared000 < FalloffRadiusSquared) {
        Contribution(seeds, destination, distanceSquared000, xBasePrimed, yBasePrimed, zBasePrimed, xDelta000, yDelta000, zDelta000);
    }
    if (distanceSquared111 < FalloffRadiusSquared) {
        Contribution(seeds, destination, distanceSquared111, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta111, yDelta111, zDelta111);
    }

    bool xGreaterThanY = (xDeltaSkewed > yDeltaSkewed);
    bool xGreaterThanZ = (xDeltaSkewed > zDeltaSkewed);
    bool yGreaterThanZ = (yDeltaSkewed > zDeltaSkewed);

    if (xGreaterThanY && xGreaterThanZ) {
        double xDelta100 = xDelta000 - (1 + Unskew3D), yDelta100 = yDelta000 - Unskew3D, zDelta100 = zDelta000 - Unskew3D;
        double distanceSquared100 = xDelta100 * xDelta100 + yDelta100 * yDelta100 + zDelta100 * zDelta100;
        if (distanceSquared100 < FalloffRadiusSquared) {
            Contribution(seeds, destination, distanceSquared100, xBasePrimed + PRIME_X, yBasePrimed, zBasePrimed, xDelta100, yDelta100, zDelta100);
        }
    } else if (!xGreaterThanY && yGreaterThanZ) {
        double xDelta010 = xDelta000 - Unskew3D, yDelta010 = yDelta000 - (1 + Unskew3D), zDelta010 = zDelta000 - Unskew3D;
        double distanceSquared010 = xDelta010 * xDelta010 + yDelta010 * yDelta010 + zDelta010 * zDelta010;
        if (distanceSquared010 < FalloffRadiusSquared) {
            Contribution(seeds, destination, distanceSquared010, xBasePrimed, yBasePrimed + PRIME_Y, zBasePrimed, xDelta010, yDelta010, zDelta010);
        }
    } else {
        double xDelta001 = xDelta000 - Unskew3D, yDelta001 = yDelta000 - Unskew3D, zDelta001 = zDelta000 - (1 + Unskew3D);
        double distanceSquared001 = xDelta001 * xDelta001 + yDelta001 * yDelta001 + zDelta001 * zDelta001;
        if (distanceSquared001 < FalloffRadiusSquared) {
            Contribution(seeds, destination, distanceSquared001, xBasePrimed, yBasePrimed, zBasePrimed + PRIME_Z, xDelta001, yDelta001, zDelta001);
        }
    }

    if (xGreaterThanZ && yGreaterThanZ) {
        double xDelta110 = xDelta000 - (1 + 2 * Unskew3D), yDelta110 = yDelta000 - (1 + 2 * Unskew3D), zDelta110 = zDelta000 - 2 * Unskew3D;
        double distanceSquared110 = xDelta110 * xDelta110 + yDelta110 * yDelta110 + zDelta110 * zDelta110;
        if (distanceSquared110 < FalloffRadiusSquared) {
            Contribution(seeds, destination, distanceSquared110, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed, xDelta110, yDelta110, zDelta110);
        }
    } else if (xGreaterThanY && !yGreaterThanZ) {
        double xDelta101 = xDelta000 - (1 + 2 * Unskew3D), yDelta101 = yDelta000 - 2 * Unskew3D, zDelta101 = zDelta000 - (1 + 2 * Unskew3D);
        double distanceSquared101 = xDelta101 * xDelta101 + yDelta101 * yDelta101 + zDelta101 * zDelta101;
        if (distanceSquared101 < FalloffRadiusSquared) {
            Contribution(seeds, destination, distanceSquared101, xBasePrimed + PRIME_X, yBasePrimed, zBasePrimed + PRIME_Z, xDelta101, yDelta101, zDelta101);
        }
    } else {
        double xDelta011 = xDelta000 - 2 * Unskew3D, yDelta011 = yDelta000 - (1 + 2 * Unskew3D), zDelta011 = zDelta000 - (1 + 2 * Unskew3D);
        double distanceSquared011 = xDelta011 * xDelta011 + yDelta011 * yDelta011 + zDelta011 * zDelta011;
        if (distanceSquared011 < FalloffRadiusSquared) {
            Contribution(seeds, destination, distanceSquared011, xBasePrimed, yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta011, yDelta011, zDelta011);
        }
    }
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static void Contribution(
        Span<long> seeds, Span<double> destination, double distanceSquared, long xPrimed, long yPrimed, long zPrimed, double xDelta, double yDelta, double zDelta) {
    long vertexHashComponent = xPrimed ^ yPrimed ^ zPrimed;
    double falloff = Pow4(distanceSquared - FalloffRadiusSquared);

    for (int i = 0; i < seeds.Length; i++) {
        long hash = seeds[i] ^ vertexHashComponent;
        hash *= HashMultiplier;
        hash ^= hash >> (64 - Gradients3DBitsConsidered);

        int index = (int)hash & Gradients3DHashMask;
        index = (index * NGradients3DMultiplier) >> Gradients3DBitShift;
        index &= Gradients3DSelectionMask;

        var gradient = Gradients3D[index];
        destination[i] += falloff * (gradient.X * xDelta + gradient.Y * yDelta + gradient.Z * zDelta);
    }
}
```
</div>
      </div>
    </div>
  </div>
</div>

---

### Type: Perlin

{% include figure.html url="/assets/images/multi-seedable-noise/rotated-perlin-plain.png" max_height="384" caption="" caption="Plain 2D slice of 3D Perlin (without/with domain rotation)" %}
{% include figure.html url="/assets/images/multi-seedable-noise/rotated-perlin-fbm-max-regions.png" max_height="384" caption="" caption="Perlin (domain-rotated 3D slices, fBm, max) forming color-coded regions. Winner chooses color." %}

Perlin noise follows a simpler process to Simplex, and it shares similar seed dependency patterns. While it omits the skew transform (we will still include a [domain rotation](2022-01-16-The-Perlin-Problem-Moving-Past-Square-Noise.html#domain-rotation) for appearance) and ordered edge traversal steps, it retains the floor/modulo operation and seed-independence of its vertex weight computations. Accordingly, the Perlin algorithm can be adapted to support multiple seeds in much the same way as Simplex.

The following code samples demonstrate the implementation of this change. Note the refactoring of the axis interpolations in the multi-seeded variant into individual falloff weight multipliers. This removes the need for intermediate buffers.

<div class="accordion" id="accordionPerlin">
  <div class="accordion-item">
    <h2 class="accordion-header" id="headingTwo">
      <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#accordionPerlin-1" aria-expanded="false" aria-controls="accordionPerlin-1">
        3D Perlin Noise – Single Seed
      </button>
    </h2>
    <div id="accordionPerlin-1" class="accordion-collapse collapse" aria-labelledby="headingTwo">
      <div class="accordion-body">
<div markdown="1">

```cs
public static double Noise3_ImproveXZ(long seed, double x, double y, double z) {

    // Rotation to conceal grid and tune noise for XZ horizontal and Y vertical (or time).
    double xPlusZ = x + z;
    double yRescaled = y * Root3Over3; // No skew here.
    double xzOffset = xPlusZ * RotationOrthogonalizer3D + yRescaled;
    x += xzOffset;
    z += xzOffset;
    y = yRescaled - xPlusZ * Root3Over3;

    int xBase = FastFloor(x), yBase = FastFloor(y), zBase = FastFloor(z);
    double xDelta = x - xBase, yDelta = y - yBase, zDelta = z - zBase;
    long xBasePrimed = xBase * PRIME_X, yBasePrimed = yBase * PRIME_Y, zBasePrimed = zBase * PRIME_Z;
    double inFadeX = Fade(xDelta), inFadeY = Fade(yDelta), inFadeZ = Fade(zDelta);
    double outFadeX = 1 - inFadeX, outFadeY = 1 - inFadeY, outFadeZ = 1 - inFadeZ;

    double grad000 = Gradient(seed, xBasePrimed,          yBasePrimed,            zBasePrimed,           xDelta,     yDelta,     zDelta    );
    double grad001 = Gradient(seed, xBasePrimed,          yBasePrimed,            zBasePrimed + PRIME_Z, xDelta,     yDelta,     zDelta - 1);
    double grad00Z = outFadeZ * grad000 + inFadeZ * grad001;
    double grad010 = Gradient(seed, xBasePrimed,           yBasePrimed + PRIME_Y, zBasePrimed,           xDelta,     yDelta - 1, zDelta    );
    double grad011 = Gradient(seed, xBasePrimed,           yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta,     yDelta - 1, zDelta - 1);
    double grad01Z = outFadeZ * grad010 + inFadeZ * grad011;
    double grad0YZ = outFadeY * grad00Z + inFadeY * grad01Z;
    double grad100 = Gradient(seed, xBasePrimed + PRIME_X, yBasePrimed,           zBasePrimed,           xDelta - 1, yDelta,     zDelta    );
    double grad101 = Gradient(seed, xBasePrimed + PRIME_X, yBasePrimed,           zBasePrimed + PRIME_Z, xDelta - 1, yDelta,     zDelta - 1);
    double grad10Z = outFadeZ * grad100 + inFadeZ * grad101;
    double grad110 = Gradient(seed, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed,           xDelta - 1, yDelta - 1, zDelta    );
    double grad111 = Gradient(seed, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta - 1, yDelta - 1, zDelta - 1);
    double grad11Z = outFadeZ * grad110 + inFadeZ * grad111;
    double grad1YZ = outFadeY * grad10Z + inFadeY * grad11Z;
    double gradXYZ = outFadeX * grad0YZ + inFadeX * grad1YZ;

    return gradXYZ;
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static double Gradient(long seed, long xPrimed, long yPrimed, long zPrimed, double xDelta, double yDelta, double zDelta) {
    long hash = seed ^ xPrimed ^ yPrimed ^ zPrimed;
    hash *= HashMultiplier;
    hash ^= hash >> (64 - Gradients3DBitsConsidered);

    int index = (int)hash & Gradients3DHashMask;
    index = (index * NGradients3DMultiplier) >> Gradients3DBitShift;
    index &= Gradients3DSelectionMask;

    var gradient = Gradients3D[index];
    return gradient.X * xDelta + gradient.Y * yDelta + gradient.Z * zDelta;
}
```
</div>
      </div>
    </div>
  </div>
  <div class="accordion-item">
    <h2 class="accordion-header" id="headingTwo">
      <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#accordionPerlin-2" aria-expanded="false" aria-controls="accordionPerlin-2">
        3D Perlin Noise – Multi Seed
      </button>
    </h2>
    <div id="accordionPerlin-2" class="accordion-collapse collapse" aria-labelledby="headingTwo">
      <div class="accordion-body">
<div markdown="1">

```cs
public static void Noise3_ImproveXZ(Span<long> seeds, Span<double> destination, double x, double y, double z) {

    // Rotation to conceal grid and tune noise for XZ horizontal and Y vertical (or time).
    double xPlusZ = x + z;
    double yRescaled = y * Root3Over3; // No skew here.
    double xzOffset = xPlusZ * RotationOrthogonalizer3D + yRescaled;
    x += xzOffset;
    z += xzOffset;
    y = yRescaled - xPlusZ * Root3Over3;

    int xBase = FastFloor(x), yBase = FastFloor(y), zBase = FastFloor(z);
    double xDelta = x - xBase, yDelta = y - yBase, zDelta = z - zBase;
    long xBasePrimed = xBase * PRIME_X, yBasePrimed = yBase * PRIME_Y, zBasePrimed = zBase * PRIME_Z;
    double inFadeX = Fade(xDelta), inFadeY = Fade(yDelta), inFadeZ = Fade(zDelta);
    double outFadeX = 1 - inFadeX, outFadeY = 1 - inFadeY, outFadeZ = 1 - inFadeZ;

    for (int i = 0; i < seeds.Length; i++) destination[i] = 0;
    Gradient(seeds, destination, xBasePrimed,           yBasePrimed,           zBasePrimed,           xDelta,     yDelta,     zDelta,     outFadeX * outFadeY * outFadeZ);
    Gradient(seeds, destination, xBasePrimed,           yBasePrimed,           zBasePrimed + PRIME_Z, xDelta,     yDelta,     zDelta - 1, outFadeX * outFadeY * inFadeZ );
    Gradient(seeds, destination, xBasePrimed,           yBasePrimed + PRIME_Y, zBasePrimed,           xDelta,     yDelta - 1, zDelta,     outFadeX * inFadeY  * outFadeZ);
    Gradient(seeds, destination, xBasePrimed,           yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta,     yDelta - 1, zDelta - 1, outFadeX * inFadeY  * inFadeZ );
    Gradient(seeds, destination, xBasePrimed + PRIME_X, yBasePrimed,           zBasePrimed,           xDelta - 1, yDelta,     zDelta,     inFadeX  * outFadeY * outFadeZ);
    Gradient(seeds, destination, xBasePrimed + PRIME_X, yBasePrimed,           zBasePrimed + PRIME_Z, xDelta - 1, yDelta,     zDelta - 1, inFadeX  * outFadeY * inFadeZ );
    Gradient(seeds, destination, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed,           xDelta - 1, yDelta - 1, zDelta,     inFadeX  * inFadeY  * outFadeZ);
    Gradient(seeds, destination, xBasePrimed + PRIME_X, yBasePrimed + PRIME_Y, zBasePrimed + PRIME_Z, xDelta - 1, yDelta - 1, zDelta - 1, inFadeX  * inFadeY  * inFadeZ );
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static void Gradient(
        Span<long> seeds, Span<double> destination, long xPrimed, long yPrimed, long zPrimed, double xDelta, double yDelta, double zDelta, double falloff) {
    long vertexHashComponent = xPrimed ^ yPrimed ^ zPrimed;
    double xDeltaWithFalloff = xDelta * falloff, yDeltaWithFalloff = yDelta * falloff, zDeltaWithFalloff = zDelta * falloff;

    for (int i = 0; i < seeds.Length; i++) {
        long hash = seeds[i] ^ vertexHashComponent;
        hash *= HashMultiplier;
        hash ^= hash >> (64 - Gradients3DBitsConsidered);

        int index = (int)hash & Gradients3DHashMask;
        index = (index * NGradients3DMultiplier) >> Gradients3DBitShift;
        index &= Gradients3DSelectionMask;

        var gradient = Gradients3D[index];
        destination[i] += gradient.X * xDeltaWithFalloff + gradient.Y * yDeltaWithFalloff + gradient.Z * zDeltaWithFalloff;
    }
}
```
</div>
      </div>
    </div>
  </div>
</div>

---

### Type: Cellular

{% include figure.html url="/assets/images/multi-seedable-noise/cellular-plain.png" max_height="384" caption="" caption="Plain `F1²` cellular noise (2D slice of 3D)" %}

{% include figure.html url="/assets/images/multi-seedable-noise/cellular-tubular-mesh-raycast.png" max_height="384" caption="" caption="Smooth intersection between two channels of `F1²/F2²` cellular noise." %}

Cellular noise, also labeled *Worley* or *Voronoi* noise, has a distinct evaluation procedure. Instead of mixing gradient ramps, it generates its geometric appearance by tracking the distances of pseudorandomly-distributed points -- typically displaced grid vertices. The smallest values corresponding to the closest points' distances are then passed through any final calculations, and a result is returned. In spite of its differences, however, its core implementation pattern still aligns with the three-step mold.

Among the various conceptualizations of cellular noise, we will focus on a specific variant: one involving a cube grid for simplicity, a full displacement range for quality, skip conditions for efficiency, and a group-sorted iteration order to faciliate the skip conditions. In this implementation, there is no grid skew step -- only an optional rotation, and one point is chosen within each cube as an *offset* from its originating vertex. Evaluation entails checking every base cell that could contain a point that would impact the final result, until it is certain that the closest point is found. We will also use a simple `F1²` return result, which corresponds to the unprocessed squared Euclidean distance to the closest point.

Note the behavior of the skip conditions in the following implementations. In the multi-seeded embodiment, it must consider an aggregate bound across the entire channel set.

<div class="accordion" id="accordionCellular">
  <div class="accordion-item">
    <h2 class="accordion-header" id="headingTwo">
      <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#accordionCellular-1" aria-expanded="false" aria-controls="accordionCellular-1">
        3D Cellular Noise – Single Seed
      </button>
    </h2>
    <div id="accordionCellular-1" class="accordion-collapse collapse" aria-labelledby="headingTwo">
      <div class="accordion-body">
<div markdown="1">

```cs
public static double Noise3_ImproveXZ(long seed, double x, double y, double z) {

	// Rotation to further conceal grid and tune noise for XZ horizontal and Y vertical (or time).
	double xPlusZ = x + z;
	double yRescaled = y * Root3Over3; // No skew here.
	double xzOffset = xPlusZ * RotationOrthogonalizer3D + yRescaled;
	x += xzOffset;
	z += xzOffset;
	y = yRescaled - xPlusZ * Root3Over3;

	int xBase = ExtraMath.FastFloor(x), yBase = ExtraMath.FastFloor(y), zBase = ExtraMath.FastFloor(z);
	double xDelta = x - xBase, yDelta = y - yBase, zDelta = z - zBase;
	long xBasePrimed = xBase * PRIME_X, yBasePrimed = yBase * PRIME_Y, zBasePrimed = zBase * PRIME_Z;

	// Axis components for calculating minimum distances to the grid cells. Used for skip conditions.
	Span<double> cellDeltasSquared = stackalloc double[] {
		(xDelta + 1) * (xDelta + 1), xDelta * xDelta, 0, (xDelta - 1) * (xDelta - 1), (xDelta - 2) * (xDelta - 2),
		(yDelta + 1) * (yDelta + 1), yDelta * yDelta, 0, (yDelta - 1) * (yDelta - 1), (yDelta - 2) * (yDelta - 2),
		(zDelta + 1) * (zDelta + 1), zDelta * zDelta, 0, (zDelta - 1) * (zDelta - 1), (zDelta - 2) * (zDelta - 2),
	};

	double distanceSquaredToClosest = MaxDistanceSquaredToClosest;
	bool groupHadCellsInRange = true;

	for (int cellIndex = 0; cellIndex < Cells.Length; cellIndex++) {
		ref readonly Cell cell = ref Cells[cellIndex]; // Using `ref` here seems to drastically improve performance.

		// If the last group didn't have any cells in range, then we know none of the rest will either.
		if (cell.IsGroupStart) {
			if (!groupHadCellsInRange) break;
			groupHadCellsInRange = false;
		}

		// Check if the cell is close enough to contain a point closer than the closest one we've found so far.
		if (distanceSquaredToClosest - cellDeltasSquared[cell.CellDeltaSquaredIndexZ] <=
			cellDeltasSquared[cell.CellDeltaSquaredIndexX] + cellDeltasSquared[cell.CellDeltaSquaredIndexY]) continue;
		groupHadCellsInRange = true;

		double distanceSquaredHere = CalculateDistanceSquared(seed,
			xBasePrimed + cell.PrimeRelativeX, yBasePrimed + cell.PrimeRelativeY, zBasePrimed + cell.PrimeRelativeZ,
			xDelta + cell.DeltaRelativeX, yDelta + cell.DeltaRelativeY, zDelta + cell.DeltaRelativeZ);

		if (distanceSquaredHere < distanceSquaredToClosest) {
			distanceSquaredToClosest = distanceSquaredHere;
		}
	}

	return distanceSquaredToClosest;
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static double CalculateDistanceSquared(long seed, long xPrimed, long yPrimed, long zPrimed, double xDelta, double yDelta, double zDelta) {
	long hash = seed ^ xPrimed ^ yPrimed ^ zPrimed;
	hash *= HashMultiplier;

	double xDeltaDisplaced = xDelta - (((hash >> 43) ^ hash) & DisplacementHashMask) * DisplacementHashRescale;
	double yDeltaDisplaced = yDelta - (((hash >> 43) ^ (hash >> 22)) & DisplacementHashMask) * DisplacementHashRescale;
	double zDeltaDisplaced = zDelta - ((hash >> 43) & DisplacementHashMask) * DisplacementHashRescale;

	return xDeltaDisplaced * xDeltaDisplaced + yDeltaDisplaced * yDeltaDisplaced + zDeltaDisplaced * zDeltaDisplaced;
}
```
</div>
      </div>
    </div>
  </div>
  <div class="accordion-item">
    <h2 class="accordion-header" id="headingTwo">
      <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#accordionCellular-2" aria-expanded="false" aria-controls="accordionCellular-2">
        3D Cellular Noise – Multi-Seed
      </button>
    </h2>
    <div id="accordionCellular-2" class="accordion-collapse collapse" aria-labelledby="headingTwo">
      <div class="accordion-body">
<div markdown="1">

```cs
public static void Noise3_ImproveXZ(Span<long> seeds, Span<double> destination, double x, double y, double z) {

	// Rotation to further conceal grid and tune noise for XZ horizontal and Y vertical (or time).
	double xPlusZ = x + z;
	double yRescaled = y * Root3Over3; // No skew here.
	double xzOffset = xPlusZ * RotationOrthogonalizer3D + yRescaled;
	x += xzOffset;
	z += xzOffset;
	y = yRescaled - xPlusZ * Root3Over3;

	int xBase = ExtraMath.FastFloor(x), yBase = ExtraMath.FastFloor(y), zBase = ExtraMath.FastFloor(z);
	double xDelta = x - xBase, yDelta = y - yBase, zDelta = z - zBase;
	long xBasePrimed = xBase * PRIME_X, yBasePrimed = yBase * PRIME_Y, zBasePrimed = zBase * PRIME_Z;

	// Axis components for calculating minimum distances to the grid cells
	Span<double> cellDeltasSquared = stackalloc double[] {
		(xDelta + 1) * (xDelta + 1), xDelta * xDelta, 0, (xDelta - 1) * (xDelta - 1), (xDelta - 2) * (xDelta - 2),
		(yDelta + 1) * (yDelta + 1), yDelta * yDelta, 0, (yDelta - 1) * (yDelta - 1), (yDelta - 2) * (yDelta - 2),
		(zDelta + 1) * (zDelta + 1), zDelta * zDelta, 0, (zDelta - 1) * (zDelta - 1), (zDelta - 2) * (zDelta - 2),
	};

	for (int i = 0; i < seeds.Length; i++) destination[i] = MaxDistanceSquaredToClosest;
	double maxDistanceSquaredToClosest = MaxDistanceSquaredToClosest;
	bool groupHadCellsInRange = true;

	for (int cellIndex = 0; cellIndex < Cells.Length; cellIndex++) {
		ref readonly Cell cell = ref Cells[cellIndex]; // Using `ref` here seems to make a huge difference.

		// If the last group didn't have any cells in range, then we know none of the rest will either.
		if (cell.IsGroupStart) {
			if (!groupHadCellsInRange) break;
			groupHadCellsInRange = false;
		}

		// Check if the cell is close enough to contain a point closer than the closest one we've found so far.
		if (maxDistanceSquaredToClosest - cellDeltasSquared[cell.CellDeltaSquaredIndexZ] <=
			cellDeltasSquared[cell.CellDeltaSquaredIndexX] + cellDeltasSquared[cell.CellDeltaSquaredIndexY]) continue;
		groupHadCellsInRange = true;

		maxDistanceSquaredToClosest = CalculateAndUpdateDistanceSquared(seeds, destination,
			xBasePrimed + cell.PrimeRelativeX, yBasePrimed + cell.PrimeRelativeY, zBasePrimed + cell.PrimeRelativeZ,
			xDelta + cell.DeltaRelativeX, yDelta + cell.DeltaRelativeY, zDelta + cell.DeltaRelativeZ);
	}
}

[MethodImpl(MethodImplOptions.AggressiveInlining)]
private static double CalculateAndUpdateDistanceSquared(
		Span<long> seeds, Span<double> destination, long xPrimed, long yPrimed, long zPrimed, double xDelta, double yDelta, double zDelta) {
	long cellHashComponent = xPrimed ^ yPrimed ^ zPrimed;
	double maxDistanceSquaredToClosest = 0;

	for (int i = 0; i < seeds.Length; i++) {
		long hash = seeds[i] ^ cellHashComponent;
		hash *= HashMultiplier;

		double xDeltaDisplaced = xDelta - (((hash >> 43) ^ hash) & DisplacementHashMask) * DisplacementHashRescale;
		double yDeltaDisplaced = yDelta - (((hash >> 43) ^ (hash >> 22)) & DisplacementHashMask) * DisplacementHashRescale;
		double zDeltaDisplaced = zDelta - ((hash >> 43) & DisplacementHashMask) * DisplacementHashRescale;

		double distanceSquared = xDeltaDisplaced * xDeltaDisplaced + yDeltaDisplaced * yDeltaDisplaced + zDeltaDisplaced * zDeltaDisplaced;
		if (distanceSquared < destination[i]) {
			destination[i] = distanceSquared;
		}

		if (destination[i] > maxDistanceSquaredToClosest) {
			maxDistanceSquaredToClosest = destination[i];
		}
	}

	return maxDistanceSquaredToClosest;
}
```
</div>
      </div>
    </div>
  </div>
</div>

---

### Performance

The following graphs depict the total measured runtime, as a function of the number of seeds, to populate a 256³ volume with noise at a frequency of 0.01.

{% include figure.html url="/assets/images/multi-seedable-noise/chart-simplex-total.svg" max_width="400" caption="" alt="Simplex Noise Total Runtime. The standard implementation runs 1,533ms for one seed up to 15,688ms for twelve seeds in a roughly linear fashion. The multi-seedable implementation runs 1,721ms for one seed up to 3,100ms for twelve." %}
{% include figure.html url="/assets/images/multi-seedable-noise/chart-perlin-total.svg" max_width="400" caption="" alt="Perlin Noise Total Runtime. The standard implementation runs 1,699ms for one seed up to 17,550ms for twelve seeds in a roughly linear fashion. The multi-seedable implementation runs 1,816ms for one seed up to 4,678ms for twelve." %}
{% include figure.html url="/assets/images/multi-seedable-noise/chart-cellular-total.svg" max_width="400" caption="" alt="Cellular Noise Total Runtime. The standard implementation runs 1,386ms for one seed up to 18,043ms for twelve seeds in a linear fashion. The multi-seedable implementation runs 1,433ms for one seed up to 11,01ms for twelve." %}

The next three capture the amortized runtime per individual value produced.

{% include figure.html url="/assets/images/multi-seedable-noise/chart-simplex-amortized.svg" max_width="400" caption="" alt="Simplex Noise Amortized Runtime. The standard implementation runs 91ns for one seed down to 78ns per seed for twelve seeds. The multi-seedable implementation drops quickly down from 103ns for one seed and gradually reaches 15ns per seed for twelve." %}
{% include figure.html url="/assets/images/multi-seedable-noise/chart-perlin-amortized.svg" max_width="400" caption="" alt="Perlin Noise Amortized Runtime. The standard implementation runs 101ns for one seed down to 87ns per seed for twelve seeds. The multi-seedable implementation drops quickly down from 108ns for one seed and gradually reaches 23ns per seed for twelve." %}
{% include figure.html url="/assets/images/multi-seedable-noise/chart-cellular-amortized.svg" max_width="400" caption="" alt="Cellular Noise Amortized Runtime. The standard implementation falls consistently in the 82-90ns range a rough increasing trend. The multi-seedable implementation drops quickly from 85ns for one seed down to the 50-60ns range, with a slight increasing trend starting at around 8 seeds." %}

Several notable observations can be drawn from this data.

First, the multi-seeded noise implementations display measurable improvements in efficiency as the number of seeds increases. Among the evaluated noise types, Simplex showcases the most gradual growth in runtime, followed by Cellular, with Perlin falling in last. This discrepancy in the technique's efficacy appears correlated with the intensity of the seed-independent operations in each noise type. Simplex involves grid transformations and falloff functions, this particular Cellular implementation performs some pre-processing for its skip conditions, all while Perlin requires only a few smoothstep evaluations. As a result, it is difficult to optimize Perlin to the same degree with this technique.

Secondly, Cellular's amortized runtime curves convey non-monotonic trends, which might relate to the skip conditions in this implementation. The early stopping conditions significantly improve the algorithm's initial performance, as it would otherwise process all 81 possible points every evaluation, but can become less likely to take effect as more seeds' distances are processed in parallel. The increasing trend for the individual evaluator may relate to discrepancies at the hardware branch prediction level. It's worth noting that the 3D cellular functions in many popular libraries only check 27 distances, but lack the early return functionality present in the example here.

Furthermore, for a single seed, the standard evaluators remain marginally more efficient. This is to be expected, as the loops and dereferencing in the multi-seedable implementations introduce some overhead into the procedures. Once two or more seeds are in play, all three multi-seedable implementations take the lead.

Finally, the absolute efficiencies of the multi-seed variants follow the same order as the standard evaluators: Simplex boasts the shortest overall runtimes, followed by Perlin in close second, and Cellular taking upwards of three times as long as either to produce its results. Simplex/Perlin are typically performance-competitive in modest dimensionalities, so their similarities here are not unexpected. Cellular's higher numbers are of no surprise either, as its numerous distance checks tend to make the noise more expensive to compute.

---

### Additional Considerations

**Aligned Noise Grids**: Combining noise layers with identical offsets and frequencies can occasionally reveal their shared underlying grid structure. Mitigation measures may include implementing noise using an irregular point layout, or foregoing the optimization in favor of separate offsets per layer. Note that fixed known-good offsets may be preferable to seed-dependent displacements, as the latter can alter the global probability space of the generator between seeds.

{% include figure.html url="/assets/images/multi-seedable-noise/max-ridge-without-offsets.png" max_width="384" caption="`1-2*avg(abs(value0), abs(value1), ...)` on multi-seed noise" alt="Noise producing clear bright hexagonal regions along a regular grid." %}

{% include figure.html url="/assets/images/multi-seedable-noise/max-ridge-with-offsets.png" max_width="384" caption="The same formula on individually-offset layers" alt="Cloudier but similar noise without the vislble regularities." %}

**Coordinate Premapping**: Efficiency improvements can already be achieved in the standard algorithms by extracting the rotation/skew transforms. Typically, in Cellular and Perlin, the rotations are not even present -- though in Perlin I certainly recommend including one. These could be performed once at the top level while the remaining logic proceeds as normal. This can also serve to optimize fractal noise, as scalar frequency multipliers can still be applied in the transformed vector space.

{% include figure.html url="/assets/images/multi-seedable-noise/chart-simplex-fractal-total.svg" max_width="400" caption="" alt="Simplex Fractal Noise Total Runtime. The standard evaluator took 399ms through 3,672ms to populate a 256³ region with one through eight octaves of fractal noise. The coordinate premapping technique took 374-3,189ms." %}

{% include figure.html url="/assets/images/multi-seedable-noise/chart-simplex-fractal-amortized.svg" max_width="400" caption="" alt="Simplex Fractal Noise Total Runtime. 24ns-27ns for the standard evaluator; 22-24ns with premapping. There is a very slight increasing trend." %}

**Cellular Implementation Disparities**: As covered, other cellular noise implementations may yield different efficiency curves. Here are the cellular graphs from the above section with the results from [FastNoiseLite](https://github.com/Auburn/FastNoiseLite)'s single-seed evaluator added. Note that the same domain rotation is applied via the library's `RotationType3D` configuration field.

{% include figure.html url="/assets/images/multi-seedable-noise/chart-cellular-total-with-fnl.svg" max_width="400" caption="" alt="Cellular Noise Total Runtime with FastNoiseLite added. Its timings run from 1,644ms to 18,482ms. It starts out with a very slight advantage over the standard implementation tested in this article, then gradually converges toward it." %}

{% include figure.html url="/assets/images/multi-seedable-noise/chart-cellular-amortized-with-fnl.svg" max_width="400" caption="" alt="Cellular Noise Amortized Runtime including FastNoiseLite. FNL runs from 98ns down to 92ns in a roughly linear fashion. It starts out with a very slight advantage over the standard implementation tested in this article, then gradually converges toward it." %}

**Cellular Distance Order**: Many applications of cellular noise, such as the tubular mesh depicted in the section's leading figure, call for more point information than the closest distance (`F1`). While the presented code handles this basic case, scenarios like the one illustrated also require `F2` (the second-closest distance). Related processes, such as sparse convolution, may even rely on all samples within a given radius, regardless of their relative distances. The multi-seed technique extends naturally to these additional requirements. However, the resulting implementation may exhibit different efficiency characteristics due to considerations surrounding when calculations can be skipped.

**Value Noises**: This technique is applicable to the various flavors of Value noise as well, if you're so inclined. As with Perlin, I recommend checking the [previously mentioned article](https://noiseposti.ng/posts/2022-01-16-The-Perlin-Problem-Moving-Past-Square-Noise.html#not-just-perlin) for a comprehensive understanding of their visual limitations and mitigation measures.

**Numerical Discrepancies**: In all of the multi-seedable refactorings presented, some floating point operations were reordered to streamline the seed loops or to ensure that independent operations are computed only once. While the adjusted noise procedures are mathematically equivalent to their single-seeded counterparts, they [may not produce numerically-identical results](https://en.wikipedia.org/wiki/Floating-point_arithmetic#Accuracy_problems). In applications depending on precise output parity, it may be necessary to modify the implementations to match -- or commit to one or the other.

**Data-Driven Decisions**: This trick is straightforward to apply in natively-coded generators where developers are afforded precise control over the execution flow. However, in data-driven applications, there is a choice to make. Developers can opt to expose explicit configuration options that enable a noise node to produce multiple values. Alternatively, it can be applied as an automatic, hidden transformation, requiring formula artists only to be aware of the conditions under which it takes effect. In situations where package authors do not have influence on the host codebase, such optimizations may be outside the realm of possibility.

**SIMD**: Although conceptually reminiscent, it's worth acknowledging a concept that this article has not delved into: SIMD (Single Instruction, Multiple Data). SIMD-based implementations use platform-specific instruction sets to compute entire algorithms on multiple inputs in parallel. [FastNoise2](https://github.com/Auburn/FastNoise2), a noise libary built around these instruction sets, is capable of evaluating up to 16 coordinate-seed pairs simultaneously depending on machine architecture. There is, moreover, no theoretical reason that the two approaches could not be combined.

**Beware Premature Optimization**: Lastly, it's important to consider whether this optimization should be prioritized. Like any performance enhancement, its necessity depends on the specific requirements and constraints of the system. In most cases, focusing on achieving the desired results first and addressing performance concerns when necessary is the best approach. Determining the applicability of this optimization technique should be based on the project's goals and time constraints.

---

### Conclusion

Combining identically-configured noise channels into a single function call can considerably improve the performance characteristics of a generator. By repeating only the seed-dependent steps, this optimization yields measurable speed increases -- nearly doubling the efficiency of Simplex for only four layers. While the situations where this technique applies can be specific, and certain tradeoffs may need to be considered, it provides promising potential to accelerate a host of common scenarios.
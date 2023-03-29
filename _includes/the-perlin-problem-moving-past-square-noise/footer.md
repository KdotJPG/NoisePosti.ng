### Libraries

- [FastNoiseLite](https://github.com/Auburn/FastNoiseLite) - Straightforward library for common use cases or for getting started, that supports OpenSimplex2(S) noise as well as 3D domain rotation through a configuration option.
- [FastNoise2](https://github.com/Auburn/FastNoise2) - Hyper-optimized SIMD-enabled library with support for Simplex-type noise and an angle-based rotation node*.
- [OpenSimplex2](https://github.com/KdotJPG/OpenSimplex2) - My own repository that serves as a home base for OpenSimplex2(S) noise.
- [WebGL noise](https://github.com/ashima/webgl-noise) - GLSL implementations of noise functions including Simplex.

<div markdown="1" style="font-size: 0.85em">

\* 3D-only. Use Euler angles `-0.2617994, 0.6154797, -0.7853982` for XY-horizontal, or `-0.6319143, 0.2129302, 0.6319143` for XZ-horizontal.

</div>

### Resources

- [BIT-101: Perlin vs Simplex](https://www.bit-101.com/blog/2021/07/perlin-vs-simplex/)
- [Red Blob Games: Making Maps with Noise Functions](https://www.redblobgames.com/maps/terrain-from-noise/) - Comprehensive, recurrently-updated noise fundamentals article presenting primarily-modern\* information.
- [Stefan Gustavson: Simplex Noise Demystified](https://weber.itn.liu.se/~stegu/simplexnoise/simplexnoise.pdf)
- [Ken Perlin: Noise Hardware](https://www.csee.umbc.edu/~olano/s2002c36/ch02.pdf) - Original Simplex noise paper.

<div markdown="1" style="font-size: 0.85em">

\* The [accordingly-named](https://www.redblobgames.com/maps/terrain-from-noise/#implementation) section in *Making Maps with Noise Functions* contains links to several implementations of noise. The mentioned Unity `Mathf.PerlinNoise` function can work for simple experimentation, but is difficult recommend beyond that. The linked `libnoise` library offers a rotation node, but is missing support for Simplex-type noise, teaches several terms incorrectly in its documentation, and has not been updated since 2007. The article itself is a great introduction to the application of noise to terrain generation.

</div>

---

###### Proofreading Credit (original article)
- YUNGNICKYOUNG
- Amit Patel (Red Blob Games)

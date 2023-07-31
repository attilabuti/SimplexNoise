# SimplexNoise

## Usage

```javascript
// simplex_noise_2d.js
seed(1337);
let value = fbm(x, y, octaves, amplitude, frequency, persistence, lacunarity);

// simplex_noise_2d.min.js
s(1337);
let value = b(x, y, octaves, amplitude, frequency, persistence, lacunarity);
```

## Simplex noise

[Simplex noise](https://en.wikipedia.org/wiki/Simplex_noise) is the result of an $n$-dimensional noise function comparable to [Perlin noise](https://en.wikipedia.org/wiki/Perlin_noise) ("classic" noise) but with fewer directional artifacts and, in higher dimensions, a lower computational overhead. [Ken Perlin](https://en.wikipedia.org/wiki/Ken_Perlin) designed the algorithm in 2001 to address the limitations of his classic noise function, especially in higher dimensions.

The advantages of simplex noise over Perlin noise:

- Simplex noise has lower computational complexity and requires fewer multiplications.
- Simplex noise scales to higher dimensions (4D, 5D) with much less computational cost: the complexity is $O(n^2)$ for $n$ dimensions instead of the $O(n2^n)$ of classic noise.
- Simplex noise has no noticeable directional artifacts (is visually [isotropic](https://en.wikipedia.org/wiki/Isotropy)), though noise generated for different dimensions is visually distinct (e.g. 2D noise has a different look than 2D slices of 3D noise, and it looks increasingly worse for higher dimensions).
- Simplex noise has a well-defined and continuous gradient (almost) everywhere that can be computed quite cheaply.
- Simplex noise is easy to implement in hardware.

Whereas classical noise interpolates between the [gradients](https://en.wikipedia.org/wiki/Gradient) at the surrounding hypergrid end points (i.e., northeast, northwest, southeast and southwest in 2D), simplex noise divides the space into [simplices](https://en.wikipedia.org/wiki/Simplex) (i.e., $n$-dimensional triangles). This reduces the number of data points. While a hypercube in $n$ dimensions has $2^n$ corners, a simplex in $n$ dimensions has only $n+1$ corners. The triangles are [equilateral](https://en.wikipedia.org/wiki/Equilateral_triangle) in 2D, but in higher dimensions the simplices are only approximately regular. For example, the tiling in the 3D case of the function is an orientation of the [tetragonal disphenoid honeycomb](https://en.wikipedia.org/wiki/Tetragonal_disphenoid_honeycomb).

Simplex noise is useful for computer graphics applications, where noise is usually computed over 2, 3, 4, or possibly 5 dimensions. For higher dimensions, n-spheres around n-simplex corners are not densely enough packed, reducing the support of the function and making it zero in large portions of space.

## About Perlin's "Simplex" Noise

- Perlin's "Classic" Noise (1984) is an algorithm producing pseudo-random fluctuations simulating natural looking variations, producing paterns all of the same size. It is a kind of gradiant-noise algorithm, invented by Ken Perlin while working on visual special effects for the Tron movie (1982). It works by interpolating pseudo-random gradiants defined in a multi-dimensionnal grid. [Ken Perlin original references](http://mrl.nyu.edu/~perlin/doc/oscar.html)
- Perlin's "Improved" Noise (2002) switches to a new interpolation fonction with a 2nd derivative zero at t=0 and t=1 to remove artifacts on integer values, and switches to using predefined gradients of unit lenght to the middle of each edges. [Ken Perlin original references](http://mrl.nyu.edu/~perlin/paper445.pdf)
- Perlin's "Simplex" Noise (2001) rather than placing each input point into a cubic grid, based on the integer parts of its (x,y,z) coordinate values, placed them onto a simplicial grid (think triangles instead of squares, pyramids instead of cubes...) [Ken Perlin original references](http://www.csee.umbc.edu/~olano/s2002c36/ch02.pdf)

## Fractional Brownian Motion

[Fractional Brownian Motion](https://en.wikipedia.org/wiki/Fractional_Brownian_motion) is the summation of successive octaves of noise, each with higher frequency and lower amplitude. It doesn't especially matter what kind of noise, most will do. Fractional Brownian Motion (often abbreviated as fBm) is often confused with noise algorithms like [Value Noise](https://en.wikipedia.org/wiki/Value_noise) and Perlin Noise, when in fact it just takes a type of noise and makes a more interesting picture.

```javascript
function fBm(x, y, octaves, amplitude, frequency, persistence, lacunarity) {
    let total = 0;

    for (let i = 0; i < octaves; i++) {
        total += noise(x * frequency, y * frequency) * amplitude;
        frequency *= lacunarity;
        amplitude *= persistence;
    }

    return total;
}
```

There are five terms here: octaves, amplitude, frequency, lacunarity, and persistence.

**Octaves** are how many layers you are putting together. If you start with big features, the number of octaves determines how detailed the map will look.

The **amplitude** is how tall the features should be. Frequency determines the width of features, amplitude determines the height. Each octave the amplitude shrinks, meaning small features are also short. This doesn't have to be the case, but for this case it makes pleasing maps.

The **frequency** of a layer is how many points fit into the space you've created. So for the mountain scale, you only need a few points, but at the rock scale you may need hundreds of points. In the code above, I start with a small frequency (which equates to large features) and move to higher frequencies which produce smaller details.

**Persistence**, also called **gain**, is what makes the amplitude shrink (or not shrink). Each octave the amplitude is multiplied by the gain. I use a gain of 0.65. If it is higher then the amplitude will barely shrink, and maps get crazy. Too low and the details become miniscule, and the map looks washed out. However, most use 1/lacunarity. Since the standard for lacunarity is 2.0, the standard for the gain is 0.5. Noise that has a gain of 0.5 and a lacunarity of 2.0 is referred to as 1/f noise, and is the industry standard.

**Lacunarity** is what makes the frequency grow. Each octave the frequency is multiplied by the lacunarity. I use a lacunarity of 2.0, however values of 1.8715 or 2.1042 can help to reduce artifacts in some algorithms. A lacunarity of 2.0 means that the frequency doubles each octave, so if the first octave had 3 points the second would have 6, then 12, then 24, etc. This is used almost exclusively, partly because octaves in music double in frequency. Other values are perfectly acceptable, but the results will vary.

## SplitMix32

SplitMix32 is a transformation of the `fmix32` finalizer from MurmurHash3 into a PRNG. It has a 32-bit internal state, like Xorshift and Mulberry32.

```js
function splitmix32(a) {
    return function() {
      a |= 0; a = a + 0x9e3779b9 | 0;
      var t = a ^ a >>> 16; t = Math.imul(t, 0x21f0aaad);
          t = t ^ t >>> 15; t = Math.imul(t, 0x735a2d97);
      return ((t = t ^ t >>> 15) >>> 0) / 4294967296;
    }
}
```

This is based on an algorithm known as `SplitMix` included in Java JDK8. It uses 64-bit arithmetic and doesn't define a 32-bit version. However, It is derived from the `fmix64` finalizer used in MurmurHash3 and appears to be an application of Weyl sequences. MurmurHash3 also contains a 32-bit equivalent of this function, `fmix32`. The constant `0x9e3779b` is the 32-bit truncation of the golden ratio, which is also what is used in the original.

## References

- **Simplex Noise**
    - [Wikipedia, Simplex Noise](https://en.wikipedia.org/wiki/Simplex_noise)
    - [Wikipedia, Perlin noise](https://en.wikipedia.org/wiki/Perlin_noise)
    - [Patent, Standard for perlin noise](https://www.google.com/patents/US6867776)
    - [The Perlin noise math FAQ](https://web.archive.org/web/20120107051039/http://webstaff.itn.liu.se/~stegu/TNM022-2005/perlinnoiselinks/perlin-noise-math-faq.html)
    - [Perlin Noise](https://web.archive.org/web/20160530124230/http://freespace.virgin.net/hugo.elias/models/m_perlin.htm)
    - [Ken Perlin's "Noise Hardware" Reference Implementation](http://www.csee.umbc.edu/~olano/s2002c36/ch02.pdf)
    - [Improving Noise, Ken Perlin](https://mrl.cs.nyu.edu/~perlin/paper445.pdf)
    - [Improved Noise reference implementation](https://mrl.cs.nyu.edu/~perlin/noise/)
    - [Stefan Gustavson's Simplex noise demystified](https://web.archive.org/web/20160315144303/http://staffwww.itn.liu.se/~stegu/simplexnoise/simplexnoise.pdf)
    - [Improved Simplex noise demystified Java Implementation](https://web.archive.org/web/20180711022502/http://webstaff.itn.liu.se/~stegu/simplexnoise/SimplexNoise.java)
    - [The Book of Shaders, Noise](https://thebookofshaders.com/11/)
    - [SimplexNoise](https://github.com/SRombauts/SimplexNoise)
- **Fractional Brownian Motion**
    - [Wikipedia, Fractional Brownian Motion](https://en.wikipedia.org/wiki/Fractional_Brownian_motion)
    - [The Book of Shaders, Fractal Brownian Motion](https://thebookofshaders.com/13/)
    - [Fractional Brownian Motion, Inigo Quilez](https://iquilezles.org/articles/fbm/)
    - [FBM](https://code.google.com/archive/p/fractalterraingeneration/wikis/Fractional_Brownian_Motion.wiki)
- **SplitMix32**
    - [Wikipedia, Pseudorandom number generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)
    - [Pseudorandom number generators](https://github.com/bryc/code/blob/master/jshash/PRNGs.md#splitmix32)
    - [Fast Splittable Pseudorandom Number Generators](https://gee.cs.oswego.edu/dl/papers/oopsla14.pdf)
    - [SplitMix32](https://github.com/joelkp/ranoise#other-interesting-functions)
    - [splitmix32.c](https://github.com/umireon/my-random-stuff/blob/master/xorshift/splitmix32.c)
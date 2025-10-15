---
dg-publish: true
---
To calculate whether the boundary is within the SXI field of view, we need the `x` and `z` position of the spacecraft (denoted `xsc` and `zsc` respectively), the aimpoint along the `x` dimension (`xa`), and the magnetopause subsolar point along the `x` axis (`xmp`).

We will construct a set of triangles:
![[withinFOV.jpg]]

Earth is in the bottom left, and sunwards (rightwards) emanates the sun-earth line.
SXI is the blue dot at the top of the figure.
SXI's central aimpoint is the red dot. It's position along the sun-earth line is known (this dimension is `x`).
The yellow curve is our boundary. We want to see whether this resides within our field of view. It's position along the sun-earth line is also known, and `y = 0` and `z = 0`.

# Process
We begin by constructing a right-angled triangle and using Pythagorean theorem we can find out `c`. Then using the three trigonometric ratios, we can find `x1'` with `atan(x1 / z)`. Then `180 - 90 - x1'` gives us `z'` . This enables us to find `b'` with `b' = 180 - z'`. We know the angle of `a'` because it is half the FOV width of the SXI instrument. Since we know that all angles of a triangle add up to 180 degrees, we can find `c'` with `180 - a' - b'`. Finally, by using the Law of Sines, we can state the following:
$$
\begin{equation*}
\frac{a}{sin(a')} = \frac{c}{sin(c')}
\end{equation*}
$$
Rearranging this, we can get
$$
\begin{equation*}
a = sin(a') \times \frac{c}{sin(c')}
\end{equation*}
$$
to find length `a`. 

# Usage
For our purposes, we can then add length `a` to `xa` to get the final visible `x` location within the FOV. Once we've found that extremity, we then ask if the `xmp` is below that value. If it is, we know that the boundary is visible. This process is used in the final part of my [[A tool to help fitting for Magnetopause standoff distance#Measuring Success|fit helper tool]].
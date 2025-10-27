I'm trying to find out which parameters affect the accuracy of the fit. I would think orbit position has a lot to do with it, since we're likely to get better results when we're high up above the Earth looking downwards - let's have a look:

![[orbit_pos_within_FOV2 1.png]]

**The darker the dot, the better the fit**. Does orbit position affect the fit? It's hard to tell, I'll need some other way to visualise it. After some poking around, it became obvious to me that we should create a correlation matrix, specifically with the residuals and whether or not the subsolar point is within FOV. 

I need to flesh out my plan a little before I can write any code, so I'm writing here. Since residual and within/outside FOV are what we'd like to measure against, we'll include these in the assessment, but what else would we like to see?  A few of the more influential CMEM parameters would be good, along with the orbit positions. However, assessing the orbit positions individually may not be entirely useful, I'll calculate a "distance from Earth" value to include in the correlation calculation as well. 

It looks like `pandas` has a `.corr()` function ready to go. I'll mostly be following [this guide from W3Schools](https://www.w3schools.com/datascience/ds_stat_correlation_matrix.asp). 

|          |     x |     y |     z |   distance |    p0 |    p1 |   truth |   residual |   group |
|:---------|------:|------:|------:|-----------:|------:|------:|--------:|-----------:|--------:|
| x        |  1    |  0.07 | -0.25 |      -0.15 |  0.15 |  0.15 |   -0.07 |      -0.09 |   -0.25 |
| y        |  0.07 |  1    | -0.23 |      -0.16 | -0.03 |  0.05 |   -0.06 |       0.07 |   -0.07 |
| z        | -0.25 | -0.23 |  1    |       0.96 |  0.13 | -0.05 |    0.11 |      -0.21 |    0.28 |
| distance | -0.15 | -0.16 |  0.96 |       1    |  0.16 | -0.09 |    0.1  |      -0.19 |    0.31 |
| p0       |  0.15 | -0.03 |  0.13 |       0.16 |  1    |  0.06 |    0.63 |       0.15 |   -0.45 |
| p1       |  0.15 |  0.05 | -0.05 |      -0.09 |  0.06 |  1    |   -0.03 |      -0.15 |   -0.16 |
| truth    | -0.07 | -0.06 |  0.11 |       0.1  |  0.63 | -0.03 |    1    |       0.27 |   -0.59 |
| residual | -0.09 |  0.07 | -0.21 |      -0.19 |  0.15 | -0.15 |    0.27 |       1    |   -0.19 |
| group    | -0.25 | -0.07 |  0.28 |       0.31 | -0.45 | -0.16 |   -0.59 |      -0.19 |    1    |
We're interested mostly in the last two columns titled "residual" and "group". The residual array is the absolute value of the residual, so think of it not as how far above or below the truth value the fit it, but more of a total distance, how wrong. Bigger number, bigger wrong. A result is in Group 0 if it is outside the FOV, in Group 1 if it's within the FOV.

Let's look at some key takeaways:
- **z v distance**: a good sanity check, it shows most of our distance comes from changes in z (0.96), which is correct.
- **p0 v group**: It seems a higher p0 is likely to put the subsolar point outside the FOV.
- **distance v residual**: A larger distance between the spacecraft and the Earth has a slight correlation (0.19) with better fits.
- **distance v group**: A larger distance between the spacecraft and the Earth is more likely (0.31) to put the subsolar point within FOV.

This matrix is also great for looking at biases in our data. For example, we see that the `x` value has some slight correlation with `p0` which we really don't want. This value should lower with more simulations.

There are not many good correlations here which is unfortunate. I'd love to see these results on the full 1500 dataset, the above is just 100 results. These results should be ready shortly.

---
dg-publish: true
---
The popular (and possibly ill-fitting) methodology is to use the Mean Squared Error value to guide the algorithm along gradient descent. This returns values that are in whatever units the comparison subjects are in. This is usually fine and works acceptably for most applications, but I had an idea to change this.

The new methodology would be to normalise each comparison subject so that each value in the comparison images would be between 0 and 1. This means that, when comparing images, the error function would be more concerned with the location of peaks in the image, and would try to match them up. I thought this would work best because we are not trying to fit for the solar wind properties - we are trying to fit the MP standoff distance, something that is only concerned with the location of the peaks in the image.

Anyway, after 375 different fits, the scattergraph looked like this:
![[scatter_NM_375_norm.png]]
This is extremely unconvincing, and had a very low correlation value, something like -0.3. But, if we compare this with MSE used in the [[Nelder-Mead SXI Fitting]] page, we can see the results are far less dependant on the error surface. I would like to dive into a comparison here if I have the time.
## Cause
Perhaps the absolute value of the ambient background guides the solution to the right ballpark of results. Maybe removing that context confuses the gradient descent of Nelder-Mead. This makes sense, as the Differential Evolution method has no access to ambient values due to the [[Differential Evolution fitting|custom error function]], yet still performs reliably. It would be interesting using this technique with an algorithm that relies on differentiation like BFGS but ATOW I can't seem to get that working. Or using DE to find a global solution and letting NM find the rest of the parameters via a normalised error function? Honestly I think that last idea is the most solid. Time may tell.

> [!info] This page was updated due to [[Residual Miscalcuation|residual miscalculations]]. Previous results were scattered, but not meaningfully different.

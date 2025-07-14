![[scatter_DE_375_fixed.png]]

Here are 375 different fits using Differential Evolution to find the magnetopause standoff distance, created using my [[A tool to help fitting for Magnetopause standoff distance||fitting tool]]. Overall I think these results are very good. We can't be expected to create good fits when the MP subsolar point is outside the boundary, but we have to be doing decently well when they are. However, you can see it's even good enough to get a handful of outside FOVs correct.

It likes to slightly underestimate the fit at further MP distances. Until I can work out why, I fear that all our fitting algorithms will do the same. This could be mitigated with a scalar value to compensate, maybe the added offset increases at further distances (i.e. MP distance is always multiplied by 1.05).

The Differential Evolution algorithm here uses my [[My custom Differential Evolution error function||custom error function]].

I would like to try using this as a starting point, and then passing the two calculated parameters over to something like BFGS or Nelder-Mead to calculate the rest of the values. If the [[Residual Miscalcuation|residual miscalculations]] are any indication, it seems that the other values can make a big difference, so hopefully we could tune that process for accuracy.

> [!info] This page was updated due to [[Residual Miscalcuation|residual miscalculations]]. Previous results were far more scattered, but not meaningfully different.
---
dg-publish: true
---
![[scatter_NM_375_fixed.png]]
Above is a run I did with 375 fake observations using the [[A tool to help fitting for Magnetopause standoff distance||MP fitting tool]]. One can see that these lines that surface during the Nelder-Mead fit are very present. 
## Cause
I would say that this is the error surface being graphed, highlighting the local minimums that arise with gradient descent. That is until I consider that each of these is a new observation with a different set of parameters, therefore surely a new set of false minima.

The bit that gets me is that, when I created this library of observations, I set up the important parameters of CMEM to increase in a series of steps. For example, the solar wind velocity could be -700, -533, -366, or -200. The velocity can be no other value than one of these. These groupings we see on the scattergraph could be due to the input parameters being grouped in this way.

One would expect to see these same groupings in the DE solver as a result, but as you can see from the [[Differential Evolution fitting|DE run]], this is not the case. Another point against this idea is that I can make out 3 rows, with outliers. Here are the number of possible values for each parameter:
- Velocity: 4
- Magnetic direction: 5
- p0: 5
- bs: 3

The only one these results would line up with would be the bowshock standoff, which I am sure would not have this drastic of an effect on how the error function finds the peaks. 

I don't know where these lines are coming from.

> [!info] This page was updated due to [[Residual Miscalcuation|residual miscalculations]]. Previous results were far more scattered and had around 7 lines showing diagonally (a result of plotting, not present in the data), but would not meaningfully change the conclusion drawn.
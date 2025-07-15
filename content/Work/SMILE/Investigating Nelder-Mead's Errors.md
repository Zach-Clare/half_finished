![[scatter_NM__375_redisual_time_error.png]]

Every dot on the graph above is a single result from the [[A tool to help fitting for Magnetopause standoff distance||fitting tool]]. Any result within 0.5 on the Residual axis is classed as a successful fit. One can see the majority of successful fits taking around 6 seconds, though some have taken as long at 14 seconds to fit. Many other fits that have taken 14 seconds have also been very wrong.

This leads us to the conclusion that:

>[!knowledge] We cannot use time taken as a success indicator (at least with Nelder-Mead).

You'll see some results with a low residual have a non-negligible error. Many successful fits have far more error than unsuccessful fits. This means that a low error value does not conflate with a  successful fit. This is a problem. The error function is rewarding unsuccessful fits. How do we fix this? [[Nelder-mead's normalised error function]] gives poor results. Perhaps there's another way. 

> [!info] This page was updated due to [[Residual Miscalcuation|residual miscalculations]]. Previous results were far more scattered, but not meaningfully different.
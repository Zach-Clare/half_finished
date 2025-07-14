This describes how strongly two sets of values are related. If the correlation is between 0 and 1, it is known as a "positive correlation" and means that, as set X increases, so does Y. If the correlation is between -1 and 0, that is described as a "negative correlation", meaning that as set X increases, Y decreases, and vice vera.

I'm using it in [[A tool to help fitting for Magnetopause standoff distance]] to show how accurate a particular machine learning method is by calculating correlation between truth data and fitted data.

Considered a [[Summary Statistics||summary statistic]].
# Usage
In Python, many people use `NumPy`, but I find it easier to use the inbuilt `statistics` library. 

Here's an example:
```
from statistics import correlation

x = [0.10, 0.20, 0.30, 0.40, 0.50]
y = [0.15, 0.17, 0.29, 0.45, 0.61]

corr = correlation(x, y)
```

Simple as that.
#python #machine_learning #ucl
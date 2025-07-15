This is a simple calculation. The maximum value minus the minimum value in the dataset. If the resultant value is large, there is a large spread of values throughout the dataset. If the value is small, it means the grouping is tight.

Considered a [[Summary Statistics||summary statistic]].

Notice that this doesn't take distribution into account. Outliers can heavily influence spread. The variance and standard deviation can also describe a dataset as a summary statistic.

In python, you can get this statistic with:
```
maximum = max(dataset)
minimum = min(dataset)

range = maximum - minimum

```
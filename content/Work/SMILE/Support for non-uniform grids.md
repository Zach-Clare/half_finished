I'm using this note to understand how best to support non-uniform datacubes. The way I see it, we have two options. We could either pre-process a cube to "cast" it to a uniform cube, or we can use some logic in our sampling method to translate the requests in real-time.

## Sampling Logic
We'd need to adjust some of our datacube loading logic to account for this but that'd be fine. It might get a bit slow because we'd have to translate every sample, resulting in thousands more calculations. 

The actual method I envision would be for any given coordinate, find the index value of the best-matching coordinate in the loaded coordinate array and return this. It's slower, but how much slower? Well, that depends on how many places away the value is. Since it's an ordered array, we can use a binary search style algorithm to greatly improve speed. Once we have the infrastructure, we can try a few different algorithms to see which is quickest.

## Pre-processing
This would be a one-time step when loading the datacube. You'd choose your smallest step size and interpolate a whole new grid. I think I prefer this one as it's likely to be quicker. But there are some restrictions we'd need to place on this. What's the smallest allowed cell? If it's too small, our grid will be huge and we may have to worry about memory, especially because this program may be running on multi-core.
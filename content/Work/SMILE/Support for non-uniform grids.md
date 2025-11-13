I'm using this note to understand how best to support non-uniform datacubes. The way I see it, we have two options. We could either pre-process a cube to "cast" it to a uniform cube, or we can use some logic in our sampling method to translate the requests in real-time.

## Sampling Logic
We'd need to adjust some of our datacube loading logic to account for this but that'd be fine. It might get a bit slow because we'd have to translate every sample, resulting in thousands more calculations. 

The actual method I envision would be for any given coordinate, find the index value of the best-matching coordinate in the loaded coordinate array and return this. It's slower, but how much slower? Well, that depends on how many places away the value is. Since it's an ordered array, we can use a binary search style algorithm to greatly improve speed. Once we have the infrastructure, we can try a few different algorithms to see which is quickest.

## Pre-processing
This would be a one-time step when loading the datacube. You'd choose your smallest step size and interpolate a whole new grid. I think I prefer this one as it's likely to be quicker. But there are some restrictions we'd need to place on this. What's the smallest allowed cell? If it's too small, our grid will be huge and we may have to worry about memory, especially because this program may be running on multi-core.

Okay let's discuss implementation. I'm currently writing this at midnight, I'm in China and my body feels like it's only 4pm. So our logic will be when we load the cube, let's take a look at that. Okay our first hurdle is when we calculate our gaps with `InitSpacing()` (all methods belong to `DataCube` unless otherwise stated). We need to process this spacing differently, depending on on if they're uniform, so I think we have to turn this into some kind of loop, or calculate each gap and, if it's different, save the gap. And if not, record it each time. Due to CPU tramlining, this shouldn't affect the performance so much because the CPU can guess what's going to happen most of the time, apart from when the grid is non-uniform (and again unless the grid is consistently inconsistent). Here's what I cam up with, but I wrote it in the comments to I'll copy it here:
```c++
// okay, here we want to handle our non-uniform axis.
// let's find the smallest, that's our new grid size!

auto min = std::min_element(distances.begin(), distances.end());

// this min value is new grid size. Maybe we need to break out this whole if statement because we can't return a float here.
// we need to iterate through our coords again and see how many of these minimums fit inside each one
// Then we add how many multiples to a new array e.g. [3, 3, 2, 2, 1, 1, 1, 1, 2, 2, 3, 3]
// And when we load our data from the file, we use this array to control how many times we add it.
// The obvious question here is what happens if it's not exact?
// I don't know.
```

A lot to think about. This is a more complex task that I thought, and it deserves a sharper mind than what I'm able to give it right now.
At this point in the project, most of the [[Differential Evolution fitting||DE results I show]] are done with a custom error function. It's a bit more complex than something like MSE, but it's not difficult to understand. 

Firstly, a pre-processing step. Each input image is thresholded in respect to a percentage, usually something like 70 or 80. This means that every value in the image is turned into a 1 or 0, depending on whether they're in the bottom 80% or the top 20% of values (or whatever your threshold value is). 

This results in images like below:
![[binarisation_example.png]]

The next step is to run the numerical comparison. An easy approach would be to count every correct pixel as a 0 and every incorrect pixel as a 1. The actual implementation is slightly different. Let's say image A is our truth image (is there a better term for this?) and image B is our image to compare that's just been generated via our simulation or via whatever. 

Counting pixel by pixel, if A's pixel is a 1 (that is to say high, within the upper threshold) and Bs's pixel is 0, add 1 to the cost value.

Conversely, if A's pixel is 0 and B's pixel is 1, add 0.5 to the cost. 

This has the effect of if B's thresholded area is too large, there is less punishment than if it is in the wrong place.
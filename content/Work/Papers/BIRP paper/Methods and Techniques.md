*Overall explanation or referencing of computation techniques*
*That means input file ingestion, CMEM algorithms, camera orientation, ray marching, and file output*

*I suppose we just use the logical program flow as a framework to explain the techniques we used, so it would look like:
- *Datacube setup*
	- *File ingestion*
	- *Array setup (slices)*
- *Sightline generation*
	- *Camera instantiation*
	- *Local pixel coordinates*
	- *Local coords -> world space*
	- *Ray marching algorithm*
- *Image creation*
	- *Sampling*
	- *Trilinear*

## Datacube Setup
The datacube is iterated through, line by line, to be stored in a 3 dimensional array. The elements at the highest level of the 3 dimensional array hold 2 dimensional slices in the XY plane, where the values in each nested array have a common Z value. One may visualise this as a stacking of square plates. The second level of nesting is a 1 dimensional array holding X values, where the values in each array have a common Y value.
### Sampling
The BATSRUS file contains an index of tick values that specify the scale and position of each emission value. Assuming a uniform grid pattern, the program uses this to store the emission values without labels. When the value of a particular coordinate is asked of the datacube, it will calculate how far that requested coordinate is from the beginning of the whole datacube. This value, when divided by the uniform spacing between values taken from the tick value array, can be used as an index when referencing the emissions array.

In nearly all cases, the requested sample does not perfectly align with the coordinate system of the provided emissions datacube. In the interests of speed, the default behaviour is to return a sample using a simple Nearest Neighbour interpolation. The user can request a trilinear sampling method via a command line flag, meaning the sample is calculated as a distance-weighted average of its 8 closest points. This provides smoother and more natural-looking values but increases computation time. 
## Sightline Generation
*Constituting:
- Camera *
*[Do I reference Scratchapixel's tutorial?](https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-generating-camera-rays/generating-camera-rays.html)*
*I could reference these papers:*
- *https://www.scilit.com/publications/c7f8e414a53b377c2c8497ec3e698da5*
- *https://dl.acm.org/doi/10.1145/74334.74359 (references the above for the ray marching algorithm. I think it also has the method of image plane pixel generation. I don't think it includes the translation from image space to world space, the bit with the rotation matrices.)*

The program accepts camera properties through the command line. Field of view ($FOV$) and pixel width in world degrees ($s$) are used to define the camera's image plane. Through the following, dimensions of the final image ($\|x\|$ and $\|y\|$) can be calculated:

$$
\begin{equation*}
\|x\| = \frac{FOV_{height}}{s}
\end{equation*}
$$
$$
\begin{equation*}
\|y\| = \frac{FOV_{width}}{s}
\end{equation*}
$$

>[!question] Do I need to discuss this whole thing? Can I not just point them to a paper or the tutorial I used?

These values are used to construct a 2 dimensional array with shape ($x$, $y$). Consider this X-Y plane as a new 3 dimensional coordinate system, with the Z axis pointing towards the viewer of the image. After normalising the X and Y pixel coordinates between -1 and 1 and placing this plane -1 unit in the Z axis away from the viewer, the program will generate a ray from the camera's origin through the centre of each pixel ([Do I reference Scratchapixel's tutorial?](https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-generating-camera-rays/generating-camera-rays.html)). This vector is then turned into a unit vector and a perspective transformation based on the concept introduced by Newman & Sproull (1979) (https://archive.org/details/principlesofinter00newm) is then performed on each camera ray. Now that the camera ray is in the context of the world space, it can be used to sample from the data in the same manner shown by Perlin & Hoffert (1989). By sampling along this vector at some interval, an integrated line-of-sight value can be calculated. Further, by doing this with the unit vector of each pixel and storing the final sum in the image array, the final image can be built.

 Given the aimpoint and position of the camera, a ray is generated from the camera's origin point through the centre of each pixel in the form of a unit vector ([Do I reference Scratchapixel's tutorial?](https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-generating-camera-rays/generating-camera-rays.html)). %% For  every pixel, we normalise it's position in the array between -1 and 1, effectively creating a new coordinate system for our image where the centre pixel has the coordinates (0, 0).  %% 

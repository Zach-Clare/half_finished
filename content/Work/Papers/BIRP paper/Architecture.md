*What are the different parts of the program?*
*How do they interact? Parents?*

The `main` function handles arguments passed via command line and will then instantiate classes to achieve the desired result. `Camera` and `Space` are the two primary classes in the program. The `Camera` class can be thought of as an abstraction of the SMILE spacecraft, having a position and pointing direction, with the ability to take images through a concrete child of the `Space` class. The `Space` class can be thought of as the physical space that this simulation runs in. The `Space` class itself is abstract and relies on two concrete child classes for implementations; `DataCube` and `CMEM`. The interaction between `Camera` and one of `Space`'s concrete children constitute the core functionality of the program. `Camera::GetSample()` will provide x, y, and z coordinates to a concrete child of `Space` so that it can return the x-ray emissivity for that point. This interaction, when combined with `Camera`'s ability to generate emanating and directed sightlines to then take  samples along them creates an image of the world space from the camera's perspective. All other parts of the program facilitate this core process by preparing data required for this integration operation or by dealing with necessary inputs and outputs.

.....

## Input
The data volume to render through is defined through the command line interface to the software. The user may specify an input file containing a 3 dimensional grid or a set of parameters for CMEM initialisation.

If using an input file, it must be a uniformly-distanced grid following the format of the BATS-R-US simulation outputs. Line 10 of the file contains three space-separated numbers that define the size of the grid (X, Y, and Z respectively), which the following three portions of the file describe using every point on this grid as a floating point number. From the grid specification, the program records the starting point and grid spacing, which can be used to calculate an index from an X, Y, or Z coordinate. The final portion of the input file describes the data, where the data is loaded into the software as described in \ref{DataCube Setup}.

If CMEM is used, the initialisation parameters can be passed through the command line. Where any parameter is missing, the software will substitute in a default parameter. For this reason, unless the user is merely testing their installation or is otherwise unconcerned with science, it is highly recommended to pass in all values for CMEM to simulate the intended solar wind composition accurately. A full list of input parameters can be found in ??? APPENDIX. 

## Core Computation
The program follows an object-oriented pattern by using classes to simulate the various components of the problem. The `Camera` class can be thought of as an abstraction of the SMILE spacecraft. It has a position and pointing direction in space and has the ability to take renderings through the `Space` class, which can be thought of as the physical space the simulation runs in. While the `Space` class is abstract and cannot be instantiated directly, `DataCube`and `CMEM` are both concrete child classes and implement the necessary functions for the core data exchange that facilitates the main use case of this software. Namely, the interaction of `Camera`'s rendering method and `Space`'s sampling method. 

`Camera`'s core rendering method, named `Integrate`, will iterate through each image pixel and perform the necessary rotation to put the pixel unit vector into world space. This step happens at the rendering stage and not as a separate pre-computation step beforehand so that `Camera`'s aimpoint or position cannot be changed midway through the rendering process. Once the rotation have been calculated, the vector will be travelled along in steps defined earlier in the program flow. `Space`'s emissivity will be sampled at each of these points until it becomes out of bounds, as defined by the implementation of `Space`. When this happens, the sum is placed into the image array and the next image ray is processed. All other parts of the program exist to facilitate this core exchange, whether through pre-requisite data initialisation or by dealing with inputs and outputs.

By having `DataCube` and `CMEM` as child classes, it allows an easy framework for extension. Should there be a need to integrate through a new solar wind composition simulator, it can be dropped in place with no change to or knowledge of other parts of the program, as long as it abides by the expected interface set out by `Space`. This idea of modularisation is a core principle of object-oriented design and should be respected by further extensions to this software.
## Output
The final image can be outputted to a FITS file or a simple text file. The FITS file output is done with Euclid libraries and executables for FITS (EleFits). The resultant FITS file has a number of headers attached that describe the aim and position of the Camera (??? monospace) class at the time of the render and the field-of-view characteristics used. This FITS file can be opened with any available FITS file viewer, and so the visualisation of output data fell outside the scope of this software. The text-only output is intentionally basic and will output a comma-separated list of output values, with rows in the image separated with a newline character. This allows for easy transport of the data elsewhere. This file is not created by default, so if it's wanted, it must be manually added and the program must be rebuilt using the provided script.


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

From the calculated image dimensions, we construct a 2D array with shape ($x$, $y$). For every pixel, we normalise it's position in the array between -1 and 1, effectively creating a new coordinate system for our image where the centre pixel has the coordinates (0, 0).
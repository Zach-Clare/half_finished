*Presented a 3D to 2D image renderer for the SMILE mission*
*Capable of rendering through 3D data grids or through CMEM, an empirical model*
*The software is extensible and outputs data to the FITS format*
*The code is open source and available through ?*

This paper presented an image renderer capable of creating line-of-sight integrations in near real-time through 3 dimensional data cubes or through the empirical emissivity model, CMEM. The software was created to solve a specific problem related to the Level 4 data product pipeline of the SMILE mission. Its computation speed and its complete operation through the command line interface makes it well suited to be used as part of a machine learning pipeline. 

The internal architecture was described in this paper, covering the program's ingestion of input data as CMEM parameters or a 3 dimensional grid, execution of its core computation to turn the 3 dimensional data to a 2 dimensional integration image, and subsequent output to a transportable FITS format with appropriate headers. The methods used to achieve file parsing, sampling, and the generation of pixel rays were described. Performance was assessed and plotted, and further use cases were discussed.

......

This paper introduces and discusses a new image renderer capable of taking line-of-sight integrations through science data in under 50 milliseconds. This computation speed places it as a useful tool for SMILE's Level 4 data product pipeline to derive the magnetopause subsolar distance from Level 3 data.

The internal architecture of the software was discussed, showing how the software organises input, processes it, and outputs it to the user. The extensible nature of the software was discussed, with specific areas for extensibility pointed out. The methods of the software were then discussed, including how the program stores the input data, how the data is sampled, and how the pixel rays are generated according to input parameters. Performance is shown to degrade with resolution as expected, and two figures are offered to visualise this scaling. Finally, the use cases for the software are briefly discussed, explaining its applicability for the SMILE Level 4 data pipeline and to aid in understanding MHD output — applicable in both academia and engineering.


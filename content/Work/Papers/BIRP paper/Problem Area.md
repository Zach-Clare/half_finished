*Discussion of science goals*
*What problem does this software solve?*
*Discussion of why we need very fast renders of this?*
*Why is it useful for people to input their datacube?*
*Is it obvious what a datacube is?*

This software has arisen from the SMILE project (???) which, among other aims, makes an attempt to find the location of the magnetopause within 0.5Re. This Level 4 product will be created in real-time (in under 5 minutes) from a machine learning process that takes an image attained from the soft x-ray imager (SXI) aboard the spacecraft. The process inputs various parameters into a mathematical model and then renders an image through it, hoping to reconstruct the SXI image as closely as possible. Once those inputs are found, the subsolar magnetopause point can be trivially calculated (Lin or Shue???).

Previously, Read and Sembay (???) created an image renderer that took a 3D array of data and outputted a 2D line-of-sight integration. An image renderer is a relatively computationally expensive objective function in a machine learning pipeline. The software described in this paper increased the speed of the renderer by a factor of 10, meaning that the machine learning pipeline can perform far better in the 5 minute goal.

Therefore, int he context of SMILE's Level 4 data products, will be invoked from some other program running the machine learning process and selecting new parameters to assess against the SXI's image. However, as a standalone program, individual integrations can be made from arbitrary positions in space and with custom solar wind parameters. 

Support also exists for running an integration through a 3D datacube, such as those outputted by ==Andrei's main simulation for the BATSRUS cubes==. The outputs of PPMLR simulations can also be run with minor modifications to the output. This widely opens the applicability of the software, for example for use with ==Lucie and Hamish's sun project?==. 

>In short, I'm trying to say that this all came out of a need for a fast but specialised renderer, right? With Datacube support


*This software has arisen from a need for a near real-time 3D to 2D image renderer necessary for SMILE*
*SMILE aims to __whatever__ (Smile 2016)*
*The level 3 output is **whatever** with specific cadence (5 mins)*
*The level 4 product is the actual values*
*One of the methods to get this is a machine learning method*
*Using a POV simulator to capture emission and machine learning to try new values*
*Use empirical model outputs to calculate MP subsolar point*
*But this means potentially hundreds of epochs, renderer gotta be quick*
*This paper presents a renderer with real-time performance on consumer-grade hardware*
*Single core so ML can be multicore*
*Other use cases*

The Solar wind Magnetospheric Ionospheric Link Explorer (SMILE) is a novel mission selected by the European Space Agency and the Chinese Academy of Sciences to study Earth's magnetospheric response to the solar wind. Among other mission operations, X-ray images will be obtained from the Soft X-ray Imager (SXI) instrument to determine the location of the subsolar magnetopause point within 0.5 Earth radii (Re) with a time resolution of at most 5 minutes (Branduardi-Raymont 2018). Images from the SXI instrument processed to Level 3 will be background-subtracted X-ray images with the likely position of the magnetopause boundary in the centre of the frame. Level 4 data products will be science values (e.g. magnetopause subsolar distance from Earth) derived from the Level 3 images, through a number of possible methods (Wang & Sun, 2022).

Wharton (2025) presents an empirical model that builds upon the scaled Lin et al. (2010) model to include the cusp regions and goes on to fit model parameters to a 3 dimensional emissivity cube. From this, the ideal model parameters can be derived and so the magnetopause subsolar distance can be calculated. Using the SXI Level 3 images as input to a machine learning model, the 3 dimensional structure of CMEM can be rendered to a 2 dimensional image for trivial comparison. The CMEM parameters can be tweaked until an ideal fit is found and the ideal parameters can provide the subsolar magnetopause location. Other information is required, such as the location of the spacecraft and the dipole tilt of the Earth at the time the data product was captured, but this can be easily retrieved.

This line-of-sight integration from a 3 dimensional dataset to a 2 dimensional image can be thought of as a volume rendering problem. When compared to the other necessary computations in a machine learning optimisation problem such as calculating error, the volume rendering problem is computationally expensive and a non-trivial problem to solve. This paper presents a volume renderer purpose-built for the SMILE mission and discusses the computation methods and trade-offs involved in developing this software.

................
The Solar wind Magnetospheric Ionospheric Link Explorer (SMILE) is a novel mission selected by the European Space Agency and the Chinese Academy of Sciences to study Earth's magnetospheric response to the solar wind. Among other mission operations, X-ray images will be obtained from the Soft X-ray Imager (SXI) instrument to determine the location of the subsolar magnetopause point within 0.5 Earth radii (Re !!!fix) with a time resolution of 5 minutes or under \cite{branduardi-raymont2018}. Images from the SXI instrument processed to Level 3 will be background-subtracted X-ray images with a predetermined aim point in the centre of the frame. Level 4 data products will be science values (e.g. magnetopause subsolar distance from Earth) derived from the Level 3 images, which is possible through a number of methods \citep{wangsun2022}.

\cite{wharton2025} presents an empirical model that builds upon the scaled \cite{lin2010} model to include the cusp regions, and from a set of CMEM parameters, the magnetopause subsolar distance can be calculated according to the Lin model. If such an empirical 3 dimensional model could go through a line-of-sight integration process to produce a 2 dimensional image, this image could be mathematically compared the SMILE's Level 3 output images in a machine learning pipeline to derive Level 4 science parameters about the magnetopause. This line-of-sight integration from a 3 dimensional dataset to a 2 dimensional image can be approached as a volume rendering problem, and when compared to the other necessary computations in a machine learning optimization problem — such as parabolic calculations often involved in gradient descent or the more commonplace calculations of a loss function — this volume rendering problem is comparatively computationally expensive and non-trivial to solve.

This paper presents a volume renderer purpose-built for the SMILE mission. The software's capabilities are well-suited to solving the problems that SMILE's Level 4 pipeline faces, but are also applicable in visualising magneto-hydrodynamic (MHD) model output for further use cases. The software's design, architecture, and performance are discussed and applicable use cases are outlined. The Level 4 machine learning process falls outside of the scope of this paper and will not be discussed in detail.

.......... I went with this one above^^^^^^^

 By using a machine learning algorithm to assess CMEM parameters and compare the rendered image the the Level 3 data, the magnetopause subsolar distance can be estimated, fulfilling the requirement for Level 4 data.
 ......

The Solar wind Magnetospheric Ionospheric Link Explorer (SMILE) is a novel mission selected by the European Space Agency and the Chinese Academy of Sciences to study Earth's magnetospheric response to the solar wind. Among other mission operations, X-ray images will be obtained from the Soft X-ray Imager (SXI) instrument to determine the location of the subsolar magnetopause point within 0.5 Earth radii (Re !!!fix) with a time resolution of 5 minutes or under \cite{branduardi-raymont2018}. Images from the SXI instrument processed to Level 3 will be background-subtracted X-ray images with a predetermined aim point in the centre of the frame. Level 4 data products will be science values (e.g. magnetopause subsolar distance from Earth) derived from the Level 3 images, which is possible through a number of methods \citep{wangsun2022}.

Deriving the magnetopause subsolar distance is possible with a number of methods \citep{wangsun2022}, but this paper presents a volume renderer purpose-built for visualising solar wind composition simulations to aid in the creation of such level 4 data products. Such simulations can be the output of magneto-hydrodynamic models such as BATS-R-US (??? reference?) or CMEM (Wharton, 2025).

This paper presents a volume renderer purpose-built for visualising solar wind composition simulations to aid in the creation of Level 4 data products for SMILE. Using 3 dimensional output from magneto-hydrodynamic (MHD) models or a set of parameters for CMEM (Wharton, 2025), a 2 dimensional line-of-sight integration image is calculated. This is especially useful in the context of SMILE (red book) which, among other data products, will produce background-subtracted X-ray images of the magnetopause. Through time-efficient creation of these simulated line-of-sight integration images, this software can assist in the processing of the background-subtracted images taken from the SXI instrument onboard SMILE with the aim to derive the magnetopause subsolar distance to constitute a Level 4 data product.



Refs:
- Lin 2010: https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2009JA014235
- Wharton 2025: https://figshare.le.ac.uk/articles/journal_contribution/Modeling_the_Magnetospheric_3D_X_Ray_Emission_From_SWCX_Using_a_Cusp_Magnetosheath_Emissivity_Model/28523957?file=52730147
- Branduardi-Raymont 2018: https://sci.esa.int/web/smile/-/61194-smile-definition-study-report-red-book
- Wang & Sun 2022: https://geoscienceletters.springeropen.com/articles/10.1186/s40562-022-00240-z



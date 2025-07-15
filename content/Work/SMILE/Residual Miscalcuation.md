 When calculating the [[Residuals||residuals]] for the [[A tool to help fitting for Magnetopause standoff distance||fitting tool]], I made a mistake in calculation that made the results seem far less coherent than they actually were.

When the fit is complete, the MP standoff distance is calculated for the truth image and for the fitted image. To calculate this, it needs solar wind velocity and the magnetic vector (to calculate the dynamic and magnetic pressure), dipole tilt, and all p parameters.

It's important to know that when fitting the CMEM parameters, it only fits two or three parameters in total. There are 13 (ish) parameters to CMEM, so for every parameter that isn't being fitted, a default value is used and those values all stay the same throughout the entire process. 

Unfortunately, when I was calculating the standoff distance, I was passing the two or three fitted parameters along with the pressure values from the truth image, not the default values used during the fit. Since all the parameters work in unison to describe the shape of the magnetopause, using the fitted values with different pressure values will obviously result in a different standoff distance. Once the calculation uses the default parameters during the fit, the residuals get a lot closer to what we would expect.

Below is a visual comparison. The first image is before the residual fix. The second image is with the fixed residuals, along with a fixed calculation for the inside/outside FOV markers.

![[scatter_DE_375.png|300]]![[scatter_DE_375_fixed.png|300]]

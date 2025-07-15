---
dg-publish: true
---

During April and May 2025, I created a sort of framework for developing and (crucially) measuring the effectiveness of magnetopause fitting algorithms. It underwent a significant refactor in June 2025.

There are a few components but the general idea revolves around three areas:
	- Generating ~1500 fake observations
	- Fitting CMEM parameters and write to file
	- Read file and measure fitting success over ~1500 different observations

They are each separate files, so progress is saved at each stage. This allows me to iterate on ideas without needing to run time-consuming fits again.

# Data Generation
`create_library.py` will create the observations using the given orbit files in combination with a reasonable range for the CMEM parameters. It outputs into a given folder with a FITS file of each observation point with all the input parameters used present in the filename. It also has a huge comment describing the format of the filename so that you know how to parse it once you start processing it.

# Fitting CMEM
`read_library.py` is a poorly named file that will do the actual fitting. It will also parse the filenames to derive the parameters used to generate the image and save them in a ~~CSV~~ JSON file alongside the parameters it acquired via the fitting process. This script takes the longest to run (because it's indirectly rendering many images every iteration), so I often just cap it at 100 iterations or so to get a decent idea for the fit characteristics. 

# Measuring Success
`read_results.py` looks at the created CSV and parses the results into easily accessible lists and dictionaries. From there, a few plots are made and shown (an [[Differential Evolution fitting|example can be found here]]), along with a quick [[Correlation]] calculation.

The main assessment function is in the Results class as `analyse()`. All the analysis results are stored internally to the Results class, but they're accessible from the outside. In the script, it's shown how one can reach inside to further process the values they want. This is how the plots are prepared.

# Next Steps
Once we have all of this, we can run it multiple times for different fitting algorithms to comparatively measure their accuracy. We can then take this into account, along with average fit times and parameter bias to select the best option to use on real data. 

#cmem #birp #smile #machine_learning #python #ucl 

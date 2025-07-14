---
dg-publish: true
---

This note should help provide a brief overview of how the [[A tool to help fitting for Magnetopause standoff distance|fitting tool]] works in practise. I encourage you to open the code and follow along - look in the main functions and get inquisitive.

## Creating observations
To create a set of observations, the Library class will need a group of `states` and a group of `positions`. This, fairly eloquently, is created with `get_states()` and `get_positions()` respectively. The `get_positions()` function requires a number of positions to create, so it's best to create all of your solar wind `states` first and then pass the length of those to the positions function.

Once this is done, you can call `generate()` to combine the arrays and create a CMEM rendering for each observation.

You have all these images of the magnetopause now. They're ideal images (super sparkling clean), so there's no noise on them. You could go ahead and start fitting here but it might not mean much if you're trying to simulate the actual L4 pipeline for each of these. 

Really, you should pass them through the SXI simulator, which will add noise and vignetting. Then you may want to look into some sort of restoration method. I think Rongcong's AI patch estimator works best, but you can do whatever. 

I do concede that there's a bit of a hole in this tool. Ideally, there would be the SXI simulator and Rongcong's restorer slotted in here. Alas we'll have to do without. Process the observations however you see fit, and then proceed.
## Processing observations
To start doing some fitting, you'll probably want to loop through the filenames of all the images you have just created (and maybe processed). 

The Library class has a `parse_filename()` method that returns all the parameters used to generate the image (provided the filename is not malformed). If it doesn't work, you can create a replacement function for a different type of filename or, even better, create a child class of Library and override it.

Anyway, you can then pass your filename to whichever fitting function within the `Fitting` class you'd like to use. This is the meat of the whole tool, I think. This is the science. Tweaking the error function in the `assessor` class to reflect your problem space is very important. 

>I should refactor the assessor function and clean up the error functions.

Once you have your result object, pass it to the `Results` class and it'll keep a hold of it. When you've done all the fits you'd like to do, call the `Result:writeJson()` method and it should record it in your file of choice.

## Visualising results
Instantiate the `Results` class, pass in your JSON file with `results.readJson(filename)` and then call `results.analyse()`. That will fill out `Results`' properties with a bunch of data you can use.

You can plot the residuals (which is how wrong the MP position is), you can also inspect time taken per fit, and look for relationships between time, cost function errors, and accuracy. 

I've found that this is most useful when comparing different fitting algorithms, but there's also a lot of use in finding how accurate the fitting is in various solar wind states. Perhaps it's awful when the spacecraft is in a specific position, maybe we get trash results when the density is low. I don't know, but the data is now there and it exists, all you gotta do now is roll up your sleeves and get in there.
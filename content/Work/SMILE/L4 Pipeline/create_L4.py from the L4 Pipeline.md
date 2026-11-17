This file is doing a lot of work. It bins the raw Level 3 files into L4-ready processable chunks and then creates the L4 file, along with any programmatic flags.

My first challenge is once we've calculated the initial signal to noise ratio, how do we then programmatically search "outward" from the beginning time? Let's use a step counter inside a while loop, which we can then use to re-create folder names to sample the Level 3 table. I thought about using this to scan the Level 3 path but it seems a good idea to me that discover_L3.py is the only file in charge of bringing Level 3 files into context and, after that script, we should only use files known in that context. If we find Level 3 files not being found, then we simply bump up the occurrence rate of the discovery script. It makes tweaking and problem-solving more simple.

## Plumbing into the L4 Fitting Tool
Once we adaptively (temporally) bin up the files to figure out how long of an integration we need to do, we then have to actually plumb that into the tool that takes a Level 3 file and performs the image analysis (the fitting with CMEM and the DIA methods) to create the Level 4 file. This tool expects certain headers from the Level 3 file. It also currently creates things like the total counts map from data that's inside the file.

We could create this `Image` object, which is an object that represents the data in the Level 3 file, and then look inside to update certain values, like the counts map, to reflect the binned up example.

But I think we should use a different approach. I think we should create an intermediary Level 3 file - one that's binned up and ready for direct processing. As it stands, the fitting tool doesn't deal with TOT, BKG, or VCY files. That is to say, it doesn't account for vignetting or for background subtraction at all, and assumes all of this is part of a previous step. This complicates matters. Where do we put these intermediary files, what do we call them, how do we track them, do we use a queue? There's a lot there, but let's take this slowly and discover the implications.
## Creating an intermediary Level 3 file
Looking at the process for actually creating this file in the first place, do we have what we need, and can we supply what the fitting tool requires? Let's look into the files it's currently using. These are all the extensions of the file named `SMILE_SXI_L3_SCIM15-SCI-CXF_20260317T0240-20260317T0245_V01_1x1.fits`:
![[Pasted image 20261117161318.png]]

Every extension listed above has an identical header:
![[Pasted image 20261117161432.png]]

In fact, that's not exactly true. Every extension with a name also has one final comment at the end describing why this extension is unique in plain English. For example, PBMAP has "COMMENT Predicted Particle Background Map". CTSMAP has "COMMENT Observed Total Counts Map". I think we can quite comfortably create this. We simply place the correct data into the correct headers, and place the background-subtracted de-vignetted image into the CXFOV extension. We can use Sam's code to guide us here especially. 
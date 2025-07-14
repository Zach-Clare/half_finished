---
dg-publish: true
---
This is extremely simple, actually.

When you define the model function, you can accept a `detector` object (with type `Detector` or `CMOS` or `CCD`). Inside that `detector` object, you can access the `photon` attribute, which holds an `array` attribute. 

Any changes to that array are applied to the detector object that was passed, that is to say, the detector object is _passed by reference_. 

~~I'm currently unsure whether the photon array is then correctly passed down the flowchart of models, because when I use `pyxel.display_detector(detector)`, the photon array changes are only really visible in the `photon` dropdown, when I would expect those changes in the final `image` array. Perhaps I'm misunderstanding what these arrays actually are.~~

Turns out that the default `full_well` and the `simple_adc` will whiteout the image if either is included in the `.yaml` file. This needs to be overcome somehow, but I'm not sure if this is a tuning issue or perhaps our input FITS file is just too bright. I'm sure I'll learn enough soon for the answer to become obvious.
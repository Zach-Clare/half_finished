---
dg-publish: true
---
Spoiler alert: I'm now partially bald. The simple line of `import pyxel` had me ripping my hair out for a solid few hours. After fixing the problems and washing my hands of dearest headfluff, I can write about it to save the next programmer.

## The Problem
```
ImportError: Function: 'shield' is not contained in 'pyxel.models.photon_collection' or may not be defined in its __init__.py
```
`shield` is the name of my test function to represent the light shield - something that sits in front of the detector to give a few reference pixels for what a truly dark sky should look like. I suppose it helps contextualise the rest of the image.

Anyway, I had gone through the suggested steps of adding a model only to receive the above error with everything I tried. Only when I followed the `pyxel.load()` function with my debugger and looked at the breadcrumbs of the file that was being debugged did I realise that the function actually called the `site-packages` version of `pyxel`, and not the version in the root of my project directory that my code editor was telling me was being used. 

Even more strange is that when I tried to remove `pyxel` from pip with `pip uninstall pyxel`, it told me it wasn't installed at all. Not to worry, I just deleted the folder in `site-packages` and it had the same effect.

However, that caused the `import` statement to fail since it couldn't find the `pyxel` folder anywhere. This is fixed by simply adding the folder where you have your development version of `pyxel` to your system path. I chose to do this temporarily with:
```python
import sys
sys.path.append("/path/to/my/dev/Pyxel")
import pyxel
```

It's probably worth saying that the path above then contains the actual `pyxel` (with a lower case "p"). 

Now my model imports correctly and I can finally get to working on the functionality (but not before I write the tests for it).
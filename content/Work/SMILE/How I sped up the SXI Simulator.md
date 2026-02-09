When optimising code, knowing where to put the effort is just as important as doing the effort itself. For this python script, it called upon a number of libraries, some custom-built for this project. I decided to use a mixture of perf and my own timing class to find out which part of it was slow. Initial runnings showed the script taking about 13 minutes or so to run, and with this particular example, it gave a mathematical output of 12.31 repeatedly.

Here's the output of the perf report, basically saying where it thought the bulk of the execution took place:

```bash
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 1M of event 'cpu-clock:upppH'
# Event count (approx.): 118276726490
#
# Children      Self  Command  Shared Object                                             Symbol                                                                                                                 >
# ........  ........  .......  ........................................................  .....................................>
#
    62.28%    62.28%  python3  _sigtools.cpython-310-x86_64-linux-gnu.so                 [.] pylab_convolve_2d
    31.23%    31.23%  python3  _sigtools.cpython-310-x86_64-linux-gnu.so                 [.] DOUBLE_onemultadd
     1.58%     0.00%  python3  [unknown]                                                 [.] 0000000000000000
            |
            ---0
               |
                --0.98%--_PyEval_EvalFrameDefault

     0.98%     0.98%  python3  python3.10                                                [.] _PyEval_EvalFrameDefault
            |
             --0.98%--0
                       _PyEval_EvalFrameDefault

     0.78%     0.78%  python3  libscipy_openblas64_-6bb31eeb.so                          [.] blas_thread_server
            |
            ---blas_thread_server

     0.53%     0.00%  python3  [unknown]                                                 [.] 0x00007fafeb44a580
            |
            ---0x7fafeb44a580

     0.42%     0.42%  python3  libscipy_openblas-c128ec02.so                             [.] blas_thread_server
     0.36%     0.00%  python3  [unknown]                                                 [.] 0x000055c3d2a477c0
     0.24%     0.00%  python3  [unknown]                                                 [.] 0x000055c3d2a4b9a0
     0.21%     0.00%  python3  [unknown]                                                 [.] 0x00007fafeb44b420
     0.20%     0.20%  python3  python3.10                                                [.] PyObject_Malloc
...
```

As you can see, it seems like ~60% of the execution time is spent convolving the matrix. This is because, looking further into the symbol, it's from `scipy.convolve2d()`. There actually exists a faster way to do convolutions when you're working with large arrays. 3Blue1Brown has an [excellent YouTube video](https://www.youtube.com/watch?v=KuXjwB4LzSA) on this where you can start understanding convolutions, and he even gives an introduction to something called a Fast Fourier Transform, which is what we'll be leveraging here.

So that's simple enough then, we'll just replace `scipy.convolve2d()` with `scipy.fftconvolve()`!

![[Pasted image 20260209174600.png]]

Ah okay, that's a lot of mentions (178), we don't actually know which one is the important one because this isn't our codebase and we don't know how it works. This is where I did some custom timing to work out which convolution operation took the time up. Here's my code, please steal it.

```python
import time
import pprint

class Timing():
    def __init__(self):
        self._begin = time.perf_counter()
        self._iter = time.perf_counter()
        self._card = {}
  
    def add(self, key):
        if key in self._card:
            self._card[key] = self._card[key] + (time.perf_counter() - self._iter)
        else:
            self._card[key] = (time.perf_counter() - self._iter)
        self._iter = time.perf_counter()
  
    def print(self):
        pprint.pprint(self._card)
```

I instantiate this on the main script and basically just pass it through the various functions that look important. Whenever it looks like something interesting has just happened, I call `timer.add("something recognisable")` . At the end of the script when I call `timer.print()` , it spits something a bit like this out (indentation added afterwards for clarity, since you can't see the code hierarchy and what area includes which other area):
```bash
Subsolar Magnetopause Estimate = 12.31 RE
Calculate Final Image...
Created: ./SMILE_SXI_L3_SCIM15-SCI-CXF_20260317T0240-20260317T0245_V01_1x1_fitted_cmem2g_nmd_BATSRUS_hybrid1.fits
{'fit': 40.424093454001195,
   'SXI sim': 1465.588850321994,
   'render': 4.614281184997708,
 'initialisation': 4.7030999667185824e-05,
 'pre-process': 0.48345375300050364
 }
```

This is a bit of a mess to someone who doesn't know what they're looking at, I get that. Pay particular attention to the numbers next to "fit" and "SXI sim". That's how many seconds are spent in each of these areas. 

It's not pretty but [[Slow is Smooth, Smooth is Fast|it doesn't need to be]]. In the following example and for the remainder of my test runs (while I debug and continually get syntax wrong), I set the maximum iterations of the optimisation function to 2, just to get a flavour of where time is spent under the bonnet without having to wait around so long. So now the numbers are smaller:
```bash
{'fit': 38.998542902001645,
   'SXI sim': 43.93966020800872,
   'render': 0.1292730879940791,
 'initialisation': 3.830400237347931e-05,
 'pre-process': 0.5200900710042333
} 
```

Anyway, you can see that the `fit` part of the code has two really large components, 38s and 43s, which together are nearly all the computation time. So they're doing something that's taking a lot of time. After investigating these areas, it was quite easy to find that all-important call to `scipy.convolve2d()`. 

## Let's get our hands dirty
Now we've found the problematic code, and that we know a better alternative, let's make the change:
```python
    # cxmap = convolve2d(cxmap, psfmap, mode='same', boundary='wrap')
    cxmap = fftconvolve(cxmap, psfmap, mode='same')
```

When we run the code (remembering to increase our max iterations from 2 to 200, silly me), what happens?

```bash
Subsolar Magnetopause Estimate = 11.81 RE
Calculate Final Image...
Created: ./SMILE_SXI_L3_SCIM15-SCI-CXF_20260317T0240-20260317T0245_V01_1x1_fitted_cmem2g_nmd_BATSRUS_hybrid1.fits
{'fit': 1.3547999499996877,
   'SXI sim': 43.58130087899008,
   'render': 4.158290917986051,
 'initialisation': 3.809400004683994e-05,
 'pre-process': 0.4479326739992757
 }
 ```
 
 Well it's certainly faster, but we actually get a different answer, we get 11.81. Now why's that? If you watched the 3Blue1Brown video (or if you're paying attention to the arguments we are able to pass in), you might know why. Our boundaries are treated wrongly.

Treating boundaries incorrectly in ~~relationships~~ matrix convolution can get you in a lot of trouble. 

Since we can't ask `scipy.fftconvolve()` to wrap the boundary for us, we'll have to do it ourselves. You can see the kernel that we're passing in, `psfmap`, well that has a shape of (200,200). `np.pad()` handily lets us extend the array (which we need to do by at least half the kernel size in each direction). I originally wrote my own method of doing this until I found out that NumPy can just do it for me. Anyway, we extend (or pad, or reflect, or whatever) the original `cxmap` array so it plays kindly with the kernel (`psfmap` in this case), and we run the convolution. We then have to centre crop our resultant image, since otherwise it'd be 100 elements larger in each direction (300, 270 to 500, 470). And then we can use the `cxmap` as we usually would, happy with the knowledge that we've tied up all our loose ends.

```python
## Extend edges of cxmap for convolution
new_cxmap = np.pad(cxmap, 100, mode='wrap')

## Convolve
# cxmap = convolve2d(cxmap, psfmap, mode='same', boundary='wrap')
cxmap = fftconvolve(new_cxmap, psfmap, mode='same')

## Centre crop 
cxmap = cxmap[100:-100,100:-100]
```

Okay let's run it, how did we do?
```bash
Subsolar Magnetopause Estimate = 12.31 RE
Calculate Final Image...
Created: ./SMILE_SXI_L3_SCIM15-SCI-CXF_20260317T0240-20260317T0245_V01_1x1_fitted_cmem2g_nmd_BATSRUS_hybrid1.fits
{'fit': 1.5052788280008826,
   'SXI sim': 45.51753102803923,
   'render': 3.995385640017048,
 'initialisation': 4.495200118981302e-05,
 'pre-process': 0.43014994499390014
 }
```

Sweetness. Joy. Valhalla. 

It seems a lot of time is still spent inside the SXI simulator but that's just because there are so many iterations still. Our answers agree with the original script (tested multiple times over) and we're operating *way* faster, to around 50 seconds. To go further, we could make this script use the C++ renderer to shave off a large majority of those 4 seconds still spent rendering. All of this is before we even think about using parallel computing.

This will need to undergo some rigorous checks with other input examples to ensure it behaves the same as the original script, but I'm really happy with these initial results.
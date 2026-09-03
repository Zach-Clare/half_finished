My version of the L4 fitting software is giving strange cost values, very different from the code provided by my colleague, Sam. I was working through three relevant codebases yesterday (Sam's utilising Python, mine utilising both Python and C++). Sam's cost values tend to start around 14500 and finish around 1340. Mine sometimes began at 10,000 and other times 300, depending on what I had changed, fixed, or (speaking honestly) broken. We know Sam's code to be scientifically correct, so let's investigate start to finish.
## Initial Parameters
Let's check that the initial parameters we give to the renderer are correct. To do this, let's join together the command given to the renderer so we can steal it with the debugger just before it gets sent.

```python
    def render(self, outfile, datfile = False):
        self.filename = outfile
		
        outfile = str(outfile)
        # run command and pass outfile name
        cmd = [ 
            self.exe,
            "-c",
            "-o" + outfile,
            "-h" + str(self.params['h']),
            "-w" + str(self.params['w']),
            "-s" + str(self.params['s']),
            "-x" + str(self.params['x']),
            "-y" + str(self.params['y']),
            "-z" + str(self.params['z']),
            "-a" + str(self.params['a']),
            "-d" + str(self.params['d']),
            "-p" + ','.join([str(self.params['p0']), str(self.params['p1']), str(self.params['p2']), str(self.params['p3'])]),
            "-q" + str(self.params['B']),
            "-g" + str(self.params['A1']),
            "-m" + str(self.params['dbeta'])
        ]
		
        debug_cmd = " ".join(cmd)
		
        # if (datfile):
            # cmd.append("-m")
        return subprocess.run(cmd, stdout=subprocess.DEVNULL) 
```

`debug_cmd` in this toy case instance reads:
``` bash
/home/zc/code/birp/cpp/build/birp -c -otestfile -h27 -w16 -s1 -x-4.11704 -y-0.471463 -z8.02912 -a6.90359 -d-0.15531549356613475 -p10.0,0.46258744242873295,9.708627593503989,1.4012313072472948 -q3.0539863145285135 -g2.4028344763593523e-05 -m0.8646178486919
```

I would *love* to spend some time improving the argument passing to make it more readable, but for now, the renderer only supports single-character flags. In the meantime and for the purposes of this write-up, we can use the Python snippet to decode the parameters.

Probing Sam's code, let's find the initial parameters. I don't have permission to post Sam's code, so we won't see any of his Python, only extracted values (possibly along with some code descriptions, if appropriate).

His will first calculate the linear coefficients, which are as follows:
- `a`: 
	1. 1.2544e+01
	2. -1.9400e-01
	3. 3.0500e-01
	4. 5.7300e-02
	5. 2.1780e+00
	6. 5.7100e-02
	7. -9.9900e-01
	8. 1.6473e+01
	9. 1.5200e-03
	10. 3.8200e-01
	11. 4.3100e-02
	12. -7.6300e-03,
	13. -2.1000e-01
	14. 4.0500e-02
	15. -4.4300e+00
	16. -6.3600e-01
	17. -2.6000e+00
	18. 8.3200e-01
	19. -5.3280e+00
	20. 1.1030e+00
	21. -9.0700e-01
	22. 1.4500e+00
- `dn`: -2.8577493153891598
- `ds`: -2.599304334095111
- `theta_n`: 1.2438711526644841
- `theta_s`: 0.9621288473355157

I'd be lying if I said I knew what these were exactly. Anyway, Sam's code then goes on to estimate the `p0` parameter first, and then looks up usual relationships to `p0` to calculate the rest. For this example, `p0` is estimated at 10, since the collective count rates are under 100 in this example. This is the full list of initial parameters given by this second function:
- `p0`: 10
- `A1`: 2.4028344763593523e-05
- `B`: 3.0539863145285135
- `dbeta`: 0.8646178486919
- `p1`: 0.46258744242873295
- `p2`: 9.708627593503989
- `p3`: 1.4012313072472948

Wonderful, let's compare this to our arguments being passed to the renderer:

| Parameter | Mine                   | Sam's                  |
| --------- | ---------------------- | ---------------------- |
| p0        | 10.0                   | 10                     |
| p1        | 0.46258744242873295    | 0.46258744242873295    |
| p2        | 9.708627593503989      | 9.708627593503989      |
| p3        | 1.4012313072472948     | 1.4012313072472948     |
| A1        | 2.4028344763593523e-05 | 2.4028344763593523e-05 |
| B         | 3.0539863145285135     | 3.0539863145285135     |
| dbeta     | 0.8646178486919        | 0.8646178486919        |
Well, they're rather identical, wouldn't you say? That would give me reason to believe that the inconsistencies are happening inside the renderer. Let's explore deeper.

## The Renderer
I copy the command given to the renderer and structure it inside my `launch.json` file. This file controls the launch options when debugging my C++ code, meaning we can launch a C++ debugging session with these exact parameters without ever having to run the Python.

We can confirm that the CMEM object is being correctly initialised with these parameters, and looking in CMEM's GetSample() method, we can see these values are set correctly when taking each individual sample. 

The two renderers work quite differently when actually taking samples though which means that they're a bit difficult to compare. My renderer will generate all sample points in GSE and ask the space it's sampling to return a unit at that space. This is a reusable design, making the Camera object easy to interact with a DataCube, empirical model, or anything else we want to throw at it. It can even take 3D images through functions, if it wants to. However, CMEM operates in a spherical coordinate system, using `r`, `theta`, and `phi`. 

`r` is the distance to Earth. 
`theta` is the rotational angle across the horizon.
`phi` is how far upwards from the horizon.

At least, I think this is correct. Sometimes, Physicists swap `theta` and `phi`, but Sam has a [paper from 2025](https://www.researchgate.net/publication/388779301_Modeling_the_Magnetospheric_3D_X-Ray_Emission_From_SWCX_Using_a_Cusp-Magnetosheath_Emissivity_Model) where Appendix A shows `theta` as something unexpected. 

![[Pasted image 20260609122830.png]]
*S. Wharton et al. (2025)*

If following the RA/DEC convention, one would expect `theta` to be laying down on the floor, but it's actually the angle between the subsolar point and `r`. The appendix goes on to list a handful of helpful equations to describe this system in comparison to the cartesian system.

![[Pasted image 20260609123106.png]]

Using these, let's check our CMEM GetSample method correctly translates from `x`, `y`, and `z` to spherical.

```c++
float point[3] = {x, y, z};
    float theta_components[2] = {y, z};
	
    float r = Helper::VectorDistance(point);
	
    float theta;
    float phi;
	
    if (x != 0) {
        phi =  std::acos(x / r);
    } else {
        phi = 0;
    }
	
    if (y != 0) {
        theta = std::acos(y / Helper::VectorDistance2D(theta_components));
    } else {
        theta = 0;
    }
	
    return std::vector<float> {r, theta, phi};
```

I have to say, this looks wrong. `r` looks correct, but we'll have to try something new. Let's replace the `phi` and `theta` parts with:

```c++
float theta = std::acos(x / r);
float phi = std::atan2(z, y);
```

We'll rebuild by running `build/build.sh` and launch our debugging session to see if we get any errors. Perhaps any divisions by zero?

No errors. Let's start it from the Python to see what the cost values do. Still high, but comparing the `chi_squared` functions reveals we aren't calculating it correctly. First is the old, followed by the new. Note the use of truth image vs error image in the first line of the function.

```python
def chi_squared(countmap, truth_image) -> float:
        residuals = (countmap - truth_image)/np.sqrt(abs(truth_image))
        cost = (residuals**2).sum() # suitable for Nelder-Mead and BFGS only
        return cost

def chi_squared(countmap, truth_image, error) -> float:
        residuals = (countmap - truth_image)/error
        cost = (residuals**2).sum() # suitable for Nelder-Mead and BFGS only
        return cost
```

This first function was based off of the belief that the error image was simply the absolute values in the truth image square rooted. But now, running this debug session again shows the cost value is now very similar. Mine gets 1531 as the initial cost, whereas Sam's gets 1467. These are much closer, and if we allow mine to finish, it will settle at 1493. Sam's will settle over one hundred units lower, but the behaviour is consistent. 

For this reason, I'll overlook these cost values (for now), and focus on the more important statistic. That is, what does each codebase think the subsolar magnetopause position is? Sam's thinks it's 12.31RE. My code can't actually calculate that yet, because I've been so focused on overall process (like right now). Let's do some programming to get the final position.

Okay, my code seems to think it's 9.6RE. Clearly there are still some problems.

But right now, I need to make lunch.
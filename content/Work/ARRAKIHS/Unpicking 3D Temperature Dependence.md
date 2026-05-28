We're coming bak to the temperature dependence implementation after some time away. Since we last work on this, we've had numerous changes to the underlying detector code. For example, now, when I run the temperature dependence testing otebook, I get the folowing error:
```python
#File ~/code/pyxel/pyxel/models/charge_measurement/linearity_user_array.py:74, in output_node_linearity_poly_user_array(detector, path)
     72     bit_res = detector.characteristics.adc_bit_resolution
     73     # save as shorter names for readability
---> 74     array = array * ((vr_max - vr_min) / (2**bit_res) / gain_array)
     75 except AttributeError:
     76     array = array * detector.characteristics.charge_to_volt_conversion

#ValueError: operands could not be broadcast together with shapes (10,10,3) (10,10)
```

Now, why is this? What's being broadcasts, and why?

`gain_array` and `array` are our only two arrays here. `array` is the linearity array, that's got three dimensions. `gain_array` only has two. It only has two because we have a single value for each pixel. The linearity array has three because we have an array for each pixel. Therefore, we need an operation that will multiply each element in the array with the appropriate pixel value. How do we do that? Giving it a quick google (yes I know I'm [[Defund Tech Giants|trying to move away from Google]]),it looks like we don't want the linear algebraic version of multiplication with dot products. We want to keep rows as rows and columns as columns, so I think broadcasting is the way to do this. We can broadcast  `gain_array` to the linear `gain`'s shape. 

Since we need a lot of control here, and it's only for 3 dimensions specifically, I used the answer given in [this StackOverflow answer](https://stackoverflow.com/a/32171998) to create the following snippet:
```python
69	 try:
69	 	vr_min = detector.characteristics.adc_voltage_range[0]
70	 	vr_max = detector.characteristics.adc_voltage_range[1]
71	 	gain_array = detector.characteristics.gain_array
72	 	bit_res = detector.characteristics.adc_bit_resolution
73	 	# save as shorter names for readability
74	 	# detect 3rd dimension, meaning temperature dependence
75	 	if len(array.shape) == 3:
76	 		# convert shape to (x, x, 3) with value repeated 3 times in 3rd dimension
77	 		gain_array = np.dstack([gain_array]*3)
78	 	array = array * ((vr_max - vr_min) / (2**bit_res) / gain_array)
79	 except AttributeError:
80	 	array = array * detector.characteristics.charge_to_volt_conversion
```
The gain array is now (10, 10, 3) which is broadcastable with the linear array (because they're now the same shape). 

## The Polynomial Problem

So that's got us through the gain array issue. Now let's look at how we create and then uniquely call a new polynomial function for every pixel without using a loop. My first idea was to try and use Python's `map()`. [W3School's reference page](https://www.w3schools.com/Python/ref_func_map.asp) puts it simply by stating:

>[!quote] The `map()` function executes a specified function for each item in an iterable. The item is sent to the function as a parameter. For example: `x = map(myfunc, ('apple', 'banana', 'cherry'))`

Upon casual inspection, this seems perfect, but looking at [Numpy's Polynomial documentation](https://numpy.org/doc/2.4/reference/routines.polynomials.html), we see that each Polynomial is actual it's own function. That messes with our plans because we can't simply call the same function with different pixel values, because that wouldn't be *per-pixel* temperature dependence, just *detector-specific* temperature dependence.

Here's the current version of the code:

```python
variance_poly = []
for row in variance:
	variance_row = []
	for pixel in row:
		pix_poly = np.polynomial.polynomial.Polynomial(pixel)
		variance_row.append(pix_poly(detector.environment.temperature))
	variance_poly.append(variance_row)

variance = variance_poly
```

`pixel` ends up being our 1D array with three coefficients inside. We can create a function to construct the polynomial and then sample it in one go. To do that, our code might look like this:

```python
def construct_and_sample(coeffs: list):
	pix_poly = np.polynomial.polynomial.Polynomial(coeffs)
	return pix_poly(detector.environment.temperature)

variance_poly = []
for row in variance:
	variance_row = []
	for pixel in row:
		variance_row.append(construct_and_sample(pixel))
	variance_poly.append(variance_row)

variance = variance_poly
```

Now we're talking. Do you see how `map()` might be useful? Tell you what, how about we flatten our input array and reshape it afterwards?

```python
def construct_and_sample(coeffs: list):
	pix_poly = np.polynomial.polynomial.Polynomial(coeffs)
	return pix_poly(detector.environment.temperature)

flattened = np.flatten(variance)
variance = map(construct_and_sample, flattened)
variance = variance.reshape(detector.geometry.shape)
```

I don't know, let's write that in my python file, not my notes application, and see what happens. I'm expecting it to not work for some reason.

...yeah, of course. If we flatten the list, we will of course not be able to pass in the coefficients together. Maybe we can reshape it, rather than flatten it?

```python
flattened = variance.reshape(detector.geometry.shape[0]*detector.geometry.shape[1], variance.shape[2])
            variance_new = np.array(list(map(partial(construct_and_sample, temp=detector.environment.temperature), flattened)))
            variance_new = variance_new.reshape(detector.geometry.shape)
            variance = variance_new
```

There we go! Sort of lengthy, but that code does the same thing, along with our `construct_and_sample()` code above. I'm just checking it has the same output.

It has the same output, but when I change the temperature, the results don't seem to have any temperature dependence at all. Why?

Consider how I've set up the tests.
```python
# low has temp of 200
config_low = pyxel.load("./temp_dependence_ron_validation.yaml")

config_med = config_low
config_med.detector.environment.temperature = 250

config_hi = config_low
config_hi.detector.environment.temperature = 300
```

This comes down to how Python handles variable assignment. The above performs an assignment, creating a new object with a reference to the old one. For example, `config_hi` contains a reference to `config_low`, not it's own object. So, when we change the temperature, we're actually overwriting the `config_low` temperature as well. 

To get around this, we have to perform what's called a *deep copy*. This is sort of how it sounds. Instead of creating a copy via reference, it creates an actual duplicate object stored separately in memory. When we change that above snippet to use deep copies, we get actual real temperature dependence.

```python
import copy

config_med = copy.deepcopy(config_low)
config_med.detector.environment.temperature = 250

config_hi = copy.deepcopy(config_low)
config_hi.detector.environment.temperature = 300
```

![[Pasted image 20260528140039.png]]

The variance here is zero, so we get the same figures every time. If I introduce a temperature-independent variance (and up the temperature-dependent offset, just to space out the results a little), we get the following.

![[Pasted image 20260528140225.png]]

Amazing. It looks like the deep copies have solved our problem!
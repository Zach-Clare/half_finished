In our custom readout noise function for [[Pyxel Usage|Pyxel]], we've implemented a version of the [CMOS Readout Noise](https://esa.gitlab.io/pyxel/doc/stable/references/model_groups/charge_measuremnt_models.html#output-node-noise-cmos] there are two 2D input arrays that define the variance of the new readout noise. The first describes the offset of the noise, the other describes the sigma (or the width of variance) that can have a different offset and standard deviation for each pixel. This is important because for ARRAKIHS's CMOS chip, the image will be extremely faint, so we need to account for every pixel's variance. When the CMOS is characterised in the lab (don't ask me how they do that), they will measure a distribution of readout noise for each pixel, essentially figuring out the mean average and the standard deviation of the signal with some constant input.

## The Function
The custom function, `charge_masurement.output_node_noise_cmos_user_array()` accepts four things:
- A detector object
- The 2D offset array
- The 2D sigma (standard deviation) array
- A seeded generator object

The two 2D arrays come together to describe an offset and sigma value for each pixel, creating a [Normal distribution](https://www.mathsisfun.com/data/standard-normal-distribution.html). This distribution is sampled against using the given generator to decide the readout noise for that pixel and is added (or subtracted) from the signal array contained within the detector object. The [NumPy RNG generator](https://numpy.org/doc/2.2/reference/random/generator.html#numpy.random.Generator) has a normal function that accepts scalar values, so the core of our code looks as simple as this:
```python
noise = seed.normal(loc=offset, scale=sigma)
```
## But hold on
There's another layer of complexity here though, because the signal array needs to be in volts, but the input arrays we get from the lab are in respect to the electrons of the noise. To use the correct units, we need to apply the gain (or sensitivity). To simulate this we take the gain array, [[The User Input Gain Array Problem|saved as a detector characteristic]], and multiply the offset and sigma arrays by this value, then take our sample. 

```python
sensitivity = detector.characteristics.gain_array
noise = seed.normal(loc=offset*sensitivity, scale=sigma*sensitivity)
```

We then apply this resultant noise to our signal array. With the units problem averted, all is well in the world once again. Until they ask us to implement distributions leaning to one side. Then we're *skewed*.
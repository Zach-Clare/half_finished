Having these objects as Python allows you to edit them before passing them to the `run_mode()` method. It's a bit annoying to do it, but let's say you have to increase everything by 10% or something, you can overwrite it by calling the function listed in the `.yaml` file and saving it as the property name. For example, let's say the `.yaml` file doesn't specify a seed for the output noise, but we need a reproducible scenario under certain noise conditions. We could do the following:
```python
pipeline = config.pipeline
old_args = pipeline.charge_measurement.output_noise.arguments

pipeline.charge_measurement.output_noise = pyxel.models.charge_measurement.output_node_noise_cmos(
    detector,
    readout_noise = old_args.readout_noise,
    readout_noise_std = old_args.readout_noise_std,
    seed = 1
)
```

Most python projects use an Object-Oriented programming structure, which has the benefit of your objects having persistence and (particularly) a state. Under the hood, I don't think this project is any different, but during configuration, we'd ideally just write:
```python
pipeline.charge_measurement.output_node_noise_cmos.seed = 1234 # won't work
```

But for whatever (probably good) reason, the authors decided not to do this. It's very likely that there's a better way to achieve the above but for now this is the best I've found. It tracks that we could even include some custom properties in the `.yaml` file to programmatically fluctuate the values, for example. In best practise, we'd implement those example fluctuations in Pyxel itself, but maybe it's just a usage decision, in the same way that the [[A tool to help fitting for Magnetopause standoff distance|ensemble renderings]] for CMEM shouldn't be part of the renderer itself. Simply, perhaps it's an intended separation to suit a specific task that we wouldn't necessarily want the base project to be capable of.
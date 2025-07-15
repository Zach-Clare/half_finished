Everything is loaded from a config file, to the point that if you were to share with someone a specific detector simulation, you would simply share the `.yaml` file. That config file is read into Pyxel via:
```python
config = pyxel.load(path_to_yaml_file)
```

It's customary to then set the variables `exposure`, `detector`, and `pipeline` by grabbing them from the `config` object like this:
```python
exposure = config.exposure
detector = config.cmos_detector
pipeline = config.pipeline
```

The properties are editable, and I have a [[Editable Properties|note here explaining how]]. We'll brush over that for now, but once you've loaded the config file, you can run the actual simulation by passing the necessary arguments from the config file with:
```python
result = pyxel.run_mode(
	mode = exposure
	detector = detector
	pipeline = pipeline
)
```

> [!knowledge] You can pass the whole config with the `config` keyword argument, saving you having to pass the `exposure`, `detector`, and `pipeline` objects separately.

To see your results, you can use:
```python
pyxel.display_detector(detector)
```
 
 Which should give something like this (using example data consisting only of read-out noise):
 ![[Pasted image 20250702113716.png]]
Changing the scale from linear -> log gives some weird errors, something about missing references to what looks like an object hash. This might be a problem but we'll see.
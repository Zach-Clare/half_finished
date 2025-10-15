---
dg-publish: true
---
A nifty little operation I'd like to commit to memory (hence the fact I'm writing about it now), is array slicing in Python. It's come in useful for me a few times by now and I figured it was time to keep it in my toolbox.

It looks a little strange, which is perhaps why I've been scared of it, but this little snippet is what brought me over to the dark side of array slicing.
```python
mask = np.zeros(shape)
mask[width:-width, width:-width] = 1
```

With just these two lines, we create an array of zeros with a centre group (`width` elements away from the edges) of ones. The array slicing is on the second line of this snippet. The first mention of `width` defines the start of the group in the `x` dimension, and the second defines the end. In this application, the resultant pattern is symmetrical, but if we wanted an arbitrarily-placed window, we could change all the values where `width` currently is to get what we want. 

Similarly, if we wanted a box of darkness (as opposed to a box of light), we would initialise the array with ones and change our selected pixels to zero.

![[Pasted image 20250710155810.png]]

If all the values in our mask were 1 and we multiplied our image array with it, nothing would change. However, if some of these values in our mask are 0, we can essentially remove those parts of the image. Let's say suddenly that our light shield is not a shield at all, it's now actually a light *shade*. That would be super easy to simulate as we can use 0.2 or 0.5 instead of zero to remove only some light.

The title implies the existence of a "complex" light shield. Hopefully we never have to make that.
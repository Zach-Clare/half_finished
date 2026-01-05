The main script uses [Cyclopts](https://cyclopts.readthedocs.io/en/stable/) to support `main.py argument` type commands. The default behaviour is to load, fit, and print the result.

At the time of writing, I'm aiming to use Sam Wharton's fitting method, since it produces demonstrably accurate results. More importantly however, the program will be written in a way that the fitting method can be very easily swapped for something else.

By default, the program spits out a help message. If you pass `fit` and then a filename, it'll attempt to fit the image. Maybe this will be updated to instead specify a fit type as a command but we'll see.
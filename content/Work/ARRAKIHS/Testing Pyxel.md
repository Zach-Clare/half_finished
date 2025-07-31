Pyxel uses the PyTest framework. It has some nifty features and I actually find it quite pleasant to use, which is new for me. 

## Fixtures
These are very handy structures that help you standardize test inputs. Say you have a filename parameter to one of the functions you need to test, or even to set up the test. You can define a fixture function that will insert the returned variable into any parameter where that variable name is matched. 

For example, I use this to test a function that requires a valid path to an exported `.npy` file. I do a few tests on this function so I would need to use this file a couple of times. Instead of setting up this exported array every time I test the objective function, I can write a function to do the setup once and use it in a bunch of tests. All you need to do is prefix a function with: `@pytest.fixture` and then use the function name as an argument name when defining your test functions.

I wouldn't be surprised if there was a mock function too, but I haven't had a need for that just yet.
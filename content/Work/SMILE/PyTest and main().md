I'm using `pytest` to run unit tests for the Level 4 data products project for SMILE. When I have my test script, let's say it looks like this:

```python
import pytest
from src.sxi_l4.main import app

@pytest.fixture
def mock_fit_wharton2025(mocker):
	return mocker.patch('Fit.wharton2025')

def test_fit_calls_fit_wharton2025(mock_fit_wharton2025):
	mock_fit_wharton2025.return_value = 15
	app("fit")
	mock_fit_wharton2025.assert_called_once()
```

I would end up getting a wild error looking like this:

```console
user@COMPUTER:~/code/SXI_L4$ poetry run pytest -v
=================================================================================== test session starts ===================================================================================
platform linux -- Python 3.14.2, pytest-9.0.2, pluggy-1.6.0 -- /home/user/.cache/pypoetry/virtualenvs/sxi-l4-BboN5yha-py3.14/bin/python
cachedir: .pytest_cache
rootdir: /home/user/code/SXI_L4
configfile: pyproject.toml
plugins: mock-3.15.1
collected 0 items                                                                                                                                                                               
╭─ Error ──────────────────────────────────────────────────────────────────────╮
│ Unknown option: "-v".                                                        │
╰──────────────────────────────────────────────────────────────────────────────╯
INTERNALERROR> Traceback (most recent call last):
INTERNALERROR>   File "/home/zc/.cache/pypoetry/virtualenvs/sxi-l4-BboN5yha-py3.14/lib/python3.14/site-packages/cyclopts/core.py", line 1749, in parse_args
INTERNALERROR>     command, bound, _, ignored, _ = self._parse_known_args(
INTERNALERROR>                                     ~~~~~~~~~~~~~~~~~~~~~~^
INTERNALERROR>         tokens,
INTERNALERROR>         ^^^^^^^
INTERNALERROR>         raise_on_unused_tokens=True,
INTERNALERROR>         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
INTERNALERROR>     )
INTERNALERROR>     ^
INTERNALERROR>   File "/home/zc/.cache/pypoetry/virtualenvs/sxi-l4-BboN5yha-py3.14/lib/python3.14/site-packages/cyclopts/core.py", line 1649, in _parse_known_args
INTERNALERROR>     raise UnknownOptionError(
INTERNALERROR>     ...<2 lines>...
INTERNALERROR>     )
INTERNALERROR> cyclopts.exceptions.UnknownOptionError: Unknown option: "-v".
INTERNALERROR> 
INTERNALERROR> During handling of the above exception, another exception occurred:
INTERNALERROR> 
INTERNALERROR> Traceback (most recent call last):
INTERNALERROR>   File "/home/zc/.cache/pypoetry/virtualenvs/sxi-l4-BboN5yha-py3.14/lib/python3.14/site-packages/_pytest/main.py", line 318, in wrap_session
INTERNALERROR>     session.exitstatus = doit(config, session) or 0

...

INTERNALERROR>   File "/home/zc/.cache/pypoetry/virtualenvs/sxi-l4-BboN5yha-py3.14/lib/python3.14/site-packages/cyclopts/core.py", line 1767, in parse_args
INTERNALERROR>     sys.exit(1)
INTERNALERROR>     ~~~~~~~~^^^
INTERNALERROR> SystemExit: 1
mainloop: caught unexpected SystemExit!
```

Sure, there are a few moving parts in the test script but it's not too crazy, right? We create a mock object so that we can tell if a certain method been called or not, then we ask our cyclopts-based app to call it with `app("fit")`. Surely an error with the test code wouldn't result in an error with both cyclopts *and* pytest errors, so what's going on? 

I know — let's simplify our test script. How about this?

```python
from src.sxi_l4.main import app

def test_nothing():
	assert 0
```

Lo and behold, we get the same error. What's happening here?

Let's take a look at the (rather bare) `main.py` file:

```python
from astropy.io import fits
from cyclopts import App

from src.sxi_l4.fit import Fit

app = App()

@app.default
def help():
	help_str = """Use the following commands:
		fit: Creates and saves a result of a fitted image. Requires string argument of input FITS file location."""
	print(help_str)

@app.command
def fit(filename: str):
	try:
		file = fits.open(filename)
	except(FileNotFoundError):
        raise("Input file not found.")
        
    input = file[0]
    
    # call our encapsulated fitting function
    fitting = Fit()
    result = fitting.wharton2025(input)
  
    # and display the data
    print(result)

app()
```

Upon casual inspection, nothing seems so out of order...but wait. Ah, that's a bit embarrassing, isn't it? It looks like `app()` is called every time, and if you remember back to your early days of learning python, you'll remember that **the script is executed upon import**.

This script was never intended to be imported so that makes sense why that protection was left out, but I still think it's good (funny?) that testing can reveal flaws in the code before you even start writing the tests. 

Let's wrap that up with a bit of bubble wrap:

```python
 
if __name__ == "__main__":
    app()
```

And try running our tests again:

```console
user@COMPUTER:~/code/SXI_L4$ poetry run pytest
=================================================================================== test session starts ===================================================================================
platform linux -- Python 3.14.2, pytest-9.0.2, pluggy-1.6.0
rootdir: /home/user/code/SXI_L4
configfile: pyproject.toml
plugins: mock-3.15.1
collected 1 item                                                                                                                                                                                

tests/test_fit.py F                                                                                                                                                                       [100%]

=================================================================================== FAILURES ===================================================================================
___________________________________________________________________________________ test_nothing __________________________________________________________________________________

    def test_nothing():
>       assert 0
E       assert 0

tests/test_fit.py:15: AssertionError
=================================================================================== short test summary info ===================================================================================
FAILED tests/test_fit.py::test_nothing - assert 0
=================================================================================== 1 failed in 0.18s ===================================================================================
```

I never thought I'd be so glad to see a failing test :)
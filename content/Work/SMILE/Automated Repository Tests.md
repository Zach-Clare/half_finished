This project uses Poetry to manage packages. GitHub Runners, which run GitHub Actions, are separate spaces that can run code. In order to run pytest on a GitHub Runner, we must download and install the project code, which means installing dependencies. This, in turn, means we need to install Poetry on the GitHub Runner.

Thankfully, as is usually the case in these more benign software development moments, [somebody has done most of the hard work for us](https://github.com/snok/install-poetry?tab=readme-ov-file#testing). There's a pre-written Install Poetry action, along with a verbose example file that will checkout the files, install python, then install poetry, install dependencies, it'll even cache the virtual environment for later use, and then finally will run tests. I actually wrote my own script up until the the installing poetry part, so this is very useful for the rest of it, since caching the environment didn't even cross my mind. 

In this post, I'll talk through the various parts of the testing script that enables us to run automated pytests on the repository every time we push or create a pull request. 

## Boilerplate
Let's get the [bumf](https://www.oed.com/dictionary/bumf_n?tab=meaning_and_use#12272617) out the way. 
```yaml
name: pytest Unit Testing
run-name: ${{ github.actor }} is unit testing
on: [ push, pull_request ]
jobs:
  pytesting:
    runs-on: ubuntu-latest
    steps:
```

Include the main mechanism and intention in the name, include the developer's name in the run title (I think code ownership is a good thing), and we want this to run on every push and pull request. We then specify what jobs we want to do, which is only one job, the tests, and we'll call that `pytesting` because I think that rather succinctly gets the idea across. Run it on the latest version of Ubuntu because we want to stay up to date (I think? Maybe there's an arguement for running on a specific version of Ubuntu. Inf act, maybe we should use the same operating system version as the production server...). And finally, we start defining the steps to be taken in this job.

Okay, onto the more juicy parts.
## Get the code
This one is the most simple. There's a pre-written action called `actions/checkout`. We use version 4 and it looks like this:
```yaml
	- uses: actions/checkout@v4
```

## Act on this branch
The tests need to run on this specific branch. No point running the tests on the `main` branch if all our proposed changes are in a feature or development branch.
```yaml
	- name: Switch to Current Branch
        run: git checkout ${{env.BRANCH }}
```

## Get Python
Get our project-specific version of `python` using a pre-written action.
```yaml
    - name: Set up Python 3.14
        uses: actions/setup-python@v6
        with:
          python-version: 3.14
```

## Load Poetry cache
We're going to do something a little fancy here. As we discussed earlier, we want to use `Poetry` so we can configure the dependencies. Installing `Poetry` apparently takes around 10 seconds, so just to speed things up, we'll actually cache our `Poetry` installation and try to use that instead. We can include an integer in  `key` that we can increment to reset the cache (thanks to the [snok](https://github.com/snok/) group for this idea).

 I'm not sure about a few parts of this, so let's ask some questions:
- When is this cache created? The [action/cache documentation](https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching) documentation states that when a cache misses (i.e. the key isn't found) and the job completes successfully, the specified path is saved to the specified cache. So we don't actually need to manually save anything to the cache. 
- That's the only question I can think of right now.

```yaml
      # Load cached poetry install
      - name: Load cached Poetry installation
        id: cached-poetry
        uses: actions/cache@v4
        with:
          path: ~/.local  # the path depends on the OS
          key: poetry-0  # increment to reset cache
```

## Get Poetry
This is the first non-official action we're including. I'm temped to write a custom code block for this if I'm honest, as I don't know if I can trust this pre-written action not to change somehow. Then again, [[I thought using loops was cheating...|I'm not trying to get into goat farming]], so this is good enough for the moment. Beware, we only want to run this if we weren't able to get a cached version of `Poetry`, hence the `if`.
```yaml
      - name: Install Poetry
        if: steps.cached-poetry.outputs.cache-hit != 'true'
        uses: snok/install-poetry@v1
```

## Configure Poetry
This is where we configure `Poetry` to behave how we'd like. We set the below rules. We could include these in a `with` statement during the Get Poetry step, but if we load a `Poetry` installation from the cache, it won't be configured with the same settings it was saved with. If we separate out the steps, we configure `Poetry` correctly every time.
```yaml
      - name: Configure Poetry
        run: |
          poetry config virtualenvs.create true
          poetry config virtualenvs.in-project true
          poetry config virtualenvs.path .venv
          poetry config installer.parallel true
```

## Load the cached virtual environment
To save time installing dependencies, we can attempt to load a cached virtual environment from an older run. To make sure we this doesn't go completely pear-shaped by loading cached dependencies built for other versions or operating systems, we can save a very specific key using values from this workflow run (again, thank you [snok](https://github.com/snok/)).

```yaml
      - name: Load cached venv
        id: cached-poetry-dependencies
        uses: actions/cache@v4
        with:
          path: .venv
          key: venv-${{ runner.os }}-${{ steps.setup-python.outputs.python-version }}-${{ hashFiles('**/poetry.lock') }}
```

## Create virtual environment 
Much like the `Poetry` install, we'll need to build the virtual environment if our cache misses. It's simple enough to do this, we just need to recite the correct words to poetry, as long as we include our `if` statement again.
```yaml
      - name: Install dependencies
        if: steps.cached-poetry-dependencies.outputs.cache-hit != 'true'
        run: poetry install --no-interaction --no-root
```

## Install the root project
Again, if we include this step earlier, we could create caching issues (as the root project necessarily changes between commits), so we'll do it as a separate step here.
```yaml
      - name: Install dependencies
        if: steps.cached-poetry-dependencies.outputs.cache-hit != 'true'
        run: poetry install --no-interaction
```

## Do the tests
Finally, let's run the actual tests. Fairly simple, we just need to make sure we run the tests through `Poetry`, otherwise we'll get missing dependencies. We should find a good way to get some pretty test result on the workflow summary page. [Driocurr's pytest-summary](https://github.com/marketplace/actions/pytest-summary) seems to do that really nicely, but we can't use it because we need to go through `Poetry`. Instead, let's echo the output of the test command into the test summary variable and it'll end up being displayed on the workflow summary page. We can also include `--cov=sxi_l4` to measure the test coverage of the `sxi_l4` directory and have them displayed. Oh yeah, here's the code.
```yaml
      - name: Test with pytest
        run: poetry run pytest -ra --cov=sxi_l4 >> $GITHUB_STEP_SUMMARY
```

And actually I'll include a screenshot of what that looks like, too. I don't think it's very clean, I'd quite like to replace this with something a bit easier to understand. Getting all the required information is pretty difficult. If you have any ideas, [please get in touch](mailto:z.clare@ucl.ac.uk).
![[Pasted image 20260108152245.png]]

The coverage report looks like this in the terminal. Perhaps there's a way to use raw output?
![[Pasted image 20260108152519.png]]

## Putting it all together
And viola! Here's the final product, complete with a few comments.

```yaml
name: pytest Unit Testing
run-name: ${{ github.actor }} is unit testing
on: [ push, pull_request ]
jobs:
  pytesting:
    runs-on: ubuntu-latest
    steps:
      # Checkout the code
      - uses: actions/checkout@v4
  
      # Make sure we're using the correct branch
      - name: Switch to Current Branch
        run: git checkout ${{env.BRANCH }}
  
      # Get correct Python version
      - name: Set up Python 3.14
        uses: actions/setup-python@v6
        with:
          python-version: 3.14
          
      # Load cached poetry install
      - name: Load cached Poetry installation
        id: cached-poetry
        uses: actions/cache@v4
        with:
          path: ~/.local  # the path depends on the OS
          key: poetry-0  # increment to reset cache
  
      # Get the Poetry package manager
      - name: Install Poetry
        if: steps.cached-poetry.outputs.cache-hit != 'true'
        uses: snok/install-poetry@v1
  
      # Configure our settings in a seperate step to catch new or cached versions
      - name: Configure Poetry
        run: |
          poetry config virtualenvs.create true
          poetry config virtualenvs.in-project true
          poetry config virtualenvs.path .venv
          poetry config installer.parallel true
  
      # Attempt to load the dependencies rather than have to install them every time
      - name: Load cached venv
        id: cached-poetry-dependencies
        uses: actions/cache@v4
        with:
          path: .venv
          key: venv-${{ runner.os }}-${{ steps.setup-python.outputs.python-version }}-${{ hashFiles('**/poetry.lock') }}
          
      # Install dependencies if the venv cache missed
      - name: Install dependencies
        if: steps.cached-poetry-dependencies.outputs.cache-hit != 'true'
        run: poetry install --no-interaction --no-root
  
      # Install the root project, even if we have a cached venv
      - name: Install root project
        run: poetry install --no-interaction
  
      # Do the testing and produce a summary file
      - name: Test with pytest
        run: poetry run pytest
```
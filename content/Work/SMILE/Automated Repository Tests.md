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

## Load poetry cache
We're going to do something a little fancy here. As we discussed earlier, we want to use `Poetry` so we can configure the dependencies. Installing `Poetry` takes apparently takes around 10 seconds, so just to speed things up, we'll actually cache our `Poetry` installation and try to use that instead. We can include an integer in  `key` that we can increment to reset the cache (thanks to the [snok](https://github.com/snok/) group for this idea).
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
To save time installing dependencies, we can attempt to load a cached virtual environment from an older run. I'm not sure about a few parts of this, so let's ask some questions. 
- When is this cache created? The [action/cache documentation](https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching) documentation states that when a cache misses (i.e. the key isn't found) and the job completes successfully, the specified path is saved to the specified cache. So we don't actually need to manually save anything to the cache. 
```yaml
      - name: Load cached venv
        id: cached-poetry-dependencies
        uses: actions/cache@v4
        with:
          path: .venv
          key: venv-${{ runner.os }}-${{ steps.setup-python.outputs.python-version }}-${{ hashFiles('**/poetry.lock') }}
```
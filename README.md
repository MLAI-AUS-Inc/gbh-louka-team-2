# National Energy Market Hackathon Competition Guide

## Competition Overview

Welcome to the [MLAI Green Battery Hack!](https://www.mlai.au/hackathon) In this competition, you'll develop a strategy to optimize battery operations within an energy market simulation. Your challenge is to create algorithms, or policies, that expertly manage battery charging and discharging in response to real-time market data and incoming energy from a simulated solar panel.


![](design/setup.png)

## Key Components of the Repository

- `bot/evaluate.py`: Code which runs your policy against local data. **NEVER ALTER THIS FILE** or all your submissions will fail.
- `bot/environment.py`: The code which makes up the simulated battery + solar panel setup. **NEVER ALTER THIS FILE** or all your submissions will fail.
- `bot/plotting.py`: Utility to visualize outcomes like actions taken, market prices, battery SoC (State of Charge), and profits.
- `bot/data/`: Data used to run unit tests and a training/validation split which mirrors the exact data you will encounter during live trading.
- `bot/requirements.txt`: keeps track of all python packages you are using
- `bot/config.json`: The file which tells `evaluate.py` which policy from `bot/policies` will be used to run the battery. You can alter this but keep the format the same.
- `bot/test_*`: these are unit tests
- `bot/Dockerfile`: only used during submissions. This will help package your code into a reusable bundle that will be run by the submission system.
- `bot/dockertest.py`: a final test to run before submitting. It runs your policy against test data using docker and yout `bot/Dockerfile`. This will throw errors if your code is not correctly set up for a submission.
- `bot/DO_NOT_TOUCH`: do not touch these files! They are a working copy of `bot/evaluate.py` and `bot/environment.py`. They are used by `bot/dockertest.py` to confirm that you haven't altered those files (which you must never alter).
- `bot/policies/`: Folder containing different policy classes for battery operation.
  - `policy.py`: Base class for all policies.
  - `random.py`: A simple policy making random decisions.
  - `historical_prices.py`: An example of a policy which uses external data (this data must also be included in bot).
  - `rolling_average.py`: A more complex policy based on market price averages.
  - `__init__.py`: Script to automatically load and register policy classes.

You can change any file in this repo except `bot/evaluate.py` and `bot/envoronment.py`. We use these files when running your code so if you change these files you could cheat very easily. If you DO alter these files our code will pick up on it and your submissions will error.

## Installation

### mac/linux

1. Make sure you have python 3.8 or greater installed you can check with `python --version` or `python3 --version`
2. Create a "virtual environment" which is a specific python environment you will use for running this repo. `python -m venv venv`
    - This will create a `venv` directory in the project root. `.gitignore` has been set up to make sure git will ignore this whole directory and it will never be tracked.
    - You can confirm this has worked by going `ls` and hopefully you will see something like 
  ```
  README.md       bot             design          venv
  ```
3. activate the virtual environment `source venv/bin/activate`. This will push your terminal inside the virtual environment. From now on all `pip install` and other `pip` commands will install within this specific environment. You can tell you are inside the virtual environment because it will show up at the start of the active line in your terminal 
```
(venv) plato@loukas-Laptop bot % 
```

4. Install dependencies `pip install -r bot/requirements.txt`
    - You can confirm this has worked by using `pip list` to list all installed dependencies and it should show you a similar list of packages as what is sitting in `bot/requirements.txt`

5. Now everything should be installed correctly, but to be sure let's run the unit tests. `python -m pytest`. This will run the tests in `bot/test_environment.py` and `bot/test_evaluate.py`. If they all pass it means everything is set up correctly and you're ready to roll.

### Windows

God help you.

## Usage

### Running an Existing Strategy

The simplest thing you can do with this repo is evaluate the currently chosen policy (i.e. the chosen class from `bot/policies/*`, chosen using `bot/config.json`) against a chosen set of data using `python bot/evaluate.py --data bot/data/april15-may7_2023.csv --output_file output.json --plot`. This will run the policy against the `--data` file and automatically present you with a summary graph of what the policy did. It will look something like 

![](/design/example_graph.png)

The data which is graphed will also be available in `json` format in the `--output_file` in this case `output.json`.

**REMEMBER: NEVER ALTER bot/evaluate.py** or all your submissions will fail. You can't even add a print statement. If you want to use it to develop make a copy and name it something else and develop from that.

### Build your own Strategy

### Step 1: Develop Your Policy
Create your custom policy by extending the `Policy` class in a new file inside `bot/policies`, e.g. `bot/policies/my_policy.py`. Feel free to extend any of the existing policy classes. Your job will be to create your own `act(external_state, internal_state)` function which makes smart decisions about how to charge and discharge the solar panel and battery.

A good policy will charge the battery (form the grid and the panel) when the market price looks like it's going to be low and discharge both when it looks like it will be high, while avoiding the battery being too full or empty to take advantage of very high or low prices.

### Step 2: Test and Evaluate
Use `bot/config.json` to choose which policy to use. You do this by setting "class_name" to the name of the policy class you would like to adopt as your policy for this submission. 

Next utilize `bot/evaluate.py` to run simulations of your policy under various market conditions. This will help you understand the effectiveness of your strategy. **NEVER EDIT bot/evaluate.py**.

### Step 3: Visualize and Refine
You can use `bot/plotting.py`, or your own plotting function to graphically analyze the performance of your policy. Use these insights to refine and improve your approach. The plot will look something like:


## Submitting

You make a submission using the following steps:

1. make changes to the repo, for instance define a new policy.
2. stage these changes with git `git add -A`
    - you can tell that the changes have been tracked by using `git status` which shows all the staged changes in green
3. commit these changes, making sure to include the word "submission" somewhere in the commit message `git commit -m "submission v1"`
    - you can tell the commit has worked because when you use `git status` it will not show you any staged changes and it will say something like "nothing to commit, working tree clean" (because you just committed everything)
4. push this commit up to your repository's `main` branch on github `git push`
    - you can tell this has worked by visiting your repo on github and seeing that your changes are up in the repo too

That's it! As long as your team is registered with the organizers our submission system will automatically pull down your commit, and because it has "submission" in the commit message it will be treated as your latest submission and we will start trading it against live energy data.

If you want to make a commit without making a submission that's easy, either don't include the word "submission" in the commit message, or make a commit to another branch other than `main`.

### The submission system

Broadly this is how the submission system works: 

[](./design/system_overview.png)

Every few minutes we check everyone's repos for new commits which have "submission" in their name. If we find one we create a task to run that code on all the data we have collected up to that point, which we add to the task queue. When the task is run the results will be displayed on the leaderboard, which always shows your latest submission.

### Testing Before Submitting

If you run `python bot/dockertest.py` with `docker` installed on your system this will bundle your code using docker and run your chosen policy on `april15-may7_2023.csv`. This is exactly what we do when we run your submission through the submission system (except we use live data). If `python bot/dockertest.py` completes successfully, there is a very good chance your submission will succeed, so it is a good policy to do this, and also re-run the unit-tests before submitting.

### Docker/Dependency management (ADVANCED)

We run your policy by wrapping it in a dockerfile, building this on our server and then running it. If you want to make doubly sure that your code will be submitted correctly you can test that your dockerfile works by running 

`python bot/dockertest.py`

from the **root of the project**. This will produce an output `.json` file in `bot/results` which you can then read and plot. It will be the same file as generated by running `bot/evaluate.py` except that this was generated by building the whole `bot` directory into a docker container, and then running `bot/evaluate.py` through docker.

The whole reason we do this is so that if you introduce any wacky dependencies into your code like importing strange python libraries or including a neural network then when we try to run the code on our server we will have access to those dependencies as well. If you want to avoid errors on our end just follow these rules:
- All data your model relies on must be inside `bot`
- All python dependencies you use must also be added to `bot/requirements.txt`
- Run `bot/dockertest.py` every time before making a submission.

## Final Notes

Best of luck to all participants! We eagerly anticipate your creative and effective solutions in this challenging competition.

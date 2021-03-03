# Setup

Implementation of full stack deep learning project highlighting important components of:

- Web deployment
- CI/CD for machine learning

## 1. Clone

```sh
git clone https://github.com/SathesanThavabalasingam/fsdl-text-recognizer.git
cd fsdl-text-recognizer
```

## 2. Set up the Python environment

Run `conda env create` to create an environment called `fsdl-text-recognizer`, as defined in `environment.yml`.
This environment will provide us with the right Python version as well as the CUDA and CUDNN libraries.
We will install Python libraries using `pip-sync`, however, which will let us do three nice things:

1. Separate out dev from production dependencies (`requirements-dev.in` vs `requirements.in`).
2. Have a lockfile of exact versions for all dependencies (the auto-generated `requirements-dev.txt` and `requirements.txt`).
3. Allow us to easily deploy to targets that may not support the `conda` environment.

So, after running `conda env create`, activate the new environment and install the requirements:

```sh
conda activate fsdl-text-recognizer
pip-sync requirements.txt requirements-dev.txt
```

If you add, remove, or need to update versions of some requirements, edit the `.in` files, then run

```
pip-compile requirements.in && pip-compile requirements-dev.in
```

Now, every time you work in this directory, make sure to start your session with `conda activate fsdl-text-recognizer`.

## 3. Kick off a command

Before we get started, please run a command that will take a little bit of time to execute.

```sh
python text_recognizer/datasets/emnist_dataset.py
```

# Testing and Continuous Integration

## Linting script

Running `tasks/lint.sh` fully lints our codebase with a few different checkers:

- `pipenv check` scans our Python package dependency graph for known security vulnerabilities
- `pylint` does static analysis of Python files and reports both style and bug problems
- `pycodestyle` checks for simple code style guideline violations (somewhat overlapping with `pylint`)
- `mypy` performs static type checking of Python files
- `bandit` performs static analysis to find common security vulnerabilities in Python code
- `shellcheck` finds bugs and potential bugs in shell scrips

A note: in writing Bash scripts, I often refer to [this excellent guide](http://redsymbol.net/articles/unofficial-bash-strict-mode/).

Note that the linters are configured using `.pylintrc` and `setup.cfg` files, as well as flags specified in `lint.sh`.

Getting linting right will pay off in no time, and is a must for any multi-developer codebase.

## Setting up CircleCI

The relevant new files for setting up continuous integration are

- `evaluation/evaluate_character_predictor.py`
- `evaluation/evaluate_line_predictor.py`
- `tasks/test_validation.sh`

There is one additional file (in the top-level directory): `.circleci/config.yml`

Set up CircleCI first and then look at the new evaluation files.

Go to https://circleci.com and log in with your Github account.
Click on Add Project. Select your fork of the `fsdl-text-recognizer-project` repo.
It will ask you to place the `config.yml` file in the repo.
Good news -- it's already there, so you can just hit the "Start building" button.

Validation test files: they simply evaluate the trained predictors on respective test sets, and make sure they are above threshold accuracy.

Now that CircleCI is done building, push a commit so that we can see it build again, and check out the nice green chechmark in our commit history (https://github.com/SathesanThavabalasingam/fsdl-text-recognizer-project/commits/master)


# Web Deployment

## Serving predictions from a web server

First, we will get a Flask web server up and running and serving predictions.

```
python api/app.py
```

Open up another terminal tab (click on the '+' button under 'File' to open the
launcher). In this terminal, send some test image to the web server
running in the first terminal.

```
export API_URL=http://0.0.0.0:8000
curl -X POST "${API_URL}/v1/predict" -H 'Content-Type: application/json' --data '{ "image": "data:image/png;base64,'$(base64  -i text_recognizer/tests/support/emnist_lines/test_case.png)'" }'
```

If you want to look at the image you just sent, you can navigate to
`text_recognizer/tests/support/emnist_lines` in the file browser on the
left, and open the image.


## Adding web server tests

The web server code should have a unit test just like the rest of our code.

Let's check it out: the tests are in `api/tests/test_app.py`.
You can run them with

```sh
tasks/test_api.sh
```

## Running web server in Docker

Now, we'll build a docker image with our application.
The Dockerfile in `api/Dockerfile` defines how we're building the docker image.

In root directory run:

```sh
tasks/build_api_docker.sh
```

This should take a couple of minutes to complete.

When it's finished, you can run the server with `tasks/run_api_docker.sh`


You can run the same curl commands as you did when you ran the flask server earlier, and see that you're getting the same results.

```
curl -X POST "${API_URL}/v1/predict" -H 'Content-Type: application/json' --data '{ "image": "data:image/png;base64,'$(base64 -w0 -i text_recognizer/tests/support/emnist_lines/or\ if\ used\ the\ results.png)'" }'

If needed, you can connect to your running docker container by running:

```sh
docker exec -it api bash
```

TO DO: 
- Add deployment to web service
- Add



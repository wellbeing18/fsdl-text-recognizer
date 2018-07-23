# Text Recognizer

Project developed during lab sessions of the [Full Stack Deep Learning Bootcamp](https://fullstackdeeplearning.com/bootcamp).

## Tasks

- End-to-end on just MLP
    - [x] Make a Flask JSON API (input is image URL)
    - [x] Modify Emnist dataset class to store mapping separately, in text_recognizer/datasets
    - [x] Get web server to run in a Docker container
    - [x] Deploy web server to AWS ECS via Docker or to AWS Lambda via nice script
        - July 20 2330: Decided to go with Lambda instead. ECS scales too slowly. Lambda will be cooler.
            - But first going to try Elastic Beanstalk real quick
            - For lambda, do the building and the zipping in a Docker container
                - serverless makes it really easy: https://serverless.com/framework/docs/providers/aws/events/apigateway/#simple-http-endpoint
                    - https://medium.com/tooso/serving-tensorflow-predictions-with-python-and-aws-lambda-facb4ab87ddd
        - July 21 2300: spent whole day but figured out deployment to lambda
    - [x] Make cursive.ai a Flask web site where user can upload image and then direct it to an API URL passed a query string
        - July 21 2300: decided that this should also be a serverless app
        - Realized it's nice and easy to deploy a Flask app via https://github.com/logandk/serverless-wsgi, which makes for nice dev environment
    - [x] Re-deploy prediction API using WSGI plugin, so that it's less of a delta from the Flask web app to deploying on Lambda
    - [ ] Set up CI that runs tests and evaluation
    - [ ] IB Make script that pings API endpoint with many requests, in order to work on monitoring
    - [ ] IB Add monitoring to the app
    - [ ] IB spruce up the cursive.ai UX
    - [ ] IB make re-deploying text-recognizer faster by caching docker image or something
    - [ ] IB get cursive.ai to work on custom domain

- [x] add synthetic line dataset (import code from all-in-one notebook)
    - july 22 2330: reading about tf.estimator and tf.dataset stuff. tf has really developed, this is nice: https://www.tensorflow.org/tutorials/
    - july 23 0200: ported most of the code, should be going to sleep now
    - [x] add self-caching to speed up loads
- [ ] train all-conv network on line synthetic dataset
    - will probably have to convert data to tensorflow sooner rather than later -- but it's not something that *has* to be done
    - [x] move convert_preds_to_string to line_cnn model
    - [ ] get line_rnn also working (with ctc loss): 1hr
    - [ ] get experiment-running framework going: 1hr
    - [ ] get W&B going again: 30min
    - [ ] test different cnn archs (sliding window, FCs on top, timeditributed on top instead of bottom) on the farm
        - results should be stored under model class name in experiments/ folder: 2h

- [ ] add paragraph synthetic dataset: 2h
- [ ] train detection network to detect lines in the paragraph images: 3h

- [ ] SB write code for launching hyperparam sweep experiments
- [ ] SB code to process a cell phone image of some handwritten text to get it black and white

- [ ] write out instructions for composing a test set of paragraph images

- [ ] IB make web app for annotating paragraph images with line locations
- [ ] IB Make Gradescope autograder
    - [ ] do it the setup.sh way
    - [ ] get ibrahim's help in being able to build my own docker image
- [ ] add IAM test set and test all-conv network on it: 1hr



## Quick Start

```
export PYTHONPATH=.  # May want to put this in your .bashrc

# Get development environment set up
# First, make sure you are using whatever Python you intend to use (conda or system).
# Then you can install packages via pipenv.
pipenv --python=`which python`
pipenv install --dev

# Train EMNIST MLP with default settings
pipenv run train/train_emnist_mlp.py

# Run EMNIST MLP experiments
pipenv run train/run_emnist_mlp_experiments.py

# Update the EMNIST MLP model to deploy by taking the best model from the experiments
pipenv run train/update_model_with_best_experiment.py --name='emnist_mlp'

# Test EMNIST MLP model
pipenv run pytest text_recognizer

# Evaluate EMNIST MLP model
pipenv run train/evaluate_emnist_mlp_model.py

# Run the API server
pipenv run python web/server.py

# Build the API server docker image
docker build -t text_recognizer -f text_recognizer/Dockerfile .

# Run the API server via docker
docker run -p 8000:8000 text_recognizer

# Make a sample request to the running API server
# TODO

# Deploy the server to AWS
pipenv run tasks/deploy_web_server_to_aws.py
```

## Project Structure

```
text_recognizer/
    data/                       # Data for training. Not under version control.
        raw/                        # The raw data source. Perhaps from an external source, perhaps from your DBs. Contents of this should be re-creatable via scripts.
            emnist-matlab.zip
        processed/                  # Data in a format that can be used by our Dataset classses.
            emnist-byclass.npz

    experiments/                # Not under code version control.
        emnist_mlp/                 # Name of the experiment
            models/
            logs/

    notebooks/                  # For snapshots of initial exploration, before solidfying code as proper Python files.
        00-download-emnist.ipynb    # Naming pattern is <order>-<initials>-<description>.ipynb
        01-train-emnist-mlp.ipynb

    text_recognizer/            # Package that can be deployed as a self-contained prediction system.
        __init__.py

        datasets/                   # Code for loading datasets
            __init__.py
            emnist.py

        models/                     # Code for instantiating models, including data preprocessing and loss functions
            __init__.py
            emnist_mlp.py               # Code
            emnist_mlp.h5               # Learned weights
            emnist_mlp.config           # Experimental config that led to the learned weights

        predict/
            __init__.py
            emnist_mlp.py

        test/                       # Code that tests functionality of the other code.
            support/                    # Support files for the tests
                emnist/
                    a.png
                    3.png
            test_emnist_mlp_predict.py  # Lightweight test to ensure that the trained emnist_mlp correctly classifies a few data points.

        web/                        # Web server for serving predictions.
            api.py

    tasks/
        train_emnist_mlp.py
        run_emnist_mlp_experiments.py
        update_model_with_best_experiment.py
        evaluate_emnist_mlp_model.py
        tasks/deploy_web_server_to_aws.py

    train/                       # Code for running training experiments and selecting the best model.
        run_experiment.py           # Script for running a training experiment.
        gpu_manager.py              # Support script for distributing work onto multiple GPUs.
        select_best_model.py        # Script for selecting the best model out of multiple experimental instances.

    Pipfile
    Pipfile.lock
    README.md
    setup.py
```

## Explanation

### Pipenv

Pipenv is necessary for being exact about the dependencies.
TODO: explain that want to stay up to date with packages, but only update them intentionally, not randomly. Explain sync vs install.

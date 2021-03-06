# Project Structure

Before we get going with the labs, let's familiarize ourselves with the high-level design of the codebase.

## Follow along

```
cd lab8/
```

## Project structure

Web backend

```sh
api/                        # Code for serving predictions as a REST API.
    tests/test_app.py           # Test that predictions are working
    Dockerfile                  # Specifies Docker image that runs the web server.
    __init__.py
    app.py                      # Flask web server that serves predictions.
    serverless.yml              # Specifies AWS Lambda deployment of the REST API.
```

Data (not under version control - one level up in the heirarchy)

```sh
data/                            # Training data lives here
    raw/
        emnist/metadata.toml     # Specifications for downloading data
```

Experimentation

```sh
    evaluation/                     # Scripts for evaluating model on eval set.
        evaluate_character_predictor.py

    notebooks/                  # For snapshots of initial exploration, before solidfying code as proper Python files.
        01-look-at-emnist.ipynb
```

Convenience scripts

```sh
    tasks/
        # Deployment
        build_api_docker.sh
        deploy_api_to_lambda.sh

        # Code quality
        lint.sh

        # Tests
        test_api.sh
        test_functionality.sh
        test_validation.sh

        # Training
        train_character_predictor.sh
```

Main model and training code

```sh
    text_recognizer/                # Package that can be deployed as a self-contained prediction system
        __init__.py

        character_predictor.py      # Takes a raw image and obtains a prediction
        line_predictor.py

        datasets/                   # Code for loading datasets
            __init__.py
            dataset.py              # Base class for datasets - logic for downloading data
            emnist_dataset.py
            emnist_essentials.json
            dataset_sequence.py

        models/                     # Code for instantiating models, including data preprocessing and loss functions
            __init__.py
            base.py                 # Base class for models
            character_model.py

        networks/                   # Code for building neural networks (i.e., 'dumb' input->output mappings) used by models
            __init__.py
            mlp.py

        tests/
            support/                        # Raw data used by tests
            test_character_predictor.py     # Test model on a few key examples

        weights/                            # Weights for production model
            CharacterModel_EmnistDataset_mlp_weights.h5

        util.py

    training/                       # Code for running training experiments and selecting the best model.
        run_experiment.py           # Parse experiment config and launch training.
        util.py                     # Logic for training a model with a given config
```

# Hackathon
## Machine Learning app in production

The goal of this hackathon is to get hands-on experience with building a predictive model and wrapping it into an API.
At the end of today, you'll not only have created and validated your model, but also implemented it as part of an API that returns predictions for unseen data.

We'll supply you with:

* a dataset, on which you build your model;
* specifications for your API;
* and a server on which to run your API.

First focus on developing the API (that may give back random responses) and deploying it on your server. Second, you can go and make/improve your model.

## Data

The data you have comes from an event-organizing-company and contains events that need to be organized. Your task is to determine whether an event is fraud or not.
Data is labeled and you can use `label` as your target. There are a few different possibilities for this value, but all we care about is fraud or not fraud.


#### Input variables

We are not providing you with a description of the input variables on purpose. Part of your task is to determine what is useful to you. Pay attention though that the dataset contains many columns and those columns in turn contain some nested data!
What do these all mean? Many of them won't be useful for you, so a big part of the task is feature extraction. 

#### Output variable 

+ **Label** (desired target)
    + Is this a fraudster event? (binary: 'yes','no')


## Deployment

Each group will get access to a server that can be accessed via SSH using a password.
As the server is pretty lightweight, it's recommended to only use it for your API (i.e. you should train your model locally and only then deploy it to the server).

A suggested workflow is:

1. Create your own central git repository (on Github) and make two clones of it, one locally and one on the server;
* Build your model and API in the local repo;
* Make sure your model & API work in the local repo;
* Commit to the local git repository;
* Push to your central repo;
* Pull changes on your server repo;
* (re-)deploy 
	`$ python -m app 0.0.0.0 5000`;
* Improve your model & API locally and go to 3.

The servers already have [miniconda](http://conda.pydata.org/miniconda.html) installed, but you need to set up your environment yourself. Use an [environment file](http://conda.pydata.org/docs/using/envs.html) to align package versions between your local machine and the server, similarly to what we used for the lecture.

Read `resources.md` for more background information on working with servers, API's and development patterns.


## API

You're free to use whatever framework and language you want, but we advise you to use Python.
Let us know if you have problems building the API.

The API has to follow the following specs:

* port: 5000
* endpoint: `api/v1/predict/`
* accepts GET requests:
    * Example parameters as given in `example_request_data.json`
* returns
    * JSON content type:
    * JSON has fields
        * `sample_uuid`: unique identifier obtained from the GET parameters
        * `probability`: probability of a yes (float between 0. and 1.)
        * `label`: predicted label (float that is 0. or 1.)

We'll supply two unit testing scripts in Python to test if your API conforms to the specification above.
These scripts call your API and quickly test if the returned result is valid. (More on unit testing [here](https://jeffknupp.com/blog/2013/12/09/improve-your-python-understanding-unit-testing/)).

Depending on your API, use one of the following two testing scripts:

1. `flask_unittests.py`
    * Unit tests that integrate with Flask.
    * Assumes you'll name your Python API file `app.py` that is in the same folder
    * Assumes the file `example_request_data.json` is in the same folder
    * Run with `python flask_unittests.py`
* `requests_unittest.py`:
    * Unit tests that are more platform agnostic (i.e. use this if you write your API in R)
    * Assumes a certain host and port (see above) and that your API is running.
    * Assumes the file `example_request_data.json` is in the same folder
    * Run with `python requests_unittest.py`

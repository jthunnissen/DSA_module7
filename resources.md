# Hackathon
## Machine Learning app in production

When working collaboratively on software projects, there's some best practices that are worth keeping in mind. 
This document contains best practices that we have learnt while building production ready machine learning systems in python. 
It is by no means a thorough guide to the topic, rather a quick reference of things you might want to consider during the hackathon project.


## Viewing Markdown files

In the development world you're often confronted with [Markdown](https://guides.github.com/features/mastering-markdown/) files, a markup language similar to HTML.
Examples are of course this document and `instructions.md`.

Download a markdown viewer to render these files, such as:

* [Retext](https://github.com/retext-project/retext) (Linux)
* [Macdown](http://macdown.uranusjr.com/) (macOS)
* [Markdownpad](http://markdownpad.com/) (Windows)


## Code versioning

We suggest using a version control system such as _git_, to collaboratively work on the hackathon. A free to use (for public repositories) git server is [Github](https://github.com).

#### Basic git commands:

Clone an existing repository into a new directory:

```
$ git clone <repo_url>
```

Add files to track under version control (AKA stage):

```
$ git add file1 file2
```

Commit changes:

```
$ git commit
```

Push to a remote git server:

```
$ git push
```

Retrieve (pull) code changes from a remote git server:

```
$ git pull
```


#### Antipatterns

* Big ball of changes: many independent changes in one commit, commit often so that a commit represents a unit of work. 
* Many small commits: you probably have small fixes in your commit, test before commiting so that your code works properly.
* `git add *`: Don't naively add all changes, explicitly list the files to add with `git add file1 file2` or use `git add -p`
* `git commit -m`: Often leads to bad commit messages, use `git commit` to open an editor.
* Merge conflicts when pulling: you probably didn't pull before commit, do `git fetch` and `git status` before committing and work on different branches.


#### Resources:
 
* [Beginner tutorial](https://try.github.io/levels/1/challenges/1)
* [Intermediate tutorial on brnaching](http://learngitbranching.js.org/)
* [The git bible](https://git-scm.com/book/en/v2)


## Working with remote (unix) servers

You can connect to a remote server using the [SSH](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys) (secure shell) tool.
This tool allows you to log into remote servers and copy files over.

An SSH client comes installed with macOS, Linux and [bash on Ubuntu on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about) for Windows 10.
(If you can't install bash on Ubuntu on Windows, use [cygwin](https://www.cygwin.com/) and [Bitvise SSH](https://www.bitvise.com/ssh-client-download)).


Log in with the `ssh` command as `user` on `host` with macOS, Linux or bash on Ubuntu on Windows:

```
$ ssh user@host
```

Copy a `file` from a local origin to a remote destination, in this case the `app/` dir on the `user` home directory (`~`) on `host`:

```
$ scp file user@host:~/app
```

Copy multiple files (eg. a directory) by specifying the `-r` option:

```
$ scp -r dir user@host:~/app
```

For more details see [this tutorial](https://linuxacademy.com/blog/linux/ssh-and-scp-howto-tips-tricks/).


## Remote sessions

It is useful to use tools like [tmux](https://tmux.github.io) to manage persistent remote sessions and GNU [screen](https://www.gnu.org/software/screen/).
tmux and screen are tools that work with a terminal session to allow users to keep a session running even if they're not connected anymore. 
They will prevent a session from "timing out" and thus keep your API running on the server when you lose connection between your machine and the server.
We advise you to use tmux.

Create a new session, named `hackathon` with

```
$ tmux new -s hackathon
```

Attach to an existing remote session, named `hackathon`, with

```
$ tmux a -t hackathon
```

List remote sessions with

```
$ tmux ls
```


## Conda

[Conda](http://conda.pydata.org/docs/) is a package and environment management system for many programming languages related to Data Science.
It makes it easy to set up and replicate your development environment for different platforms (Windows, macOS, Linux).
For instance, conda contains Python packages but also installs system binaries needed by those Python packages, making it easy to port up your project from Linux to Windows.

Install conda by downloading [miniconda](http://conda.pydata.org/miniconda.html) and running the installer.
On Linux:
```
$ wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ bash Miniconda3-latest-Linux-x86_64.sh
```

A virtual environments keeps all the tools and dependencies of your project in a separate place.
Read the [documentation](http://conda.pydata.org/docs/using/envs.html) on how to use conda's virtual environments.


## API 101

Some jargon:

* __A__pplication __p__rogramming __i__nterface: defines machine-machine communication.
* __Re__presentational __S__tate __T__ransfer: architecture style for designing networked applications.
* REST API: an interface to talk to a service using HTTP request.
* An API exposes a **resource**, that is an object with a type, associated data, relationships to other resources, and a set of methods that operate on it
* A resource is exposed via an **endpoint**, that is a url
* HTTP Verbs are used to retrieve (GET), create (POST), create or update (POST) a resource
* A client issues **requests** to a server
* A server returns **responses** to a client
* Representation: services typically exchange JSON strings
* JSON = language independence (e.g. R client to Python server)


### ML api

* Resource: model predictions
* Example endpoint: http://webapp/api/v1/predict/
* Client issues a GET request with an unlabeled instance
* The server returns a prediction


### Flask

[Flask](http://flask.pocoo.org), and its extension [flask-restful](http://flask-restful-cn.readthedocs.org/en/0.3.5/), are lightweight, yet feature rich python modules that will help you to create a web API.

A flask-restful application is built around the concept of a Resource class, which is added to an endpoint and exposed to a client.


```python
"""
Simple api to serve predictions.
"""
from flask import Flask
from flask_restful import reqparse, abort, Api, Resource
import json

app = Flask(__name__)
api = Api(app)

# A training process has stored predictions into a
# json file
with open('simple.json') as f:
    PREDICTIONS = json.load(f)

def abort_if_prediction_doesnt_exist(user_id):
    if user_id not in PREDICTIONS:
        abort(404, message="User {} doesn't exist".format(user_id))

class SimpleModel(Resource):
    """
    The resource we want to expose
    """
    def get(self, user_id):
        abort_if_prediction_doesnt_exist(user_id)
        return PREDICTIONS[user_id]


api.add_resource(SimpleModel, '/predict/<user_id>')

if __name__ == '__main__':
     app.run(host="0.0.0.0", port=5000, debug=True)
```

In the above example we pass only one parameter to the GET request (`user_id`). The `RequestParser` class in `reqparse` can help up handle more complex situations.

We can test the API by using curl from the command line:

```
$ curl http://localhost:5000/predict/123
```


## Design patterns

> In software engineering, a design pattern is a general repeatable solution to a commonly occurring problem in software design. A design pattern isn't a finished design that can be transformed directly into code. It is a description or template for how to solve a problem that can be used in many different situations. 
>
> ( --Design Patterns & Refactoring)

When deploying a model to production, it might be worth decoupling data, presentation and transformation logic (e.g. MVC pattern).

Ask yourself:  

* How am I going to store the model? Do I have to fit a model each time a request is received?
* How will I return predictions once new data comes in from the API call?
* Can I predict data as is, or will I need to carry out transformations (eg. feature encoding?)
* What will happen when I want to deploy a new (type of) model? Will I need to rewrite the API code?
* How am I going to deploy my code to server? 


### Model persistence

Scikit-learn as a good [section](http://scikit-learn.org/stable/modules/model_persistence.html
) why not to save models with the default `pickle` but with `joblib`.

```
from sklearn.externals import joblib
joblib.dump(clf, 'filename.pkl')
```

Beware! `joblib.dump()` will return a list of files.
Loading still works with:

```
clf = joblib.load('filename.pkl') 
```

Note that clf itself is a sklearn object! Eg. we can call clf.predict(X) to carry out prediction on new instances.


### Pipelines

The sklearn.pipeline module implements utilities to build a composite estimator, as a chain of transforms and estimators.

```python
# From http://scikit-learn.org/stable/auto_examples/feature_stacker.html
from sklearn.pipeline import Pipeline, FeatureUnion
from sklearn.grid_search import GridSearchCV
from sklearn.svm import SVC
from sklearn.datasets import load_iris
from sklearn.decomposition import PCA
from sklearn.feature_selection import SelectKBest

iris = load_iris()

X, y = iris.data, iris.target

# This dataset is way to high-dimensional. Better do PCA:
pca = PCA(n_components=2)

# Maybe some original features where good, too?
selection = SelectKBest(k=1)

# Build estimator from PCA and Univariate selection:

combined_features = FeatureUnion([("pca", pca), ("univ_select", selection)])

# Use combined features to transform dataset:
X_features = combined_features.fit(X, y).transform(X)

svm = SVC(kernel="linear")

# Do grid search over k, n_components and C:

pipeline = Pipeline([("features", combined_features), ("svm", svm)])

param_grid = dict(features__pca__n_components=[1, 2, 3],
                  features__univ_select__k=[1, 2],
                  svm__C=[0.1, 1, 10])

grid_search = GridSearchCV(pipeline, param_grid=param_grid, verbose=10)
grid_search.fit(X, y)
print(grid_search.best_estimator_)
```


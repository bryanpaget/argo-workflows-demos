# Argo Workflows Demos

_Demo Jupyter Notebooks for Argo Workflows_

## Introduction

### Hera 

Hera is a Python software development kit (SDK) for Argo Workflows. Argo Workflows is a tool used for orchestrating and managing complex workflows in cloud-native environments.

Hera aims to simplify the process of building and submitting workflows by abstracting away many of the technical details involved. It also uses a consistent set of terminology and concepts that align with Argo Workflows, making it easier for users to learn and use both tools together. Essentially, Hera is a tool designed to make the creation and management of complex workflows more accessible to a wider audience.

### Couler

Couler provides a unified interface for constructing and managing workflows on different workflow engines, such as Argo Workflows, Tekton Pipelines, and Apache Airflow.

While there are many different workflow engines available, they can vary in terms of programming experience required and level of abstraction, which can make some difficult to work with. Couler, on the other hand, provides a simple, unified interface for defining workflows using an imperative programming style. It also automatically constructs directed acyclic graphs (DAGs) for the workflows, which can help to simplify the process of creating and managing them.

Couler is designed to be extensible, meaning it can work with various different workflow engines. It also includes reusable steps for tasks like distributed training of machine learning models, which can help to increase efficiency. Finally, Couler includes automatic workflow and resource optimizations, which can further improve efficiency and streamline the workflow creation process.

## Instructions

### Hera

**You will need an authentication token for Hera, find it here: [https://argo-workflows.aaw-dev.cloud.statcan.ca/userinfo](https://argo-workflows.aaw-dev.cloud.statcan.ca/userinfo).**

If you are on the `aaw-dev` cluster you'll need to first specify the correct URL for `pip` to find and install `hera-workflows`.

```bash
!pip config --user set global.index-url https://jfrog.aaw.cloud.statcan.ca/artifactory/api/pypi/pypi-remote/simple
!pip install hera-workflows
```

Next import Hera and give it your credentials:

```python
import hera

hera.global_config.GlobalConfig.token = "<your-argo-workflows-token>"
hera.global_config.GlobalConfig.host = "https://argo-workflows.aaw-dev.cloud.statcan.ca:443"
hera.global_config.GlobalConfig.namespace = "<your-kubeflow-namespace>"
hera.global_config.GlobalConfig.service_account_name = "<your-kubeflow-profile>"
```

From here you should be able to run an example workflow:

```python
from hera import Task, Workflow


def random_code():
    res = "heads" if random.randint(0, 1) == 0 else "tails"
    print(res)


def heads():
    print("it was heads")


def tails():
    print("it was tails")


with Workflow("coin-flip") as w:
    r = Task("r", random_code)
    h = Task("h", heads)
    t = Task("t", tails)

    h.on_other_result(r, "heads")
    t.on_other_result(r, "tails")

w.create()
```

More information about Hera can be found on [Github](https://github.com/argoproj-labs/hera-workflows) and their official [documentation](https://hera-workflows.readthedocs.io/).


### Couler


If you are on the `aaw-dev` cluster you'll need to first specify the correct URL for `pip` to find and install `couler`.

```bash
!pip config --user set global.index-url https://jfrog.aaw.cloud.statcan.ca/artifactory/api/pypi/pypi-remote/simple
!python3 -m pip install git+https://github.com/couler-proj/couler --ignore-installed
```

Then import Couler as below and provide your namespace:

```python
import json
import random

import couler.argo as couler
from couler.argo_submitter import ArgoSubmitter


NAMESPACE = "<your-kubeflow-namespace>"
```

Then you should be able to run an example workflow:

```python
def random_code():
    import random
    res = "heads" if random.randint(0, 1) == 0 else "tails"
    print(res)


def flip_coin():
    return couler.run_script(image="python:alpine3.6", source=random_code)


def heads():
    return couler.run_container(
        image="alpine:3.6", command=["sh", "-c", 'echo "it was heads"']
    )


def tails():
    return couler.run_container(
        image="alpine:3.6", command=["sh", "-c", 'echo "it was tails"']
    )


result = flip_coin()

couler.when(couler.equal(result, "heads"), lambda: heads())
couler.when(couler.equal(result, "tails"), lambda: tails())

submitter = ArgoSubmitter(namespace=NAMESPACE)
result = couler.run(submitter=submitter)

print(json.dumps(result, indent=2))
```

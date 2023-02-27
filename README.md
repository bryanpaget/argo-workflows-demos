# Argo Workflows Demos

_Demo Jupyter Notebooks for Argo Workflows_

## Introduction

Introducing the Argo Workflows Demos - a collection of Jupyter Notebooks showcasing the power and simplicity of using Argo Workflows for complex workflow orchestration in cloud-native environments. With the help of Hera and Couler, two Python SDKs for Argo Workflows, building and submitting workflows has never been easier.

Hera abstracts away technical details, providing a consistent set of concepts and terminology, making complex workflows accessible to a wider audience. Couler, on the other hand, offers a unified interface for constructing and managing workflows on various workflow engines, including Argo Workflows, Tekton Pipelines, and Apache Airflow. Its extensibility, automatic optimizations, and reusable steps make it a powerful tool for streamlining the workflow creation process.

Explore our collection of Jupyter Notebooks and get inspired to build and manage your own complex workflows with ease!

#### Container Repository Warning

Due to security concerns, you can only use containers from the following repositories:

```python
ALLOWED_CONTAINER_REPOS = ["jfrog.aaw.cloud.statcan.ca/aaw-user-docker/", "k8scc01covidacr.azurecr.io/", "k8scc01covidacrdev.azurecr.io/", "gcr.io/ml-pipeline/frontend:", "gcr.io/ml-pipeline/visualization-server:", "gcr.io/ml-pipeline/kfp-launcher:", "gcr.io/kfserving/sklearnserver", "gcr.io/kfserving/storage-initializer:", "gcr.io/knative-releases/knative.dev/serving", "seldonio/", "docker.io/seldonio/", "docker.io/istio/proxyv2:", "docker.io/bitnami/postgresql:", "gitea/gitea:", "vault:", "hashicorp/vault:", "argoproj/argosay:", "quay.io/argoproj/argoexec:", "siscc/", "docker.io/andrewgaul/s3proxy:", "docker.io/nginxinc/nginx-unprivileged:", "trinodb/trino:", "bitsondatadev/hive-metastore:"]
```

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

## Viewing Your Workflow

Regardless of which workflow interface you choose to use, the job is submitted as an Argo Workflows workflow. Your workflows can be viewed from the Argo Workflows  web interface located at https://argo-workflows.aaw-dev.cloud.statcan.ca/.

![image](https://user-images.githubusercontent.com/8212170/221681210-016dccbf-eb07-4977-b7ff-a3f1643257e3.png)
![image](https://user-images.githubusercontent.com/8212170/221681409-4dd1e723-eff4-4e9a-aed9-955ce1b61efb.png)


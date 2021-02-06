# Kubeflow

## Kubeflow pipelines

Built on the Argo workflow engine: a container-native engine for orchestrating parallel processing jobs in kubernetes.
Argo workflow engine stores pipelines and the metadata from previous runs.


Pipelines leverage cluster autoscaling & node auto-provisioning to carry out different steps of the pipeline on appropriate hardware.

Pipelines can be created via the Kubeflow SDK or the TFX SDK + TFX pipeline templates; both submit to KF using rest api.

Which SDK to use?

| Kubeflow | TFX |
| --- | --- |
| Framework-agnostic, lower-level implementation | Higher-level abstraction, but TF-specific |
| Direct control of K8s resources | Prescriptive components (customisable) w pre-defined ML types |
| Enables simple sharing of containerised components | Encourages Google best practices for robust / scalable ML workloads |
| Ideal for fully custom pipelines (and non-TF frameworks) | Ideal for E2E TF-focused pipelines with some customisation of pre-processing / training code |

### When to use pipelines?

- Orchestration of workflows
- Repeatable, reproducible experimentation
- Sharing, re-use & composition


## KF Pipelines

KF provides a domain-specific language (DSL) for describing a sequence of interdependent tasks as a DAG using Python.

### Describing a pipeline

```python
import kfp # Import the pipelines SDK

@kfp.dsl.pipeline(
    name="ML Pipeline Name",
    description="A more verbose description of what this pipeline does"
)
def pipline_method_name(pipeline_params):
    # Specify the steps / componenets / operations from which the DAG is composed
    ...
```

Conditionality between operations can be facilitated using `with kfp.dsl.Condition(<condition>):`

## Working with KF components

There are 3 levels of customisation for working with components:

1. Pre-built components: just load the component from a URI & compose your pipeline
2. 'Lightweight' Python components: implement the operation code in Python, but all containerisation is taken care of BTS
3. Custom components:
    - Implement the component code
    - Package into a Docker container
    - Write the component description
    
### Pre-built components

[Pre-built components GitHub repo](https://github.com/kubeflow/pipelines/blob/master/components)

To use a pre-built component in your pipeline:

```python
import kfp

URI = "https://raw.githubusercontent.com/kubeflow/pipelines/0.2.5/components/gcp/"

component_store = kfp.components.ComponentStore(
   local_search_paths=None,
   url_search_prefixes=[URI]
)

bq_query_op = component_store.load_component('bigquery/query')
mlengine_training_op = component_store.load_component('mlengine/train')
mlengine_deploy_op = component_store.load_component('mlengine/deploy')
```

Documentation for each pre-built component is available on GitHub.

PB components can be composed (chained) by using the output from one component as an input for the next. 

### Lightweight Python components

Sometimes, we need a little more flexibility than the pre-built components offer, but we'd prefer to avoid the effort of
building container images for just a few lines of Python code.
In this case, we can use the `func_to_container_op` function:

```python
import kfp

from kfp.components import func_to_container_op

def my_python_logic(project_id, run_id):
    # your logic here

BASE_IMAGE = # configure the base image you want to use

my_op = func_to_container_op(
   my_python_logic,
   base_image=BASE_IMAGE
)
```

And then we can use `my_op` as we would any of the pre-built components.

This approach is best when we require greater flexibility than the pre-built components allow, but can achieve it with a small 
amount of simple python code.

### Custom components

This approach is relevant when we need:
- many files to construct the op
- many dependencies not available in the base images
- to use logic written in other languages

Step to writing your own component:

1. Write your own code (any language)
2. Package this code into a container image using a Dockerfile
3. Build & push the docker image to gcr.io
4. Describe your component in a `.yaml` [description file](https://www.kubeflow.org/docs/pipelines/sdk/component-development/#writing-your-component-definition-file)
5. Use the description file to load the component into your pipeline

The command specified in the implementation section of the description will be supplied as the entry-point when running the container.

To make use of the custom component, all you need is:

```python
import kfp
custom_component = kfp.components.load_component_from_file(URI_TO_YAML_DESCRIPTION)
output = custom_component(...)
```

### Wrapping it up

Compile >> Upload >> Run

In order to compile & run a pipline, we must:

1. Upload the training container to the registry
2. Push any base containers required for lightweight ops to our registry
3. compile the pipeline with `$ dsl-compile --py path/to/pipeline.py --output filename.yaml`
4. Upload the pipeline to the KF cluster: `$ kfp --endpoint $ENDPOINT pipeline upload -p $PIPELINE_NAME pipeline_name.yaml`

Running our pipelines

`$ kfp --endpoint $ENDPOINT run submit ...`

## CI/CD for KF pipelines with Google Cloud Build

![CI / CD for ML](automating_training.svg)

For each pipeline, we want a repo where every component container has a self-contained directory; each change results in automated rebuilds
and a push to the registry.

Our goal will be to run an appropriate cloud builder each time a trigger condition is met.

### Cloud build builders

`Cloud builders` are configuration / provisioning actions which are packaged as docker containers.

Example actions:
- Building an image from a Dockerfile
- Pushing an image to a project registry
- Deploying compute resources
- Uploading a pipeline to AI Platform pipelines

Builders fall into 1 of 2 categories:
1. Standard: pre-packaged config actions
   - registry: `gcr.io/cloud-builders/`
   - each builder provides a Dockerfile and a script to run when the container is executed (you effectively just provide some arguments to the script)
2. Custom: config actions that are packaged by you
   - registry: `gcr.io/<YOUR_PROJECT>/`
   - you have to write your own Dockerfile & script, and push it to your registry

### Cloud build configuration

The cloud build configuration file tells Cloud build which builder to run when a specific trigger is detected.

Example cloud build config

```yaml
# cloudbuild.yaml
steps:
- name: 'gcr.io/cloud-builders/docker' # URI of the builder's container image
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/$_TRAINER_IMAGE_NAME:$TAG_NAME', '.'] # args to be passed to the container entry-point
  dir: $_PIPELINE_FOLDER/trainer_image # specifies the working directory in the container from which the entry point will be run
  env: # These are passed to the container as environment variables
     - 'PYTHON_VERSION=$_PYTHON_VERSION'

# Running a builder with the config above will build the images locally, but won't push them to anywhere they can actually be used...
# To push them to the registry, we also need to specify:

Images:
   - 'gcr.io/$PROJECT_ID/$_TRAINER_IMAGE_NAME:$TAG_NAME'
```

Triggering a cloud build

`$ gcloud builds submit . --config cloudbuild.yaml --substitutions $SUBSTITUTIONS`

If we use a relative path for the dir, then changes to the contents of the dir will be shared across all steps executed during the build
(normally they would not be available to different instances of a container)

### Cloud build triggers

Running a builder manually:
1. Get a local copy of the cloudbuild.yaml
2. Run `$ gcloud bulds submit ...`

But we can also automate the running of builds given an appropriate trigger :)

1. Activate the GCP Cloud Build app in GitHub to work with your repositories
2. Activate your repositories for monitoring in cloud build

## Local installation notes

I had a heap of trouble getting a local installation of kubeflow up and running so I could play with the features without needing to deploy (and pay for) a cluster in a cloud.

At the time of running into the issues (late 2020, early 2021), I'm running:
- OS: Pop!_OS 20.04 LTS (Ubuntu 20.04 equivalent)
- GPU: Nvidia GTX 2070
- Memory: 32GB

For this set-up, I wasn't a big fan of the MiniKF approach because I'm already running an ubuntu machine, so having to install another one in a virtual machine seemed unnecesary.

Instead, I really enjoyed getting up and running with microk8s: super-easy to install (with snap) and maintained by the team at [Canonical](https://canonical.com).

In theory, deploying kubeflow in the microk8s environment is as easy as `$ microk8s enable kubeflow`. However, I kept hitting a wall when one of the components (OIDC Gatekeeper) would fail to run.
There were [some issues](https://github.com/kubeflow/kubeflow/issues/5407) reported around this and the suggested solutions/workarounds either didn't work for me, or didn't look complete.

Eventually though, I tried the edge release and had success deploying the 'lite' flavour of KF. So I'm going to stash a summary of what I did to reduce the risk of running into this again!

```shell
# Install microk8s from snap, using the 'latest/edge' channel
# At the time of writing, this installed v1.20.2 from Canonical
sudo snap install microk8s --classic --channel=latest/edge

# Enable a number of components that kubeflow will be dependent on
microk8s enable dns dashboard storage gpu

# Enable the kubeflow component, with a few non-default settings:
# KUBEFLOW_DEBUG: turns on verbose logging, not necessary but helped when I was trying to fix things
# KUBEFLOW_BUNDLE: 'lite' installs a lighter-weight bundle of KF components
# KUBEFLOW_AUTH_PASSWORD: set your password for the KF dashboard
KUBEFLOW_DEBUG=true KUBEFLOW_BUNDLE=lite KUBEFLOW_AUTH_PASSWORD=xxxxxxxx microk8s enable kubeflow

# After a few minutes, you'll hopefully see the following:
# Congratulations, Kubeflow is now available.
#
# The dashboard is available at http://XX.XX.XXX.XX.xip.io
#
#    Username: admin
#    Password: xxxxxxxx
#
# To see these values again, run:
#
#    microk8s juju config dex-auth static-username
#    microk8s juju config dex-auth static-password
```

At this point, KF is running in your microk8s 'cluster', but you may not be able to open the KF dashboard when you open the link they provided in your browser (the "http://XX.XX.XXX.XX.xip.io").
To sort that out, I also needed to do the following:
```shell
sudo apt-get install -qq -y iptables-persistent
sudo iptables -P FORWARD ACCEPT
sudo -- sh -c "echo 'XX.XX.XXX.XX\tXX.XX.XXXXX.xip.io' >> /etc/hosts"
```
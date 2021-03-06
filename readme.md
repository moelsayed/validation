# General Notes about this framework current status:

## ENV Variables:
If you add any new ENV variables, check the scripts/rke/configure.sh will pass them into the container for Jenkins. Running on your own machine, export these variables as needed.

### General
DEBUG defaults to 'false', Prints the output from kubectl and rke commands when 'true'

### Base Cloud Provider variables:
```
OS_VERSION defaults to 'ubuntu-16.04', Used to select which image to use
DOCKER_VERSION defaults to '1.12.6', Used to select image to use if DOCKER_INSTALLED is 'true'
DOCKER_INSTALLED defaults to 'true', When false, base image is used for OS_VERSION, and docker version DOCKER_VERSION is installed 
```

### AWS specific variables:
```
AWS_ACCESS_KEY_ID no default, Your AWS access key id
AWS_SECRET_ACCESS_KEY no default, Your AWS secret access key
AWS_SSH_KEY_NAME no default, the filename of the private key, e.i. jenkins-rke-validation.pem
AWS_CICD_INSTANCE_TAG defaults to 'rancher-validation', Tags the instance with CICD=AWS_CICD_INSTANCE_TAG
AWS_INSTANCE_TYPE defaults to 't2.medium', selects the instance type and size
```

## RKE template defaults variables:
```
DEFAULT_K8S_IMAGE defaults to 'rancher/k8s:v1.8.7-rancher1-1', defaults the templates service images
DEFAULT_NETWORK_PLUGIN defaults to 'canal', defaults the templates to use the select network plugin
```

## Passing other pytest command line options
In the Jenkins job parameters PYTEST_OPTIONS can be used to pass additional command line options to pytest like test filtering or run in parallel:

Run tests methods that match 'install_roles'
PYTEST_OPTION = -k install_roles

Run in parallel:
PYTEST_OPTION = -n auto

Multiple options can be passed in '-k install_roles -n auto'

## Issue 316
Currently an issue with Ubuntu-16.04, https://github.com/rancher/rke/issues/316
prevents us from using a AMI where docker is already installed.

At first we tried rebooting in rancher validation/lib/aws.py:
```
        if wait_for_ready:
            nodes = self.wait_for_nodes_state(nodes)
            # hack for instances
            # self.reboot_nodes(nodes)
            # time.sleep(5)
            # nodes = self.wait_for_nodes_state(nodes)
            for node in nodes:
                node.ready_node()
        return nodes
```
After a while I still ran into the issue. If you need to get around this, the setting the ENV variable: DOCKER_INSTALLED=false will instead use
the base ubuntu-16.04 image provided by AWS and install the version of docker specificed by DOCKER_VERSION

## Docker image container-util
I added files images/container-utils as tool to test DNS and Intercommunication between pods/containers. It is a simple flask application, but the image also includes cli tools like 'curl', 'dig', and 'ping'

## Helpful docs:
AWS boto3 package docs:
https://boto3.readthedocs.io/en/latest/reference/services/ec2.html

DigitalOcean package docs:
https://github.com/koalalorenzo/python-digitalocean

Invoke docs (used to run commands like 'kubectl' and 'rke'):
http://docs.pyinvoke.org/en/latest/

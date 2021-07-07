# OWASP ZAP on Openshift

This repository contains all relevant parts to deploy the OWASP ZAP on OpenShift.

## Build a new Image that can be run on OpenShift

The public docker registry version of OWASP ZAP is not compatible with OpenShift without using privleged containers. To resolve this issue a new image with less privaliges has to be created. 

In OpenShift there is an option to create a new image from a Dockerfile, but unfortunately the IBM github repo is only accessible within the IBM network, which makes this option not really useful. You can download the files of this repo and host it in a public git to re-enable the option.

Another way is to build the image locally and upload it to a public repo on Docker Hub.

To do this, the first step is to build the new image from the Dockerfile:
```
$ export MY_DOCKER_ID="mydockerid"
$ docker build -t $MY_DOCKER_ID/owasp-zap-for-openshift:latest .
```

After that, you can upload it to your own Docker Hub repo:
```
$ docker login
$ docker push $MY_DOCKER_ID/owasp-zap-for-openshift:latest
```

Check if the upload was successful and make sure the image is part of a public repo!
	
## Deploy OWASP ZAP in OpenShift

After publishing the customized image on Docker Hub, this source can be used to deploy OWASP ZAP to OpenShift.

First you have to define an api key for the OWASP ZAP API. ZAP requires an API Key to perform specific actions via the REST API.

```
export MY_API_KEY="change-me-9203935709"
```

Apply the template from this repo and add your values for the required parameters:

```
$ oc new-app -f template-deployment-owasp-zap.yaml \
    -p NAMESPACE=default \
    -p IMAGE_SOURCE=$MY_DOCKER_ID/owasp-zap-for-openshift:latest \
    -p API_KEY=$MY_API_KEY
```

The deployment should now start. The following components are deployed:
* ImageStream - for the custom OWASP ZAP Image
* DeploymentConfig - for OWASP ZAP Container
* Service - to reach the ZAProxy within the Cluster

## Test the deployment

Start a remote shell into a pod and try to access OWASP ZAP API:

```
$ oc rsh <pod_name>
$ curl http://owasp-zap-service:8080/JSON/pscan/view/recordsToScan/?apikey=$MY_API_KEY
```

As a response you should see the number of records left to be scanned, which should be 0.

```
{"recordsToScan":"0"}
```

## Updating the OWASP ZAP
This deployment does not automatically update the OWASP ZAP. 

When there is a new release, OWASP will release a new version of their OWASP ZAP Docker image (stable). Since this is the base image of the customized Docker image for Openshift, the OWASP ZAP Openshift image must be recreated following the steps from before. 

After uploading the new image to DockerHub, the ImageStream in OpenShift will detect the new version and update the deployment automatically.
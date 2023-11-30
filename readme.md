# GemFire Cache.xml Configuration with Kubernetes Demo


In situations where end users lack ownership of the platform, effectively managing files on the file system poses a challenge. VMware GemFire for Kubernetes tackles this issue by offering a method to pre-position items on the file system before GemFire initiates, leveraging containerization. Primarily, these files comprise Java JAR files aimed at extending GemFire's capabilities. However, the scope of these files extends beyond JAR files; for instance, GemFire also accommodates declarative configuration through a `cache.xml` file.

This demo leverages GemFire for Kubernetes approach to getting libraries onto container file systems. These "libraries" are essentially files, and from an operating system perspective, they are just bits on a disk so lets just use that and not worry that it has the label "libraries".  

In a future release, there might be a change in the terminology of "libraries" to be more inclusive, but this would likely wait for a major release to avoid API changes.

## How It Works

Lets begin by reading about the libraries in the GemFire for Kubernetes Custom Resource Definition (CRD) documentation: https://docs.vmware.com/en/VMware-GemFire-for-Kubernetes/2.3/gf-k8s/crd.html

The operator implementation copyies the specified files from an image into the GemFire container into a standard location: /gemfire/extensions.

Why use an image? When designing this feature, it was challenging to predict how customers would set up their infrastructure. The decision to use a container image was made because Kubernetes leverages container images so a common delivery mechinism that needed to be in place.

You can observe the final results with the following commands:

```bash
> kubectl -n gemfiredemo exec -it demo-server-0 -- bash
Defaulted container "server" out of: server, gemfire-init (init), sample-cache-xml (init)
root [ /data ]# cd /gemfire/extensions/
root [ /gemfire/extensions ]# ls
demo-cache.xml
root [ /gemfire/extensions ]# cat demo-cache.xml
<?xml version="1.0" encoding="UTF-8"?>
<cache
        xmlns="http://geode.apache.org/schema/cache"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://geode.apache.org/schema/cache http://geode.apache.org/schema/cache/cache-1.0.xsd"
        version="1.0">

    <pdx persistent="true"/>
    <region name="test" refid="PARTITION_REDUNDANT"/>
</cache>root [ /gemfire/extensions ]#
```

## Building the Image
In the image directory, you can find all the required files and commands used to build and push the image to Docker Hub. Your enterprise might use a different container repository, so adjust the image name accordingly.

GemFire Cache XML is an example of a GemFire cache.xml.
The Dockerfile extends busybox as the base image and copies the GemFire Cache XML into the image.
Build and push the image with commands like:

```bash
docker build -t charliemblack/gemfire-k8s-cache-xml .
docker push charliemblack/gemfire-k8s-cache-xml
```

## How to duplicate 

This repository includes all the necessary commands in a [README](/k8s/readme.md) for installing prerequisites for GemFire and the GemFire Operator.  The certificate manager does take some time to install asynchronously, so I put a sleep in the command list which may or may not be enough depending on how much resources your K8s environement has.    The [cluster.yml](k8s/cluster.yml) file is provided to demonstrate the syntax on how to use this method is used to copy the `cache.xml`` file to the container and inform GemFire of its location.


### The secret.yml File
I'm including information about deploying secrets because it might not be widely known. I prefer using base64 encoding from the ~/.docker/config.json for logging into a private repo.

Here's a sample secret.yml (not checked in due to containing sensitive information):

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/dockerconfigjson
metadata:
  name: image-pull-secret 
data:
  .dockerconfigjson: [base64 encoding taken from the ~/.docker/config.json]
```
Learn more about this feature in the Kubernetes documentation: [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
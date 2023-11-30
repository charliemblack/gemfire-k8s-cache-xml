# A demo to show GemFire cache.xml with K8s

In GemFire there is a concept of configuring GemFire VIA a xml file.   Since the cache.xml file needs to be present on the file system it proposes a challenge for managed solutions where end users don't own the solution.   Luckly VMware GemFire for Kubernetes is an designed with an open system concepts so there is a method to get items on the file system before GemFire starts.

In this demo we will be taking advantage of GemFire for K8s method for getting libraries onto the containers file system.  Libraries you might be wondering - well it is just a file and from an OS perspective its just bits on a disk.

So in some future release I would anticpate a change in the label of libraries to be more encompassing.    That might have to wait for a major since we would be changing the API.

# How does this work

First read about the libraries in the GemFire for K8s CRD documentation: https://docs.vmware.com/en/VMware-GemFire-for-Kubernetes/2.3/gf-k8s/crd.html

How above was implemented is it just copies the files from an image  into GemFire container in a standard location `/gemfire/extensions`.

Why an image - seems hard?  When we were designing this feature it was pretty hard forcast how customers were going to have thier infrastructure setup - did they have a blob store, file service, or any method to share data in a common way?  After many interviews we settled on using an container image because k8s uses container images.

You can see how that works here:

```
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
I have provided all the commands necessary in a [README](k8s/readme.md) to install the prerequists and how to install the GemFire Operator.   I have also included a [cluster.yml](k8s/cluster.yml) file to highlight how to use this method to copy the cache.xml file to the container and tell GemFire where to find it.


# So how do we build an image

In the [image](/image) directory we can see all the files that are needed along with the commands that I used to build and push the image to docker hub.   Of course your enterprise might have another container repository and you would want to call the image by your own name.   

The [GemFire Cache XML](/image/demo-cache.xml) is an example of a GemFire cache.xml

For the [Docker file](/image/Dockerfile) I extened busybox as my base image and copied the GemFire Cache XML into the image.

After that it is a simple build image and push (your exact commands may differ):

```
docker build -t charliemblack/gemfire-k8s-cache-xml .
docker push charliemblack/gemfire-k8s-cache-xml
```

# The secret.yml file

I am including this since it might not be widely known how to deploy secrets.   I personally like the method of 

```
apiVersion: v1
kind: Secret
type: kubernetes.io/dockerconfigjson
metadata:
  name: image-pull-secret 
data:
  .dockerconfigjson: [base 64 encoding taken from the ~/.docker/config.json ]
```
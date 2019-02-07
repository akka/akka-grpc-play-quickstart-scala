# Deploy on OpenShift

### Prerequisites

Install the following:

* [Docker](https://docs.docker.com/install/)
* [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* OpenShift's CLI: [`oc`](https://docs.openshift.com/container-platform/3.10/cli_reference/get_started_cli.html#installing-the-cli) (["Installing the CLI"](https://docs.openshift.com/container-platform/3.10/cli_reference/get_started_cli.html#installing-the-cli))
* [`kustomize`](https://github.com/kubernetes-sigs/kustomize) (v2.0.0+)
* [Sbt](https://www.scala-sbt.org/)


#### Preface

There are [multiple flavors](https://www.openshift.com/products?extIdCarryOver=true&sc_cid=701f2000001OH7iAAG) of `oc` and OpenShift. This guide was tested with:

```
$ oc version

oc v3.10.45
kubernetes v1.10.0+b81c8f8
features: Basic-Auth

Server https://centralpark.lightbend.com:443
openshift v3.10.45
kubernetes v1.10.0+b81c8f8
```

Where `centralpark.lightbend.com` is Lightbend's private [OpenShift Container Platform](https://www.openshift.com/products/container-platform/) cluster. This
guide uses `centralpark.lightbend.com` as an example, you will have to use your own OpenShift cluster or a local `minishift` instance. 

### Running

First, let's prepare a few environment variables to make things easier:

```
## obtain the token at the Console UI on you Openshift server
export TOKEN=<my-token>
export OPENSHIFT_SERVER=centralpark.lightbend.com
export DOCKER_REGISTRY_SERVER=docker-registry-default.centralpark.lightbend.com
export DOCKER_REGISTRY=$DOCKER_REGISTRY_SERVER/$OPENSHIFT_PROJECT
## Use a project name that will not clash with other deployments on the cluster
export OPENSHIFT_PROJECT=play-scala-grpc-example
export IMAGE=play-scala-grpc-example
export TAG=1.0-SNAPSHOT
```

Login to OpenShift from your terminal and create the OpenShift project:

```bash
oc login https://$OPENSHIFT_SERVER --token=$TOKEN
oc new-project $OPENSHIFT_PROJECT
```

Create the docker image of your application and push it to the image registry. In this guide we're using the Red Hat 
Container Registry in Lightbend's OpenShift cluster, CentralPark. You will have to use your own registry.

```bash
sbt docker:publishLocal

docker login -p $TOKEN -u unused $DOCKER_REGISTRY_SERVER
docker tag $IMAGE:$TAG $DOCKER_REGISTRY/$IMAGE:$TAG
docker push $DOCKER_REGISTRY/$IMAGE:$TAG

## The `kustomize` step uses a `kustomization.yml` prepared for $DOCKER_REGISTRY/$IMAGE:$TAG.
## You will have to create your own `deployment/overlays` folder (make a copy of
## `deployment/overlays/centralpark` and edit `kustomization.yml`)
kustomize build deployment/overlays/centralpark | oc apply -f -
```

Finally, verify the deployment completed successfully:

```bash
$ oc get all 
NAME                                                         READY     STATUS    RESTARTS   AGE
pod/play-scala-grpc-example-v1-0-snapshot-5b77bd9849-69wws   1/1       Running   0          16h
pod/play-scala-grpc-example-v1-0-snapshot-5b77bd9849-9p657   1/1       Running   0          16h

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/play-scala-grpc-example   ClusterIP   172.30.205.57   <none>        9000/TCP,9443/TCP   17h

NAME                                                    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/play-scala-grpc-example-v1-0-snapshot   2         2         2            2           17h

NAME                                                               DESIRED   CURRENT   READY     AGE
replicaset.apps/play-scala-grpc-example-v1-0-snapshot-5b77bd9849   2         2         2         16h

NAME                                                     DOCKER REPO                                                                                         TAGS           UPDATED
imagestream.image.openshift.io/play-scala-grpc-example   docker-registry-default.centralpark.lightbend.com/play-scala-grpc-example/play-scala-grpc-example   1.0-SNAPSHOT   17 hours ago

NAME                                             HOST/PORT               PATH      SERVICES                  PORT      TERMINATION   WILDCARD
route.route.openshift.io/play-scala-grpc-route   myservice.example.org             play-scala-grpc-example   http                    None
```

Test the application:

```bash
$ curl -H "Host: myservice.example.org" \
        http://$OPENSHIFT_PROJECT.$OPENSHIFT_SERVER  
Hello, Caplin!
```


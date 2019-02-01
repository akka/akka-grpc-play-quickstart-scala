
```
$ oc version

oc v3.10.45
kubernetes v1.10.0+b81c8f8
features: Basic-Auth

Server https://centralpark.lightbend.com:443
openshift v3.10.45
kubernetes v1.10.0+b81c8f8
```

Create the Openshift project:

```
oc login https://centralpark.lightbend.com --token=<my-token>
oc new-project play-scala-grpc-example
```

Create the docker image and push it to the Openshift project.

```
sbt docker:publishLocal
docker login -p VvXtXlfI6BGltR_PQ0La3_Yq9TAaWbs5qqjW6Qqpa0k  -u unused docker-registry-default.centralpark.lightbend.com 
docker tag play-scala-grpc-example:1.0-SNAPSHOT docker-registry-default.centralpark.lightbend.com/play-scala-grpc-example/play-scala-grpc-example:1.0-SNAPSHOT
docker push  docker-registry-default.centralpark.lightbend.com/play-scala-grpc-example/play-scala-grpc-example:1.0-SNAPSHOT
```

Test your deployment: 

```
curl  http://play-scala-grpc-example-route2-play-scala-grpc-example.centralpark.lightbend.com/ -H "Host: myservice.example.org"
nslookup play-scala-grpc-example-route2-play-scala-grpc-example.centralpark.lightbend.com
curl  http://52.22.136.67 -H "Host: myservice.example.org" 
```


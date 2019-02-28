# Principal_DevOps



As a Solutions Architect, I am exposed to every corner of the development ecosystem that exist. I move from the obvious places you would find a high level architect to places where I have been asked "why are you doing this job".  I love technology - and I love my weekends more.  Sticking to this role gives me the opportunity to remain at the technological forefront while hiking, camping, kayaking and golfing on the weekends.  Moving on ... in this role, I am asked a lot of devops question on a weekly basis. Those questions range from docker, to microservices, to k8s to openshift, from enterprise standinds to best practices and from contract management to training.  The job is actually amazing - and I truly love what I do. 

Recently a team reached out to me and asked for help cleaning up their registry - Enterpise Services has told them to clean up their docker image disk space or they would do it for them ... and here's the answer to that question ... 

## Purge images from docker registry V2

You will want to create two scripts, one that tests the concept and one that actually executes it and finally execute the cleanup docker command. 

To test the scripts below we will need to load a test image into the registry, then verify the push was successful:

```
docker pull alpine:3.7
docker tag alpine:3.7 localhost:5000/demo/apline:3.7
docker push localhost:5000/demo/apline:3.7
curl http://localhost:5000/v2/_catalog
```
 
###Script 1: 
test script that will call the docker registry and pull the sha for the image. There are three parts.  
1) the Server  
2) image  
3) version  
The script will fetch the sha and echo what the DELETE command would be   

```
#!/bin/bash

registry='localhost:5000'
name='demo/apline'
version='3.7'

echo "..............................."
echo "REGISTRY: $registry"
echo "IMAGE: $name"
echo "VERSION: $version"
echo "..............................."

sha=`curl -sSL -I \
     -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
http://${registry}/v2/${name}/manifests/${version} \
     | awk '$1 == "Docker-Content-Digest:" { print $2 }' \
     | tr -d $'\r'`
echo "curl -v -sSL -X DELETE \"http://${registry}/v2/${name}/manifests/${sha}\""
```

### Script 2   
The script  will call the docker server and find the sha for the image and then delete it.  As with the first script, there are three parts:
1) the Server
2) image
3) version
This script will fetch the sha and actually DELETE the image in the registry

```
#!/bin/bash

registry='localhost:5000'
name='demo/apline'
version='3.7'
echo "..............................."
echo "REGISTRY: $registry"
echo "IMAGE: $name"
echo "VERSION: $version"
echo "..............................."

curl -v -sSL -X DELETE "http://${registry}/v2/${name}/manifests/$( \
     curl -sSL -I \
     -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
     http://${registry}/v2/${name}/manifests/${version} \
     | awk '$1 == "Docker-Content-Digest:" { print $2 }' \
     | tr -d $'\r')"
```
 
and the final step is to execute the cleanup script to free deleted images (aka: empty the recycle bin) 

```
docker exec -it docker-registry bin/registry garbage-collect /etc/docker/registry/config.yml
```

# Overview

This repository is used to build distroless base images. The workflow will build the Wolfi base images. 

### Get the APKO image builder image
   ```
   docker pull cgr.dev/chainguard/apko
   ```
### Clone the Github repository
  ```
  git clone https://github.com/jasaz/build-distroless-base-images.git
  ```
### Switch to Wolfi-Base
   ``` 
   cd ../wolfi/jre/
   ```
### Create a tar file
   ``` 
   docker run -v "$PWD":/work cgr.dev/chainguard/apko build wolfi-jre17.yaml wolfi-jre17:one wolfi-jre17.tar
   ```
### Build an image from the tar file
   ``` 
   docker load < wolfi-jre17.tar
   ```
### Scan the base image for vulnerabilities
   ```
   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/Library/Caches:/root/.cache/ trivy-scanner wolfi-jre17:one-amd64
   ```
### Login to DockerHub (https://hub.docker.com) with Username and Password and create a personal access token token with read\write access
### Create a repository with name distroless-jre
### Tag the image to push it to DockerHub
    ```
    docker tag wolfi-jre17:one-amd64 github-user/distroless-jre:wolfi-jre17-one-amd64
    ```
### In the Console, enter the docker hub credentials
   ``` 
   docker login -u github-user
   ```
### Push the image to the repository
    ``` 
    docker push github-user/distroless-jre:wolfi-jre17-one-amd64
    ```

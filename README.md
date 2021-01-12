# Dockerfiles for FashionUnited
Collection of custom dockerfiles used for builds & running apps in production.

## Quick Deploy

submit the build to Cloud Build

```shell
gcloud auth configure-docker \
    europe-west1-docker.pkg.dev
gcloud config set project pkg-fuww
gcloud builds submit --config cloudbuild.yaml
```

## Artifact registry

```shell
gcloud artifacts docker images list europe-west1-docker.pkg.dev/pkg-fuww/fu

docker build -t europe-west1-docker.pkg.dev/pkg-fuww/fu/docsite:latest .

docker push europe-west1-docker.pkg.dev/pkg-fuww/fu/docsite:latest

gcloud artifacts docker tags add \
us-docker.pkg.dev/my-project/my-repo/my-image:iteration6-final \
us-docker.pkg.dev/my-project/my-repo/my-image:release-candidate
```


## Enabling Kaniko cache in your builds
You can enable Kaniko cache in a Docker build by replacing the cloud-builders/docker workers with kaniko-project/executor workers in your cloudbuild.yaml file as follows:

```
steps:
- name: 'gcr.io/kaniko-project/executor:latest'
  args:
  - --destination=gcr.io/$PROJECT_ID/image
  - --cache=true
  - --cache-ttl=XXh
```

or for all builds:

```
gcloud config set builds/use_kaniko True

gcloud config list builds/use_kaniko
```

## Build local

```shell
cd hugo/v0.74.3-extended
gcloud builds submit --tag=europe-west1-docker.pkg.dev/pkg-fuww/fu/hugo:v0.74.3-extended
```

old way, for reference:

cd hugo/v0.62.2-extended



docker build -t eu.gcr.io/kubernetes-164514/github.com/fashionunited/dockerfiles/hugo:v0.62.2-extended .
docker push eu.gcr.io/kubernetes-164514/github.com/fashionunited/dockerfiles/hugo:v0.62.2-extended



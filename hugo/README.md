# Thin Docker image with Hugo & S3cmd

## Updating the Docker image to latest Hugo version / Building Docker images manually

You might also need to update the node release keys, see the link in the dockerfile.

```sh
gcloud auth configure-docker \
    europe-west1-docker.pkg.dev

export CI_REGISTRY=europe-west1-docker.pkg.dev/pkg-fuww
docker build -t $CI_REGISTRY/fu/hugo:v0.121.0-ext -t $CI_REGISTRY/fu/hugo:latest ./docker
docker push $CI_REGISTRY/fu/hugo:v0.121.0-ext
docker push $CI_REGISTRY/fu/hugo:latest
```

Or, for gitlab registry

```sh
export CI_REGISTRY=registry.gitlab.com
fuww/landing/hugo # name on gitlab, differs from google
```

# GCP docker setup

# Repo

Official docs [here](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images)

- Use GCP Artifact Registry (replacement for the Container Registry, and more general)
- Initialise a repo for a project

Run the following command to create a new Docker repository named quickstart-docker-repo in the location us-central1 with the description "docker repository".

```
gcloud artifacts repositories create quickstart-docker-repo --repository-format=docker \
--location=us-central1 --description="Docker repository"
```

Run the following command to verify that your repository was created.

```
gcloud artifacts repositories list
```

For more information about Artifact Registry commands, run the command gcloud artifacts.

## Configure authentication

To set up authentication to Docker repositories in the region us-central1, run the following command:￼￼

```
gcloud auth configure-docker us-central1-docker.pkg.dev
```

## Tag for the repo

```
docker tag us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0 \
us-central1-docker.pkg.dev/<PROJECT>/quickstart-docker-repo/quickstart-image:tag1
```

replacing <PROJECT>

## Push

```
docker push us-central1-docker.pkg.dev/<PROJECT>/quickstart-docker-repo/quickstart-image:tag1
```

## Cleanup

To clean all images from the repo

```
gcloud artifacts repositories delete quickstart-docker-repo --location=us-central1
```

# docker-ci-cd-pipeline

This repository contains a Dockerized application and a CI/CD pipeline setup using GitLab CI/CD. The pipeline automates the building and pushing of Docker images to the GitLab Container Registry and deploys the application to staging and production environments.

## Repository Structure

- **Dockerfile**: Defines the Docker image for the application.
- **.github**: GitLab CI/CD pipeline configuration file.
- **app/**: Directory containing the application source code.
- **README.md**: Project documentation.

## Prerequisites

- GitLab account
- Docker installed on CI runners
- GitLab Container Registry enabled

## CI/CD Pipeline Overview

The pipeline consists of two main stages:

1. **Build**: Builds and pushes the Docker image to the GitLab Container Registry.
2. **Deploy**: Pulls the Docker image and deploys it to staging and production environments.

## Configuration

### .gitlab-ci.yml

The `ci-cd.yml` file defines the CI/CD pipeline:

```yaml
# ci-cd.yml

# Define stages for the pipeline
stages:
  - build
  - deploy

# Define variables for reuse
variables:
  IMAGE_TAG: "$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG"

# Jobs configuration
build:
  stage: build
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  tags:
    - docker

deploy_staging:
  stage: deploy
  script:
    - docker pull $IMAGE_TAG
    - docker run -d --name myapp -p 8080:80 $IMAGE_TAG
  environment:
    name: staging
    url: http://staging.example.com
  only:
    - master  # Deploy only on commits to master branch
  tags:
    - docker

deploy_production:
  stage: deploy
  script:
    - docker pull $IMAGE_TAG
    - docker run -d --name myapp -p 80:80 $IMAGE_TAG
  environment:
    name: production
    url: http://example.com
  only:
    - /^release-.*$/  # Deploy on tags matching 'release-*'
  tags:
    - docker
```

## Environment Variables
Make sure to set up the following environment variables in your GitLab project settings under CI/CD -> Variables:

- CI_REGISTRY: Your GitLab Container Registry URL.
- CI_REGISTRY_USER: Your GitLab username.
- CI_REGISTRY_PASSWORD: Your GitLab password or CI/CD token.

## Deployment
### Staging
The application is deployed to the staging environment on every commit to the master branch.

- URL: http://staging.example.com
### Production
The application is deployed to the production environment on tags matching the pattern release-*.

- URL: http://example.com
### Running Locally
To run the application locally using Docker:
```
docker build -t myapp .
docker run -d -p 8080:80 myapp



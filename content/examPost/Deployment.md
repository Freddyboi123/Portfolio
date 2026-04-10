---
date: 2026-04-10T21:28:42+02:00
draft: false
title: 'Deployment'
---

# Deployment
We have deployed our project to an Ubuntu server in a way that is both simple and easy to maintain. To achieve this, we implemented a CI/CD pipeline using GitHub Actions and Watchtower.

This setup allows the deployment process to be fully automated. As a result, no manual work is required after pushing changes to the repository.

The workflow is defined in a GitHub Actions configuration file:

```yml
name: GITHUB ACTIONS WORKFLOW
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v3
      -
        name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'corretto'
      -
        name: Build with Maven
        run: mvn --batch-mode --update-snapshots package
      -
        name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      -
        name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/vores:latest
```

This workflow ensures that every time code is pushed to the main branch, GitHub Actions will automatically:

Build the project using Maven
Create a Docker image
Push the image to Docker Hub

On the server side, Watchtower continuously monitors for new versions of the Docker image. When a new image is detected, it automatically pulls the updated version and restarts the container.

This results in a fully automated deployment pipeline where code changes are quickly and reliably reflected in the running application, without requiring manual intervention.




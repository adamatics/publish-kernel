# Publish Kernel Action

![GitHub release (latest by date)](https://img.shields.io/github/v/release/adamatics/publish-kernel)
![GitHub Workflow Status](https://img.shields.io/github/workflow/status/adamatics/publish-kernel/CI)

## Overview

`Publish Kernel Action` is a GitHub Action to automate the process of publishing Docker images as new kernels in AdaLab. This action encapsulates the necessary steps for logging into a Docker registry, pushing a Docker image, and registering the image with AdaLab.

## Features

- Log in to AdaLab Docker registry
- Push Docker image to the registry
- Install Python dependencies
- Register the Docker image as a new kernel in AdaLab

## Usage

To use this action in your workflow, add the following step:

```yaml
- name: Publish Kernel in AdaLab
  uses: adamatics/publish-kernel@v1
  with:
    kernel_name: my-kernel
    kernel_description: This is a kernel used in my group.
    harbor_registry_url: ${{ env.ADALAB_HARBOR_REGISTRY }}
    harbor_password: ${{ secrets.ADALAB_HARBOR_PASSWORD }}
    harbor_robot_user: ${{ env.ADALAB_HARBOR_USER }}
    image_name: ${{ github.repository }}
    image_tag: ${{ env.IMAGE_TAG }}
    adalab_user_token: ${{ secrets.ADALAB_USER_TOKEN }}
    adalab_username: ${{ env.ADALAB_USERNAME }}
    adalab_api_url: ${{ env.ADALAB_API_URL }}
```

## Inputs

* `kernel_name` (required): Name of your kernel
* `kernel_description` (required): Brief description of your kernel
* `harbor_registry_url` (required): AdaLab Harbor registry URL.
* `harbor_password` (required): Password for the AdaLab Harbor registry.
* `harbor_robot_user` (required): AdaLab Harbor registry robot user.
* `image_name` (required): Name of the Docker image.
* `image_tag` (required): Tag of the Docker image.
* `adalab_user_token` (required): User token for AdaLab API.
* `adalab_username` (required): Username for AdaLab API.
* `adalab_api_url` (required): AdaLab API URL.

## Example Workflow

```yaml
name: Build and Publish Kernel

on:
  push:
    tags:
      - 'v*'
  workflow_dispatch:

jobs:
  build:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest

    env:
      ADALAB_CLIENT_SECRET: ${{ secrets.ADALAB_CLIENT_SECRET }}
      ADALAB_API_URL: "https://[ADALAB_HOST].com/adaboard/api"
      ADALAB_HARBOR_REGISTRY: "harbor.[ADALAB_HOST].com"
      ADALAB_HARBOR_USER: "robot$github"
      ADALAB_USERNAME: "[USERNAME]"

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Determine image tag
        id: image_tag
        run: |
          if [[ "${GITHUB_REF}" == refs/tags/* ]]; then
            echo "IMAGE_TAG=${GITHUB_REF#refs/tags/}" >> $GITHUB_ENV
          else
            echo "IMAGE_TAG=${GITHUB_SHA}" >> $GITHUB_ENV
          fi

      - name: Build Docker image
        run: |
          IMAGE_NAME=${{ github.repository }}
          docker build -t $IMAGE_NAME:${{ env.IMAGE_TAG }} -f Containerfile .

      - name: Add AdaLab certificate to trusted certificates
        run: |
          echo "${{ secrets.ADALAB_CERT }}" > adalab.crt
          sudo cp adalab.crt /usr/local/share/ca-certificates/
          sudo update-ca-certificates

      - name: Publish Kernel in AdaLab
        uses: adamatics/publish-kernel@v1
        with:
          name: my-kernel
          kernel_description: This is a kernel used in my group.
          harbor_registry_url: ${{ env.ADALAB_HARBOR_REGISTRY }}
          harbor_password: ${{ secrets.ADALAB_HARBOR_PASSWORD }}
          harbor_robot_user: ${{ env.ADALAB_HARBOR_USER }}
          image_name: ${{ github.repository }}
          image_tag: ${{ env.IMAGE_TAG }}
          adalab_user_token: ${{ secrets.ADALAB_USER_TOKEN }}
          adalab_username: ${{ env.ADALAB_USERNAME }}
          adalab_api_url: ${{ env.ADALAB_API_URL }}
          
```

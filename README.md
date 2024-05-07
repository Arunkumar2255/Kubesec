
## Kubesec Security Scanning with GitHub Actions

KubeSec is a powerful tool for performing security checks on Kubernetes manifest files. Integrating KubeSec into CI/CD pipelines, particularly with GitHub Actions, enhances the security of Helm charts by automating the scanning process. This ensures that any vulnerabilities are caught early in the development cycle, making it an essential step for maintaining robust security practices.

## Overview

This project integrates Kubesec with GitHub Actions to automatically perform security scans on Kubernetes manifest files. By embedding Kubesec scans into your CI/CD pipeline, you can detect and mitigate potential security risks before they are deployed to production, ensuring your Kubernetes configurations are secure by design.


## Prerequisites

A GitHub account.


## Repository Structure

- `.github/workflows/`: Contains the GitHub Actions workflow files.
- `my-chart/`: Directory containing Kubernetes manifest files for scanning.
- `deployment.yaml`: Kubernetes deployment manifest.



## Workflow Steps

## Step 1: Clone the repository and switch to the directory:

$ git clone https://github.com/Arunkumar2255/kubesec.git
$ cd kubesec


## Step 2: Configure GitHub Actions

Create or modify the `kubesec.yml` file in `.github/workflows/` to set up the GitHub Actions workflow. This workflow triggers on pull requests and commits to the `main` branch:

    name: CI/CD Pipeline

    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main

    jobs:
      security-scan:
        # Specifies the runner environment to be Ubuntu latest version
        runs-on: ubuntu-latest

    steps:
      # Step 1: Checks out the repository code
      - name: Checkout code
        uses: actions/checkout@v2

      # Step 2: Install Kubesec for security scanning of Kubernetes manifests
      - name: Install Kubesec
        run: |
          curl -sSL https://github.com/controlplaneio/kubesec/releases/download/v2.11.2/kubesec_linux_amd64.tar.gz | tar xz -C /tmp
          sudo mv /tmp/kubesec /usr/local/bin

      # Step 3: Validate Kubernetes manifests with Kubesec
      - name: Validate Kubernetes manifests
        run: |
          kubesec scan ./my-chart/*.yaml | tee kubesec_output.json

      # Step 4: Check validation results and determine if any manifest fails the security scan
      - name: Check validation result
        run: |
          result=$(grep '"score"' kubesec_output.json | cut -d ':' -f2 | tr -d ', ')
          integer_result=$(echo "$result" | xargs) # Ensures that the result is trimmed and treated as an integer
          if [ $integer_result -gt 0 ]; then
            echo "All Kubernetes manifests passed validation."
          elif [ $integer_result -lt 0 ]; then
            echo "Some issues found in Kubernetes manifests. Fail the workflow."
            exit 1
          fi


## Step 3: Commit Changes and Create a Pull Request

Modify the Kubernetes Manifest

Locate the deployment.yaml file in your repository where the Kubernetes resources are defined. This file typically includes one or more configurations for deploying applications to your cluster.
Edit the deployment.yaml to change security-related configurations. For example, if the original pod manifest includes a container running in privileged mode, which is a known security risk, change the privileged setting to false:



    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
    app: nginx
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
          app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        securityContext:
         privileged: false  # Changed from true to false to enhance security
         runAsNonRoot: false


Then, commit and push your changes to a new branch:
Next, create a pull request on GitHub against the main branch

## Step 4: Monitor and Respond to GitHub Actions Output

Navigate to the 'Actions' tab in your GitHub repository to view the execution status and results of the workflow.
Review the output logs for any security warnings or errors reported by Kubesec.         
If the workflow identifies issues, modify your Kubernetes manifest files based on the recommendations provided.
Commit and push the changes to retrigger the workflow and ensure all security concerns are resolved.

## Step 5: Review and Merge the Pull Request

Once the KubeSec scan passes with no issues, review the pull request.

Merge the pull request into the main branch to apply the secure configurations.


## Troubleshooting

Workflow Failures: Verify that the GitHub Actions workflow is correctly set up and triggered.Check the Actions tab for error details and address any configuration issues.

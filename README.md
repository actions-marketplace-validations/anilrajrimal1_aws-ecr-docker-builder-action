
# AWS ECR Docker Builder Action

This GitHub Action automates the process of building and pushing Docker images to Amazon Elastic Container Registry (ECR). It simplifies the integration of Docker builds into your CI/CD pipeline, handling AWS credentials, image tagging, and pushing to the specified ECR repository.

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/anilrajrimal1/aws-ecr-docker-builder-action)](https://github.com/anilrajrimal1/aws-ecr-docker-builder-action/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- **Keyless AWS Authentication (OIDC)**: Uses GitHub Actions OIDC to assume an AWS IAM role — no long-lived credentials.
- **AWS ECR Integration**: Seamlessly build and push Docker images to Amazon ECR.
- **Flexible Configuration**: Allows custom Dockerfile paths, image tags, and build contexts.
- **Support for Branch-Specific Image Tags**: Automatically tags images with the branch name, or use a custom tag.

## Prerequisites

Before using this action, ensure that:

- You have an AWS account.
- You have an ECR repository created.
- You have created an IAM Role that:
   - Trusts GitHub’s OIDC provider
   - Has permission to push images to ECR
- Your GitHub workflow has:

```yaml
permissions:
  id-token: write
  contents: read
```

### AWS Setup (OIDC – One Time)
1. Create an OIDC provider in AWS IAM:
  - Provider URL: https://token.actions.githubusercontent.com
  - Audience: sts.amazonaws.com

2. Create an IAM Role with trust policy similar to:
```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
    },
    "StringLike": {
      "token.actions.githubusercontent.com:sub": "repo:<OWNER>/<REPO>:*"
    }
  }
}
```

3. Attach an IAM policy allowing ECR push/pull.

## Usage

### Basic Workflow (OIDC)

Here’s an example of how to use this action in your GitHub Actions workflow:

```yaml
name: Build and Push Image to ECR

on:
  push:
    branches:
      - master
  workflow_dispatch:


permissions:
  id-token: write
  contents: read

jobs:
  build:
    name: Build and Push
    runs-on: self-hosted

    steps:
      - name: Build and Push Docker Image
        uses: anilrajrimal1/aws-ecr-docker-builder-action@v1
        with:
          AWS_ROLE_ARN: ${{ vars.AWS_ROLE_ARN }}
          AWS_REGION: ${{ vars.AWS_REGION }}
          ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
          BRANCH_NAME: ${{ github.ref_name }}

```

### Inputs

This action accepts the following inputs:

| **Input**           | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `AWS_ROLE_ARN`      | IAM Role ARN to assume via GitHub OIDC.                                         |
| `AWS_REGION`        | AWS region where your ECR is located (e.g., `us-east-1`).                       |
| `ECR_REPOSITORY`    | The name of your ECR repository (e.g., `my-app-repo`).                          |
| `BRANCH_NAME`       | The branch name for tagging the image (default is set automatically).           |
| `DOCKERFILE_PATH`   | Path to your Dockerfile (default: `./Dockerfile`).                              |
| `BUILD_CONTEXT`     | Path to your build context (default: `.`).                                      |
| `APT_MODULES_FILE`  | Path to apt dependencies file.                                                  |
| `PIP_MODULES_FILE`  | Path to pip dependencies file.                                                  |
| `IMAGE_TAG`         | Custom tag for the image (optional). Default will be the branch name.           |

### Security Notes

- This action does not use AWS access keys
- Credentials are short-lived and issued per workflow run
- Recommended by both GitHub and AWS

## Contributing

We welcome contributions! Feel free to fork the repository and submit pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

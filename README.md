# GitHub Actions Repository

This repository contains custom GitHub Actions for common deployment tasks.

## Available Actions

### 1. Frontend S3 Build and Deploy

**Path:** `front-s3-build-deploy/action.yml`

This action builds a frontend application (Node.js based, using Yarn or NPM) and deploys the resulting artifacts to an AWS S3 bucket. It can also optionally invalidate an AWS CloudFront distribution.

**Inputs:**

| Input                      | Description                                           | Required | Default        |
| -------------------------- | ----------------------------------------------------- | -------- | -------------- |
| `aws_account_id`           | The AWS account ID.                                   | Yes      |                |
| `aws_region`               | The AWS region to deploy.                             | No       | `us-west-2`    |
| `iam_role`                 | The IAM role to assume for AWS operations.            | Yes      |                |
| `node_version`             | The version of Node.js to use for building.           | No       | `18`           |
| `s3_bucket`                | The S3 bucket to store the build artifacts.           | Yes      |                |
| `artifacts_folder`         | The folder containing the build artifacts (e.g., `build`, `dist`). | No       | `build`        |
| `including_folder_name`    | Whether to include the `artifacts_folder` name in the S3 path. `true` means `s3://[bucket]/[artifacts_folder]/...`, `false` means `s3://[bucket]/...`. | No       | `false`        |
| `cloudfront_distribution_id` | The CloudFront distribution ID to invalidate.         | No       | `''` (empty string, no invalidation) |
| `package_manager`          | The package manager to use (`yarn` or `npm`).         | No       | `yarn`         |
| `run_lint`                 | Whether to run the lint command (`yarn lint` or `npm run lint`). | No       | `false`        |
| `run_test`                 | Whether to run the test command (`yarn test` or `npm run test`). | No       | `false`        |
| `s3_copy`                  | *Deprecated.* This input is no longer used. The action always uses `aws s3 sync --delete`. | No       | `true`         |

**Outputs:**

This action does not define formal outputs. Its primary outcomes are:
*   Build artifacts are deployed to the specified S3 bucket.
*   If `cloudfront_distribution_id` is provided, the specified CloudFront distribution is invalidated.

**Usage Example:**

```yaml
name: Deploy Frontend

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to S3 and Invalidate CloudFront
        uses: ./.github/actions/front-s3-build-deploy # Assuming this is the path in your repo
        with:
          aws_account_id: '123456789012'
          aws_region: 'us-east-1'
          iam_role: 'MyFrontendDeployRole'
          node_version: '20'
          s3_bucket: 'my-frontend-bucket'
          artifacts_folder: 'dist'
          including_folder_name: 'false' # Deploy contents of 'dist' to bucket root
          cloudfront_distribution_id: 'E123ABCDEF45G'
          package_manager: 'npm'
          run_lint: 'true'
          run_test: 'true'
```

### 2. Lambda Deployment

**Path:** `lambda-deployment/action.yml`

This action packages a Lambda function (Python or Node.js) along with its dependencies, uploads the package to an S3 bucket, and updates the Lambda function's code.

**Inputs:**

| Input           | Description                                                | Required | Default    |
| --------------- | ---------------------------------------------------------- | -------- | ---------- |
| `aws_account_id`| The AWS account ID.                                        | Yes      |            |
| `aws_region`    | The AWS region where the Lambda function is located.       | Yes      |            |
| `aws_iam_role`  | The IAM role to assume for AWS operations.                 | Yes      |            |
| `lambda_bucket` | The S3 bucket to store the Lambda function code package.   | Yes      |            |
| `function_name` | The name of the Lambda function to update.                 | Yes      |            |
| `workdir`       | The working directory containing the Lambda function code and dependencies (e.g., `src`, `functions/my-function`). | Yes      |            |
| `language`      | The language of the Lambda function (`python` or `nodejs`). | No       | `python`   |
| `version`       | The language version to use (e.g., Python version like `3.12` or Node.js version like `18`). | No       | `3.12` (for Python), Node.js default is not explicitly set by this action but by `setup-node` if not provided. The default here is more for Python. For Node.js, `setup-node` default is `18.x` if `version` is not passed from this action to it. The default `3.12` in the action is primarily for Python. |

**Outputs:**

This action does not define formal outputs. Its primary outcomes are:
*   A zip package of the Lambda function and its dependencies is created.
*   The package is uploaded to the specified `lambda_bucket`.
*   The specified `function_name` is updated with the new code from S3 and published.

**Usage Example:**

```yaml
name: Deploy Python Lambda

on:
  push:
    branches:
      - main

jobs:
  deploy-lambda:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy Lambda Function
        uses: ./.github/actions/lambda-deployment # Assuming this is the path in your repo
        with:
          aws_account_id: '123456789012'
          aws_region: 'ap-southeast-2'
          aws_iam_role: 'MyLambdaDeployRole'
          lambda_bucket: 'my-lambda-functions-bucket'
          function_name: 'myApiFunction'
          workdir: './src/my-lambda' # Path to the lambda source code
          language: 'python'
          version: '3.11'
```

```yaml
name: Deploy Node.js Lambda

on:
  push:
    branches:
      - main

jobs:
  deploy-lambda:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy Lambda Function
        uses: ./.github/actions/lambda-deployment # Assuming this is the path in your repo
        with:
          aws_account_id: '123456789012'
          aws_region: 'eu-west-1'
          aws_iam_role: 'MyLambdaDeployRole'
          lambda_bucket: 'my-lambda-functions-bucket'
          function_name: 'myNodeJSFunction'
          workdir: './functions/my-service' # Path to the lambda source code
          language: 'nodejs'
          version: '18' # Node.js version
```

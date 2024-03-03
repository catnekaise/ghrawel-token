# ghrawel-token
Use this action to request a GitHub App Installation Access Token from a deployment of [catnekaise/ghrawel](https://github.com/catnekaise/ghrawel).

## Usage

```yaml
on:
  workflow_dispatch:
jobs:
  job1:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: "Authenticate"
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: "eu-west-1"
          role-to-assume: "arn:aws:iam::111111111111:role/role-name"

      - name: "Get Token"
        uses: catnekaise/ghrawel-token@v1
        id: token
        with:
          base-url: "https://abc123d4.execute-api.eu-west-1.amazonaws.com/dev"
          provider-name: "example-provider"
      - name: "Use Token Example"
        uses: actions/github-script@v7
        with:
          github-token: "${{ steps.token.outputs.token }}"
          script: |
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Test Issue',
              body: 'Hello from GitHub Actions'
            });
```

## Usage - Cognito Basic Auth and Get Token

```yaml
on:
  workflow_dispatch:
jobs:
  job1:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: "Authenticate and get token"
        uses: catnekaise/ghrawel-token@v1
        id: token
        with:
          base-url: "https://abc123d4.execute-api.eu-west-1.amazonaws.com/dev"
          auth-type: "cognito-identity-basic"
          provider-name: "example-provider"
          identity-pool-id: "eu-west-1:11111111-example"
          aws-account-id: "111111111111"
          aws-region: "eu-west-1"
          role-arn: "arn:aws:iam::111111111111:role/role-name"
      - name: "Use Token"
        env:
          TOKEN: "${{ steps.token.outputs.token }}"
        run: |
          echo "Do something with the token"
```

## Usage - Cognito Enhanced Auth and Get Token

```yaml
on:
  workflow_dispatch:
jobs:
  job1:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: "Authenticate and Get Token"
        uses: catnekaise/ghrawel-token@v1
        id: token
        with:
          base-url: "https://abc123d4.execute-api.eu-west-1.amazonaws.com/dev"
          auth-type: "cognito-identity-enhanced"
          provider-name: "example-provider"          
          cognito-identity-pool-id: "eu-west-1:11111111-example"
          aws-account-id: "111111111111"
          aws-region: "eu-west-1"
      - name: "Use Token"
        env:
          TOKEN: "${{ steps.token.outputs.token }}"
        run: |
          echo "Do something with the token"
```

## Required Inputs

| Input         | Example Value                                            |
|---------------|----------------------------------------------------------|
| base-url      | https://abc123d4.execute-api.eu-west-1.amazonaws.com/dev |
| provider-name | example-provider                                         |
| aws-region    | eu-west-1                                                |


## Auth Types

| Type                      | Requires Additional Input                                                 |
|---------------------------|---------------------------------------------------------------------------|
| environment (default)     | -                                                                         |
| provided                  | `aws-access-key-id`, `aws-secret-access-key`, `aws-session-token`         |
| cognito-identity-enhanced | `cognito-identity-pool-id`, `aws-account-id`                              |
| cognito-identity-basic    | `cognito-identity-pool-id`, `aws-account-id`, `cognito-identity-role-arn` |

## catnekaise/cognito-idpool-auth Action
When `auth-type` is either `cognito-identity-enhanced` or `cognito-identity-basic`. Additional documentation for this action can be found at [catnekaise/cognito-idpool-auth](https://github.com/catnekaise/cognito-idpool-auth).

## Input - Aws Region
The input `aws-region` is required unless either environment variable `AWS_REGION` or `AWS_DEFAULT_REGION` has been specified.

## Input - Owner
If no value is provided, `owner` will be set to current organization/user that is owner of the reppository running GitHub Actions.

## Input - Repo
If no value is provided, `repo` will be set to current repository name running GitHub Actions.

## Input - Base Url
Base URL shall either be `https://abc123d4.execute-api.eu-west-1.amazonaws.com/STAGE_NAME` if running a RestApi without a custom domain where then `STAGE_NAME` is replaced with value of the deployed `stage`. If a custom domain is used then that value should be used instead

## Input - Owner Endpoint
Currently, the input `owner-endpoint` only has to be used and set to `true` when requesting a token from a single repository when the token provider is configured as an owner endpoint. This will force the single repo to be set in a query parameter instead of being included as part of the path.

### Example
The request will be made to `https://abc123d4.execute-api.eu-west-1.amazonaws.com/dev/x/example-provider/catnekaise?repo=example-repo`

```yaml
on:
  workflow_dispatch:
jobs:
  job1:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: "Authenticate"
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: "eu-west-1"
          role-to-assume: "arn:aws:iam::111111111111:role/role-name"
          
      - name: "Get Token"
        uses: catnekaise/ghrawel-token@v1
        id: token
        with:
          base-url: "https://abc123d4.execute-api.eu-west-1.amazonaws.com/dev"
          provider-name: "example-provider"          
          owner: "catnekaise"
          repo: "example-repo"
          owner-endpoint: true
      - name: "Use Token"
        env:
          TOKEN: "${{ steps.token.outputs.token }}"
        run: |
          echo "Do something with the token"
```

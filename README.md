# Infra

[![CircleCI](https://circleci.com/gh/upstandfm/infra.svg?style=svg)](https://circleci.com/gh/upstandfm/infra)

Infrastructure resources.

## Table of contents

- [Workflow](#workflow)
- [Development](#development)
- [CI/CD](#cicd)
- [Output variables](#output-variables)

## Workflow

### GitOps

We use a [GitOps](https://github.com/danillouz/gitops-manifesto) style of workflow to drive operations. Here we version control infrastructure resources and use [AWS CloudFormation](https://aws.amazon.com/cloudformation/) templates as the single source of truth.

By doing this we get all the benefits of Git and pull request (PR) based workflows:

- history
- possibility to revert changes
- reviews
- comments
- possibility to link to issues, other PRs, etc.

This makes our system more transparent, discoverable, easier to operate, recoverable and observable.

### Usage

Add new infra resources on a branch and open a PR. When the PR is approved and merged, the CI/CD pipeline wil release the resources to production.

## Development

#### `npm run sls:debug`

Prints the `serverless.yaml` configuration.

## CI/CD

[CircleCI](https://circleci.com/gh/organizations/upstandfm) is used to:

- Lint the Serverless manifest.
- Deploy resources via [Serverless Framework](https://serverless.com).

### Serverless Framework

CircleCI requires a "Serverless Personal Access Key" to deploy resources. This is configured as an environment variable named `SERVERLESS_ACCESS_KEY` in the [CircleCI credentials context](https://circleci.com/gh/organizations/upstandfm/settings#contexts/400c57df-2f9a-46e3-88d8-dd598b88fd19).
The value of the access key can be found in the [1Password](https://1password.com/) "Upstand FM" vault under "Serverless access key for CircleCI".

The access key allows the Serverless CLI (used by CircleCI in the `lint` and `release` jobs) to authenticate with the Serverless Framework Dashboard.<br/>
Additionally, an [access role](https://serverless.com/framework/docs/dashboard/access-roles/) has been configured to help secure resource deployments on AWS, by enabling the Serverless Framework to issue _temporary_ AWS access keys to deploy resources. These keys are generated by Serverless Framework on every command, and the credentials expire after one hour.

> The Serverless Framework leverages AWS Security Token Service and the AssumeRole API to automate creating and usage of temporary credentials, so your developers can stay productive and work securely without doing this manually.

We also use a separate CloudFormation role to limit access during deployment, to only the required set of permissions needed by Serverless to deploy resources (i.e. _no_ `AdministratorAccess`). This is done by setting `provider.cfnRole` in the Serverless manifest.

# Output variables

The following [output variables](https://serverless.com/framework/docs/dashboard/output-variables/) are available in any service that uses the app `api`:

| Variable name      | Description                             | Usage                             |
| ------------------ | --------------------------------------- | --------------------------------- |
| `standupsTableArn` | The ARN of the standups DynamoDB Table. | `${state:infra.standupsTableArn}` |

All output variables can be viewed in the [Serverless Dashboard](https://dashboard.serverless.com/tenants/upstandfm/applications/api/services/infra/stage/prod/region/eu-central-1#service-overview=overview) under "Variables".

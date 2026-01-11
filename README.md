# aws-ec2-run-script ⛓️

**Current version**: `v1`

Reusable GitHub Action for executing scripts on AWS EC2 instances via
AWS Systems Manager (SSM). Designed for secure, repeatable CI and
infrastructure automation without SSH access.

## 🔭 Overview

This action executes shell scripts on EC2 instances using
AWS SSM Run Command (`AWS-RunShellScript`).

It is intended for use in CI pipelines and infrastructure workflows
where direct SSH access is undesirable or unavailable.

## ⚙️ How it works

1. GitHub Actions assumes an AWS IAM role (via OIDC or static credentials)
2. The action pulls a script from AWS S3 on the target EC2 instance(s) via SSM
3. The script is executed using `AWS-RunShellScript`
4. Command status and output are returned to the workflow

## ☑️ Requirements

### EC2 instance

- SSM Agent installed and running
- Instance IAM role with:
  - `AmazonSSMManagedInstanceCore`

### GitHub Actions

- AWS credentials configured using:
  - `aws-actions/configure-aws-credentials`
- IAM permissions:
  - `ssm:SendCommand`
  - `ssm:GetCommandInvocation`
  - `ec2:DescribeInstances`

## 🏗️ Usage

```yaml
- name: Run script on EC2
  uses: saraiTech/aws-ec2-run-script@v1
  with:
    instance-id: ${{ vars.INSTANCE_ID }}
    bucket-name: ${{ vars.SCRIPTS_BUCKET }}
    script-location: infra/deploy/rollout.sh
    env-vars-for-script: |
      GIT_BRANCH=${{ github.ref_name }}
      DEPLOY_ENV=prod
      SOME_FLAG=true
```

## ➡️ Inputs

| Name | Required | Default | Description |
|-----|---------|---------|-------------|
| `instance-id` | Yes | — | EC2 instance ID on which the script will be executed. |
| `bucket-name` | Yes | — | S3 bucket containing the script. |
| `script-location` | Yes | — | S3 key (path) of the script within the bucket. |
| `env-vars-for-script` | No | `""` | Newline-separated environment variables to export before executing the script. |
| `script-name` | No | `""` | Optional local filename override. Defaults to `basename(script-location)`. |
| `poll-interval-seconds` | No | `5` | Number of seconds between polling SSM for command status. |
| `timeout-seconds` | No | `1800` | Maximum number of seconds to wait before the action fails. |

## 🏡 Environment variables

Environment variables passed to the script **must** be provided in
`KEY=VALUE` notation, separated by newlines.

Example:

```yaml
env-vars-for-script: |
  APP_ENV=production
  LOG_LEVEL=info
  DEPLOY_VERSION=2026-01-11
```

## 📜 License

This project is licensed under the Apache License 2.0.
See the [LICENSE](LICENSE) file for details.

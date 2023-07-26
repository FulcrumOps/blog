---
slug: terraform-with-github-actions
title: "[WIP] Terraform with GitHub Actions"
date: 2023-07-21T01:00
authors: [pete]
tags: [terraform, github actions, ci/cd]
unlisted: true
draft: true
---

## Overview

I want to use [GitHub Actions](https://docs.github.com/en/actions) as my CI/CD pipeline for Infrastructure as Code (IaC) via Terraform.

I have some requirements:

1. Creating a pull request should instantiate a `terraform plan`.
1. Merging a pull request into the `main` branch should instantiate a `terraform apply`.
1. Sensitive credentials like `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` must be inaccessible to engineers who maintain the IaC.
1. The output from `terraform plan` and `terraform apply` must be easily accessible to engineers who maintaining the IaC.

## Reusable Workflows

You can [reuse workflows across repositories](https://docs.github.com/en/actions/using-workflows/reusing-workflows). GitHub
calls the two workflows a "caller" workflow and a "called" workflow. For this scenario, my "caller" workflow would be 
associated with my `infrastructure-live` repository where the IaC resides, and the "called" workflow would be associated with
my `infrastructure-pipeline` repository.

On a pull request, the `infrastructure-live` action would use an `infrastructure-pipline` action in order to do the `terraform plan`.

My hope was that the secrets stored in the called workflow in `infrastructure-pipeline` would be available to the job.

However, experimenting proved this not to be the case. You can pass secrets from the caller to the called workflow,
but you can't leverage the secrets in the called workflow. You can only leverage the caller's repository secrets or your organization secrets.
Sensitive credentials would therefore be accessible to the engineers maintaining the Infrastructure as Code.

## Triggering an Action and waiting for the results

The next possibility was to use a `repository_dispatch` to trigger the action in the desired repository. GitHub has
a canned [Action](https://github.com/marketplace/actions/repository-dispatch) that nicely encapsulates this. A triggered action
would run in the targeted repository, and therefore use its own secrets, avoiding the problem that we have with the reusable workflow.

There's a problem with this scenario.

Once the `infrastructure-pipeline` repository's action has been triggered, we have to wait until the `terraform plan` or `terraform apply`
action is done, and then query for the output that we need (either stored as an artifact or by pulling the logs).

This means that in order to run terraform, we have _two_ actions running the whole time, doubling our resource usage. As far as I can tell,
there isn't a way to add data to an action after the action has finished. Doing this synchronously seems to be the only way.

We could get around this a bit by just redirecting the engineers at the `infrastructure-pipeline` repo so that they can see the results,
but that doesn't seem very elegant. We could also trigger an action back into the `infrastructure-live` repository with the results,
but that means looking at a different action than the one initially created, so that doesn't seem very elegant, either.

## Triggering an Action and seeing the results in a PR comment

Instead of waiting synchronously for our `terraform` run to finish, we could have the `infrastructure-live` action call the GitHub API
(there are pre-build Actions that can help here) to post a comment back to the PR that triggered the initial flow to begin with.

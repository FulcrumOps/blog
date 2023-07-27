---
title: Using Terraform's import block to import existing AWS Infrastructure
date: 2023-07-27
author: Pete Emerson
tags: [aws, terraform, iac]
toc: false
---

# Terraform's `import` block

Terraform's [1.5.0](https://github.com/hashicorp/terraform/blob/v1.5/CHANGELOG.md#150-june-12-2023) release introduced `import` blocks to assist with importing
existing infrastructure into your Infrastructure as Code. Let's take it for a spin.

## AWS Identity Store User

I've been learning about AWS's Identity Center (was AWS SSO). So, I created an Identity Center user, group, and permissions set all in the AWS Console.
Once I had a good understanding about what was going on, the next step was to represent the infrastructure in Terraform. Previous to Terraform 1.5, I would
have tried to create the resource and used `terraform import` to pull it in, hoping that my mapping was all correct. In this example I'll focus on user creation,
but the same process applies to all of the resources.

## Create the import block

I created an import block for my `aws_identitystore_user` resource.

```
import {
  to = aws_identitystore_user.pete
  id = "d-9123de61a/2739c499-3012-1922-71ae-b4fc3c123fa4"
}
```

According to the [documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/identitystore_user), the format for the `id` is
`identity_store_id/user_id`. I pulled the `identity_store_id` and the `user_id` from the AWS Console. If I were doing this for a lot of users, I'd generate the
import block programmatically, using the AWS CLI to pull the data needed.

## Run `terraform plan` to generate code

With the import block in place, I ran `terraform` to generate the code:

```
terraform plan -generate-config-out=generated_resources.tf
```

This gave me a `generated_resources.tf` file:

```
# __generated__ by Terraform
# Please review these resources and move them into your main configuration files.

# __generated__ by Terraform from "d-9123de61a/2739c499-3012-1922-71ae-b4fc3c123fa4"
resource "aws_identitystore_user" "pete" {
  display_name       = "Pete Emerson"
  identity_store_id  = "d-9123de61a"
  locale             = null
  nickname           = null
  preferred_language = null
  profile_url        = null
  timezone           = null
  title              = null
  user_name          = "pete"
  user_type          = null
  emails {
    primary = true
    type    = "work"
    value   = "pete@fulcromops.com"
  }
  name {
    family_name      = "Emerson"
    formatted        = null
    given_name       = "Pete"
    honorific_prefix = null
    honorific_suffix = null
    middle_name      = null
  }
}
```

## Importing the real resource into the terraform state

Now that I've got the terraform code, it's a matter of importing the actual resource into the terraform state. I removed the `import` block and ran `terraform import`.
The resource id needed was in the import block, but it is also available in the generated code above.

```
terraform import aws_identitystore_user.pete d-9123de61a/2739c499-3012-1922-71ae-b4fc3c123fa4
```

Terraform successfully imported the resource, and then ran cleanly:

```
terraform plan
...
aws_identitystore_user.pete: Refreshing state... [id=d-9123de61a/2739c499-3012-1922-71ae-b4fc3c123fa4]
...
No changes. Your infrastructure matches the configuration.
```

To import existing infrastructure into your terraform state, there is a four-step shuffle:

1. Create the `import` block
1. Run `terraform plan` with the `-generate-config-out` flag
1. Remove the `import` block
1. Run `terraform import` with the id of the resource

I could have generated the `aws_identitystore_user` resource by hand and ran `terraform import` and `terraform plan` until it ran cleanly,
but the `import` block takes the guesswork out of the process. Well done, Hashicorp.

---
title: A demo of Shamir Sharing Secret in Golang
date: 2023-06-25
author: Pete Emerson
tags: [development, security, shamir sharing]
toc: false
---

# Shamir Secret Sharing Example

Sharing secret credentials among colleagues is simple. I can just store them in BitWarden or 1Password or whatever password manager I want, put them in a group that my colleagues have access to, and voila! The secret is shared.

What if I want to do something slightly different?

Suppose I want to have a secret shared between four people. To retrieve the secret, however, two of those four must provide information. Or suppose I have two groups of people, one being the C-suite and the other being a group of engineers. Is there a way to craft a secret so that any one of the executives can retrieve it, but it takes two engineers to do so?

There is.

From [Wikipdia](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing):

> Shamir's secret sharing (SSS) is an efficient secret sharing algorithm for distributing private information
(the "secret") among a group so that the secret cannot be revealed unless a quorum of the group acts together to 
pool their knowledge. To achieve this, the secret is mathematically divided into parts (the "shares") from which 
the secret can be reassembled only when a sufficient number of shares are combined. SSS has the property of 
information-theoretic security, meaning that even if an attacker steals some shares, it is impossible for the 
attacker to reconstruct the secret unless they have stolen the quorum number of shares.



[Hashicorp Vault](https://www.hashicorp.com/products/vault) implements Shamir's Secret Sharing. [My Golang code](https://github.com/FulcrumOps/shamir-sharing-example) leverages [their library](https://github.com/hashicorp/vault/tree/main/shamir).

At the heart of the code is the creation of the separate secrets that can be recombined later on:

```
shamir.Split([]byte(secret), parts, threshold)
```

It can be recombined with a call to the `Combine` function:

```
shamir.Combine([][]byte)
```

[My sample code](https://github.com/FulcrumOps/shamir-sharing-example) takes a hard-coded secret and splits it into five parts, three of which must be 

Let's see the code in action.

## Building the code

```
$ git clone https://github.com/FulcrumOps/shamir-sharing-example
$ cd shamir-sharing-example
$ go get
$ go build -o bin/shamir
```

## Running the code

```
$ bin/shamir
Secret: p455w0rdhunt3r2
Part 0: WDiR3HaLzp5qgaVfLfqutQ==
Part 1: 4x9stK+FQ1IVcgYlvHPJhA==
Part 2: Y+FE3nR0Ak30AWbscwdzGQ==
Part 3: isqNdercvWeMUdjKuRi1yw==
Part 4: eIOA6FClgg3bqD9YlpfjIQ==
[1] Enter any unique part of the original 5 parts: WDiR3HaLzp5qgaVfLfqutQ==
[2] Enter any unique part of the original 5 parts: 4x9stK+FQ1IVcgYlvHPJhA==
[3] Enter any unique part of the original 5 parts: eIOA6FClgg3bqD9YlpfjIQ==
Retreived secret: p455w0rdhunt3r2
```

In this way, a secret can be shared among individuals, but to recover the secret, three of five people must be willing to submit their piece of 
the puzzle.

Here's an example of how you might use this in your tech stack:

1. An AWS IAM account with `AdministratorAccess` is created.
1. The account has an Access Key associated with it.
1. The credentials are split up using Shamir's Secret Sharing algorithm and distributed.
1. When needed, the requisite number of people share their part, which is combined, revealing the access key credentials.

In this way, one person doesn't need to hold the keys to the kingdom, but may get access to those keys if others agree.

This could be taken a step further by having the secret grant access to an entry in AWS Secret Manager. Access to it gets monitored by CloudWatch, and ultimately trips an alarm (Pager Duty, et cetera). This creates a "break glass" scenario by which a few employees can get together and get the access they need, but the access is audited and reviewed, and no one employee can be a bad actor and do damage by themselves.

I'll write this up in a future blog post.

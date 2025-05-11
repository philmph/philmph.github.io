---
date: "2025-05-09T21:07:31+02:00"
modified: "2025-05-09T21:07:31+02:00"

draft: false

summary: "Using YAML as configuration source in OpenTofu"
title: "OpenTofu ❤️ YAML"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["beginner", "opentofu", "terraform", "yaml"]
---

## Preface

TODO OpenTofu instead of Terraform - working with Terraform
TODO _Note: The approach shown in this post is highly opinionated._
TODO Using sample scenario of DNS records
TODO Following HashiCorp Style Guide for Terraform
TODO Short Style Guide recap seen in this post
TODO This is part 1 of a three part series

## About

In the first part of this post I will show how to use YAML instead of `locals` or `.tfvars`. This is mainly useful when you have a module which is used to deploy a lot of the same `resource` with different configurations (e.g. DNS records). The second part shows how to define and validate the schema of a YAML file using methos built into OpenTofu.

The main benefits from this approach I have found are:

- Read- and Writeability
- Toolchain integration (automation to automation)
- Provides an easy-to-use abstraction layer for non automation / OpenTofu teams

Without yaml schema validation however, some downsides also arise:

- TODO

## Basics: Using YAML in OpenTofu

To use `.yaml` definition files we will be using the built in functions `file` and `yamldecode`:

```plaintext
.
├── config
│   └── input.yaml
├── main.tf
├── locals.tf
└── ...
```

The following content is added to `locals.tf`

```terraform
locals {
  yaml_directory = "config"
  yaml_filename = "input.yaml"
  yaml_data = yamldecode(file("${path.root}/${local.yaml_directory}/${local.yaml_filename}"))
}
```

## Problem Statement

## Solution

### Approach 1 - Using locals merge

### Approach 2 - Using

## References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250509_opentofu-<3-yaml)
- [GitHub Source (Cloudflare)](TODO)
- [GitHub Source (Terraform-Explorer)](https://github.com/philmph/Terraform-Explorer/tree/main/yaml-input-with-validation)

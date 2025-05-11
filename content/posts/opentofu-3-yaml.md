---
date: "2025-05-11T22:00:00+02:00" # TODO
modified: "2025-05-09T21:07:31+02:00" # TODO

draft: false

summary: "Using YAML as configuration source in OpenTofu"
title: "OpenTofu ❤️ YAML"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["beginner", "opentofu", "terraform", "yaml"]
---

## Before we begin

This post is part one of a two-part series. In this one, I'll show how to use YAML configurations in OpenTofu through a simple example. The second part will cover how to define and validate a YAML schema.

### Prerequisites

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed
- A [Cloudflare](https://www.cloudflare.com/) account with an existing DNS zone

## Introduction

I'll show how to use `.yaml` files instead of `locals` or `variable` definitions with `.tfvars` files to allow YAML-based, Git-tracked configuration. I've found this useful when working with modules that deploy lots of similar `resource` definitions with different parameters (e.g., DNS records).

The main benefits I've seen from this approach:

- YAML is well-known, easy to read and write
- Works well with automation toolchains
- Adds an abstraction layer and enables users to _just write YAML_

However, without YAML schema validation (which is covered in part two), there are some downsides to consider:

- No defaults (unlike `.tfvars` files with `variable` definitions)
- No proper quality or sanity checks

### Using YAML in OpenTofu

The following folder structure is generated

```plaintext
.
├── configuration
│   └── dns_records.yaml
├── locals.tf
├── main.tf
├── providers.tf
├── terraform.tf
└── variables.tf
```

To use `.yaml` definition files we will be using the built in functions `file` and `yamldecode`:

The following content is added to `locals.tf`

```terraform
locals {
  yaml_directory = "config"
  yaml_filename = "input.yaml"
  yaml_data = yamldecode(file("${path.root}/${local.yaml_directory}/${local.yaml_filename}"))
}
```

### `locals.tf` - Importing the YAML data

### `main.tf` - Using the YAML values in Resources

## Problems with this approach

### No Defaults

### No Validation

## References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250509_opentofu-<3-yaml)
- [Terraform Style Guide](TODO)

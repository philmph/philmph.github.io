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

## Before we begin

TODO Using sample scenario of DNS records
TODO This is part 1 of a three part series

## Prerequisite

This blog post doesn't explain how OpenTofu nor YAML works and expects readers to be able to use them.

Following along will also require:

- OpenTofu / Terraform installed
- A Cloudflare account with a DNS zone

## Introduction

In the first part of this post I will show how to use YAML instead of `locals` or `.tfvars`. This is mainly useful when you have a module which is used to deploy a lot of the same `resource` with different configurations (e.g. DNS records). The second part shows how to define and validate the schema of a YAML file using methos built into OpenTofu.

The main benefits from this approach I have found are:

- Read- and Writeability
- Toolchain integration (automation to automation)
- Provides an easy-to-use abstraction layer for non automation / OpenTofu teams

Without yaml schema validation however, some downsides also arise:

- TODO

## Using YAML in OpenTofu

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

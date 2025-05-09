---
date: "2025-05-09T21:07:31+02:00"
modified: "2025-05-09T21:07:31+02:00"

draft: true

summary: "Using YAML in OpenTofu including schema validation"
title: "OpenTofu ❤️ YAML"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["intermediate", "deep-dive", "automation", "opentofu", "terraform"]
---

## About

_Note: The approach shown in this post is highly opinionated._

In the first part of this post I will show how to use YAML instead of `locals` or `.tfvars`. This is mainly useful when you have a module which is used to deploy a lot of the same `resource` with different configurations (e.g. DNS records). The second part shows how to define and validate the schema of a YAML file using methos built into OpenTofu.

The main benefits from this approach I have found are:

- Read- and Writeability
- Toolchain integration (automation feeding automation)
- Provides an easy-to-use abstraction layer for non automation / OpenTofu teams

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

```terraform
# locals.tf

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

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250509_opentofu_yaml_schema_validation)
- [GitHub Source (Terraform-Explorer, unstructured)](https://github.com/philmph/Terraform-Explorer/tree/main/yaml-input-with-validation)

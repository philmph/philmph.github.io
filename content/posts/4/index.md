---
date: "2025-09-28T19:44:06+02:00"
modified: "2025-09-28T19:44:06+02:00"

draft: false

summary: ""
title: "Let's explore: nullable"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["intermediate", "lets-explore", "nullable", "opentofu", "terraform"]
---

## 🧭 Before we begin

This is going to be a short one. We'll be exploring the argument `nullable` in `variable` definitions.

My take is that `nullable` is often overlooked or even actively disregarded which can lead to unexpected results.

### 📋 Prerequisites

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed

Additionally:

- Knowledge about authoring and consuming reusable OpenTofu modules

## 🎯 Objective

We will explore `nullable` itself and also some edge cases (which are often used but seldom known).

## 🛠️ The various

### Introduction: What does `nullable` do?

First off, `nullable` is an argument within the `variable` block which determines if the `variable` can be `null`. The argument defaults to `true` for both Terraform and OpenTofu. Instead of duplicating docs I'd like to point out at this point, that the [the documentation](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values) by OpenTofu is a short and precise read.

### Module Setup

To keep this as simple as possible we will be creating and reusing an OpenTofu module which simply has four `variables`.

1. Not `nullable` with `default`
2. Not `nullable` without `default`
3. `nullable` with `default`
4. `nullable` without `default`

---

The following folder structure is used:

```plaintext
.
├── modules
│   └── simple
│       └── main.tf
│   └── ...
│       └── main.tf
└── main.tf
```

---

In `main.tf` of the module (`modules/simple/main.tf`), we add simple logic:

```terraform
variable "not_nullable_with_default" {
  default  = "default"
  nullable = false
  type     = string
}

variable "not_nullable_without_default" {
  nullable = false
  type     = string
}

variable "nullable_with_default" {
  default  = "default"
  nullable = true # This is the default
  type     = string
}

variable "nullable_without_default" {
  nullable = true # This is the default
  type     = string
}

output "all" {
  value = {
    not_nullable_with_default    = var.not_nullable_with_default
    not_nullable_without_default = var.not_nullable_without_default
    nullable_with_default        = var.nullable_with_default
    nullable_without_default     = var.nullable_without_default
  }
}
```

### Basic Usage

Let's now call the module using `null` as value:

```terraform
module "basic_usage" {
  source = "./modules/simple"

  not_nullable_with_default    = null
  not_nullable_without_default = null
  nullable_with_default        = null
  nullable_without_default     = null
}

output "basic_usage" {
  value = module.basic_usage
}
```

This will result in an error because of the variable `not_nullable_without_default` which does not allow `null` and requires a value to be provided (no `default`).

```bash
~ tofu plan
╷
│ Error: Required variable not set
│
│   on main.tf line 5, in module "basic_usage":
│    5:   not_nullable_without_default = null
│
│ The given value is not suitable for module.basic_usage.var.not_nullable_without_default defined at modules/simple/main.tf:7,1-40: required variable may not be set to null.
╵
```

---

Let's fix this and observe the result:

```terraform
...
  not_nullable_with_default    = null
  not_nullable_without_default = "module"
  nullable_with_default        = null
  nullable_without_default     = null

  # not_nullable_without_default = null
}
...
```

---

After fixing the value we do get a result:

```bash
~ tofu plan
Changes to Outputs:
  + basic_usage = {
      + all = {
          + not_nullable_with_default    = "default"
          + not_nullable_without_default = "module"
          + nullable_with_default        = null
          + nullable_without_default     = null
        }
    }

You can apply this plan to save these new output values to the OpenTofu state, without changing any real infrastructure.
```

But wait, does that mean that a `variable` which requires an value to be set allows `null`?! **Yes**.

A simple `variable` definition like

```terraform
variable "name" {
  description = "Name of the resource."
  type = string
}
```

While many `resource` definitions require names (e.g. Azure Resource Groups) this means that the consumer of the OpenTofu Module can provide `null` and a `plan` will be valid.

Note that if a `validation` block exists, this can also prevent the usage of `null`.

### `resource` vs. `module` blocks

It is important to understand that there is a huge difference in the usage of `null` when consuming OpenTofu Modules as in comparison to direclty using `null` within `resources` blocks.

TODO: Resource omitted
TODO: Module sendet `null` an downstream
TODO: Looping inputs (lists) with unassigned values -> sendet `null`

## 📚 References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250928-nullable)
- [GitHub Code Samples (Terraform-Explorer)](https://github.com/philmph/Terraform-Explorer/tree/main/input-methods-and-nullable)
- [OpenTofu Docs - Disallowing Null Input Values](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values)

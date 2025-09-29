---
date: "2025-09-28T19:44:06+02:00"
modified: "2025-09-28T19:44:06+02:00"

draft: false

summary: "TBD"
title: "0️⃣ Let's Explore: nullable"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["intermediate", "lets-explore", "nullable", "opentofu", "terraform"]
---

## 🧭 Before we begin

In this one, we'll be exploring the argument `nullable` in `variable` blocks.

My learning is that `nullable` is often overlooked or even actively disregarded. I have seen this lead to unexpected results often times while handling `nullable` is easy and takes little zero effort.

### 📋 Prerequisites

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed

Additionally:

- Knowledge about authoring and consuming reusable [OpenTofu "Published" Modules](https://opentofu.org/docs/language/modules/#published-modules)

## 🎯 Objective

We will explore `nullable` itself and look into some edge cases (which you can find in a lot of Modules).

## 🛠️ Let's Explore: `nullable`

### Introduction: What does `nullable` do?

First off, `nullable` is an argument within the `variable` block which determines if the `variable` value can be `null`. The argument defaults to `true` for both Terraform and OpenTofu. Instead of duplicating text I'd like to point out at this point, that the [the documentation](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values) by OpenTofu is a short and on point read.

### Module Setup

To keep this as simple as possible we will be creating and reusing an OpenTofu module which simply has four `variable` blocks:

1. Not `nullable` with `default`
2. Not `nullable` without `default`
3. `nullable` with `default`
4. `nullable` without `default`

---

The following folder structure is used:

```plaintext
.
├── modules
│   └── nullable
│       └── main.tf
└── main.tf
```

---

In `main.tf` of the Module (`modules/nullable/main.tf`), we add some logic:

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

---

### Calling the Module

Let's now call the Module using `null` as value:

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

---

This results in an error because the variable `not_nullable_without_default` does not allow `null` as value. It also requires a value to be provided (no `default`).

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

After the quick fix we get a result:

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

---

### Case (& Gotcha) 1: Providing `null` for Modules doesn't omit

> Not Nullable & with Default results in the `default` of the `variable`

It is important to understand that there is a difference in the usage of `null` when consuming Modules as in comparison to direclty using `null` within `resources` blocks:

---

Used in `module` blocks, providing `null` sends it as value to the `variable` of the Module even though it has `nullable` set to `false`! We did this by defining `not_nullable_with_default = null`. This does **not** omit the parameter and instead uses the `default` defined by the `variable` in the Module.

This is especially important to realize when looping over e.g. `list` inputs when `for_each` over a `module` block. If not handled otherwise, not assigned values also send `null` to the receiving Module.

---

In `resource` blocks, providing `null` as value omits the parameter (like it is not set):

```terraform
resource "azurerm_resource_group" "this" {
  name     = "example"
  location = null
}
```

In this example, the value for the parameter `location` is not defined. This will lead to an error.

---

### 2: Not Nullable & without Default

This case results in the value provided and does not accept `null`.

Even though this initially generated and error, there is nothing special about this case. The consumer of the Module is forced to provide a value for the parameter.

---

### 3: Nullable & with Default

This case results in the value provided and accepts `null`.

The `variable` allows `null` and we gave it `null`. It will simply be `null`. Resources within the Module will omit the parameters where the `variable` is used.

---

### 4: Nullable & without Default

This case results in the value provided and accepts `null`.

But wait... does this mean that a `variable` which requires an value allows `null`? **Yes... `null` is a value**.

A simple `variable` definition like

```terraform
variable "name" {
  description = "Name of the resource."
  type = string
}
```

per default allows `null` to be provided even though the `variable` is required. Even though is very commonly used in `resource` definitions which require names (e.g. Azure Resource Groups). This means that the consumer of the Module can provide `null` and a `plan` will be valid.

Note that a `validation` block can also prevent the usage of `null`.

Also note that **this is the default behavior!**

## 📚 References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250928-nullable)
- [GitHub Code Samples (Terraform-Explorer)](https://github.com/philmph/Terraform-Explorer/tree/main/input-methods-and-nullable)
- [OpenTofu Docs - Disallowing Null Input Values](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values)

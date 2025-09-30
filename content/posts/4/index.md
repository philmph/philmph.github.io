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

In this post, we'll be exploring the use of the **`nullable`** argument within Terraform `variable` blocks.

My point of view is that `nullable` is often overlooked or even actively disregarded. I've often seen this lead to **unexpected results**, even though handling `nullable` is simple and requires little effort.

### 📋 Prerequisites

To follow along, you'll need:

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed

Additionally a basic understanding of::

- Authoring and consuming reusable [Published Modules](https://opentofu.org/docs/language/modules/#published-modules)

## 🎯 Objective

We will explore the parameter **`nullable`** in `variable` blocks and look into some edge cases (which you will find in a lot of modules).

## 🛠️ Let's Explore: `nullable`

### Introduction: What does `nullable` do?

The `nullable` parameter exists within the `variable` block and determines if the value of the `variable` can be `null`. The parameter defaults to `true` for both Terraform and OpenTofu. For more information, [the documentation](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values) by OpenTofu is a good read.

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

In `main.tf` of the module (`modules/nullable/main.tf`), we add some logic:

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

---

This results in an error because the `variable` `not_nullable_without_default` does not allow `null` as value. It also requires a value to be provided (no `default`).

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

After the changes we get a result:

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

### 1: Not `nullable` & with `default`

Why is this even possible? The `variable` is not `nullable`?

It is important to recap a key information from [the documentation](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values) here:

> If `nullable` is `false` and the variable has a `default` value, then OpenTofu uses the default when a module input argument is `null`.

In my own words: When `null` is used as parameter value in `module` blocks, OpenTofu sends it as input to the `variable` of the module **even when it has `nullable` set to `false`!**

This is especially important to realize when looping over e.g. `list` inputs with `for_each` in a `module` block. If not handled otherwise, not assigned values also send `null` to the receiving Module.

---

In `resource` blocks, providing `null` as value **omits the parameter** (behaves like it is not set):

```terraform
resource "azurerm_resource_group" "this" {
  name     = "example"
  location = null
}
```

In this example, the value for the parameter `location` is not defined. This will lead to an error.

### 2: Not `nullable` & without `default`

Even though this initially generated and error, there is nothing special about this case. The consumer of the Module is forced to provide a value unequal to `null` for the parameter.

---

### 3: `nullable` & with `default`

The `variable` allows `null` and we assigned it `null`. It will simply be `null`.

Resources within the module will omit the parameters where the `variable` is used as input.

---

### 4: `nullable` & without `default`

Wait... does this mean that a `variable` which requires an value allows `null`? **Yes... `null` is a value**.

A simple `variable` definition like

```terraform
variable "name" {
  description = "Name of the resource."
  type = string
}
```

per default allows `null` to be provided even though the `variable` is required. Even though is very commonly used in `resource` definitions which require names (e.g. Azure Resource Groups). This means that the consumer of the Module can provide `null` and a `plan` will be valid.

Note that a `validation` block can also prevent the usage of `null`.

## The "problem" with `for_each`

Lets add some code to `main.tf`:

```terraform
# Loop example
variable "loop" {
  type = map(object({
    not_nullable_with_default    = optional(string)
    not_nullable_without_default = optional(string)
    nullable_with_default        = optional(string)
    nullable_without_default     = optional(string)
  }))
}

module "loop" {
  source = "./modules/nullable"

  for_each = var.loop

  not_nullable_with_default    = each.value.not_nullable_with_default
  not_nullable_without_default = each.value.not_nullable_without_default
  nullable_with_default        = each.value.nullable_with_default
  nullable_without_default     = each.value.nullable_without_default
}

output "loop" {
  value = module.loop
}
```

---

`terraform.tfvars`:

```hcl
loop = {
  "one" = {
    not_nullable_with_default    = "var"
    not_nullable_without_default = "var"
    nullable_with_default        = "var"
    nullable_without_default     = "var"
  },

  "two" = {
    not_nullable_without_default = "var"
  }
}
```

---

The interesting one is now the input `two`. The "user" did not provide values except for the one which is required (`null` not allwoed) exactly the same as actively calling the module with `null` (as shown above).

Here we will be sending `null` for not provided values to the module:

```bash
+ two = {
    + all = {
        + not_nullable_with_default    = "default"
        + not_nullable_without_default = "var"
        + nullable_with_default        = null
        + nullable_without_default     = null
      }
  }
```

If the module doesn't handle this it could break the run.

## My Usage Pattern

For advanced checks `validation` blocks is the right choice.

Otherwise, my key thoughts are two:

- Do my `resource` blocks allow ommiting a variable input?

An `description` might be an optional input to a `resource` while `name` is likely always mandatory and not allowed to be omitted.

- Do I want to always assign the `default` if the variable is not provided?

Expanding on the above: Maybe my module always wants a `description` of `Managed by OpenTofu`. This is achieven

This mostly happens by accident when root modules use loops with `for_each` with unassigned values as shown.

## 📚 References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250928-nullable)
- [GitHub Code Samples (Terraform-Explorer)](https://github.com/philmph/Terraform-Explorer/tree/main/input-methods-and-nullable)
- [OpenTofu Docs - Disallowing Null Input Values](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values)

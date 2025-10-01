---
date: "2025-09-30T22:09:00+02:00"
modified: "2025-09-30T22:09:00+02:00"

draft: false

summary: "Dive deep into the often-ignored variable argument nullable."
title: "0️⃣ Let's Explore: nullable"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["intermediate", "lets-explore", "nullable", "opentofu", "terraform"]
---

## 🧭 Before we begin

In this post, we'll be exploring the use of the `nullable` argument within Terraform `variable` blocks.

My point of view is that `nullable` is often overlooked or even actively disregarded. I've often seen this lead to unexpected results, even though handling `nullable` is simple and requires little effort.

### 📋 Prerequisites

To follow along, you'll need:

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed

Additionally a basic understanding of:

- Authoring and consuming reusable [Published Modules](https://opentofu.org/docs/language/modules/#published-modules)

## 🎯 Objective

We will explore the parameter `nullable` in `variable` blocks and look into some edge cases (which you will find in a lot of modules).

## 🛠️ Let's Explore: `nullable`

### 📝 Introduction: What does `nullable` do?

The `nullable` argument exists within the `variable` block and determines if the value of the variable can be `null`. It defaults to `true` in both Terraform and OpenTofu. For more information, [the OpenTofu documentation](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values) is a good resource.

### 🏗️ Repository Setup

To keep this as simple as possible, we'll create and reuse an OpenTofu module that contains four distinct `variable` blocks:

1. Not `nullable` with a `default` value
2. Not `nullable` without a `default` value
3. `nullable` with a `default` value
4. `nullable` without a `default` value

We will use a simple folder structure to organize our example:

```plaintext
.
├── modules
│   └── nullable
│       └── main.tf
└── main.tf
```

---

In the example module's `modules/nullable/main.tf` file, we define four scenarios and output the result:

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

### 🔈️ Calling the Module

Now, let's call the module, explicitly setting all four input variables to `null`:

```terraform
module "basic_usage" {
  source = "./modules/nullable"

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

Executing `tofu plan` results in an error because the variable `not_nullable_without_default` doesn't have a `default` value and explicitly disallows `null` as input:

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

Let's resolve this by giving `not_nullable_without_default` a value of `module`:

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

After the changes we now get a result:

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

### 1️⃣ Not `nullable` & with `default`

**Why is this even possible?** If the `variable` is not `nullable`, shouldn't passing `null` always fail validation?

The answer lies in a key piece of information from [the documentation](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values):

> If `nullable` is `false` and the variable has a `default` value, then OpenTofu uses the default when a module input argument is `null`.

This behavior is especially important to realize when looping over, for example, `list` inputs with `for_each` in a `module` block. If not handled, unassigned or missing values will also send `null` to the receiving module (we will see this later).

---

In `resource` blocks, providing `null` as a value generally **omits the argument** (behaves like it is not set):

```terraform
resource "azurerm_resource_group" "this" {
  name     = "example"
  location = null
}
```

is exactly the same as:

```terraform
resource "azurerm_resource_group" "this" {
  name     = "example"
}
```

In the first example, the value for the `location` argument is treated like it has not been defined (just as in the second example), which will lead to an error because `location` is a required argument for the `azurerm_resource_group` resource.

---

### 2️⃣ Not `nullable` & without `default`

Even though this scenario initially caused an error, there's nothing special about it. The rule is simply that the module consumer **must** provide a value other than `null` for the argument.

---

### 3️⃣ `nullable` & with `default`

Since the `variable` allows `null` and we assigned it `null`, its value will simply be `null`.

Consequently, any resources within the module that use this `variable` as an input will **omit those arguments**.

---

### 4️⃣ `nullable` & without `default`

Wait... does this mean a `variable` that requires a value allows `null`? **Yes... `null` is a value.**

A simple `variable` definition like:

```terraform
variable "name" {
  description = "Name of the resource."
  type = string
}
```

per default allows `null` to be provided, **even though the `variable` is required** (no `default` defined).

This is commonly used for resource definitions that strictly require names (e.g., Azure Resource Groups). This means the module consumer can provide `null`, and a `plan` might initially appear valid.

Note that a `validation` block can also prevent the use of `null`.

## 💣 The "problem" with `for_each`

Let's add a loop example to our root `main.tf`:

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

And define our input map in `terraform.tfvars`:

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

The interesting input is now key `"two"`. The "user" didn't provide values for the non-required attributes. This absence of a value inside the object, combined with the use of `optional` in the object type definition, is **exactly the same as actively calling the module with `null`**.

Therefore, we'll be sending `null` for all unprovided values to the module, resulting in the following planned output for instance `"two"`:

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

Takeaway: If the module doesn't handle `null` properly, this could lead to unexpected errors.

## 💭 My Usage Pattern

For advanced checks on variable content (like format or length), the `validation` block is the right choice.

Otherwise, my key thoughts narrow down to two questions:

---

1. Do my `resource` blocks allow **omitting** a variable input?

A `description` might be an optional argument to a `resource`, while `name` is likely always mandatory and shouldn't be omitted. (This dictates the use of `nullable = true` or `nullable = false`).

---

2. Do I want to always assign a `default` if the variable is not provided?

Expanding on the above: Maybe my module always wants a `description` of `"Managed by OpenTofu"`. This is achieved by setting the `default` value in addition to `nullable = false`.

This situation mostly occurs by accident when root modules use loops with `for_each` and pass **unassigned values**, as shown in the previous example.

## 🔚 Closing

Thanks once again for stopping by! This one got way longer then I intended it to be but I hope you had fun reading.

## 📚 References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250928-nullable)
- [GitHub Code Samples (Terraform-Explorer)](https://github.com/philmph/Terraform-Explorer/tree/main/input-methods-and-nullable)
- [OpenTofu Docs - Disallowing Null Input Values](https://opentofu.org/docs/language/values/variables/#disallowing-null-input-values)

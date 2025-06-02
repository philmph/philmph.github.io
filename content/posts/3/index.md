---
date: "2025-06-01T12:00:00+02:00" # TODO
modified: "2025-06-01T12:00:00+02:00" # TODO

draft: false

summary: "Defining a YAML schema and validating configuration against it"
title: "OpenTofu 🧪 YAML Schema & Validation"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["intermediate", "deep-dive", "opentofu", "terraform", "yaml"]
---

## 🧭 Before we begin

This post is part two of a two-part series. This time, I'll show how to use OpenTofu modules to define a YAML schema and validate configuration against it.

Part 1 of 2 can be found at [OpenTofu ❤️ YAML Configuration](/posts/2/).

### 📋 Prerequisites

Same setup as in part one:

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed
- A [Cloudflare](https://www.cloudflare.com/) account with an existing DNS zone

Additionally:

- Understanding how to use `.yaml` files as configuration source

## 🎯 Objective

We will define a YAML schema to enforce that configuration files follow a specific structure. This helps prevent anyone from adding structurally invalid inputs. It does not prevent incorrect values - though we can now also make use of `variable` based `default` values and `validation` blocks.

We'll reuse the file baseline established in part one and add logic on top of it. As before, we'll create [Cloudflare](https://www.cloudflare.com/) based DNS records.

### 📈 Benefits

Unlike in part one, we can now:

- Use built-in features from `variable` definitions when importing YAML configurations
  - Support `default` values and the `optional(...)` type in `object` definitions
  - Add `validation` blocks to enforce value rules

### 📉 Drawbacks

- Adds complexity by introducing a sub-module structure
- The YAML schema definition is not automatically added as `variable` documentation by `terraform-docs`

## 🛠️ Creating the YAML Schema and validating Configurations

To add a YAML schema definition, we'll simply make use of OpenTofu [`modules`](https://opentofu.org/docs/language/modules/). It's a simple solution that stays within built-in functionality and avoids `lookup` and `merge` shenanigans.

---

What the updated folder structure looks like:

```plaintext
.
├── configuration
│   └── dns_records.yaml
├── yaml-validation
│   └── dns-records
│       └── main.tf
├── main.tf
├── providers.tf
├── terraform.tf
├── variables.tf
└── yaml-validation.tf
```

---

It's worth pointing out that I'm not usual default conventions for this addition (e.g. only a single `main.tf` file in the sub-module `dns-records`). The YAML configuration import handling has been moved from `locals.tf` into the `yaml-validation.tf` file. In my opinion, this is a clean split that keeps the root module and YAML handling `locals` logic separate.

---

The logic previously found in `locals.tf` is now located in `yaml-validation.tf` and some additions have been made:

```terraform
locals {
  # Root folder where all configuration files are located
  configuration_path = "${path.root}/configuration"

  # Load all yaml configuration files
  yaml_configurations = { for i, o in fileset(local.configuration_path, "**/*.yaml") :
    o => yamldecode(file("${local.configuration_path}/${o}"))
  }

  # Valid
  configuration_name_dns_records = "dns_records.yaml"

  # TRYME: Will throw an error
  # configuration_name_dns_records = "dns_records_with_error.yaml"
}

module "yaml_validation_dns_records" {
  source = "./yaml-validation/dns-records"

  input = local.yaml_configurations[local.configuration_name_dns_records]
}
```

---

I've introduced a minor logic change by using [`fileset`](https://opentofu.org/docs/language/functions/fileset/) to filter all `**/*.yaml` files. This results in an `object` where the file name is the key (e.g. `dns-records.yaml`) and the decoded YAML configuration is the value.

The `module` call uses the new single file `main.tf` sub-module in `yaml-validation/dns-records` as the source. The value of the decoded YAML configuration is passed as the `input` variable.

---

The YAML schema logic in the `dns_records` sub-module's `main.tf` is simply a `variable` named `input` and an `output` named `output`:

```terraform
variable "input" {
  description = "dns_records schema validation"
  type = object({
    dns_records = list(
      object({
        name    = string
        type    = optional(string, "A")
        content = string
      })
    )
  })

  validation {
    condition = alltrue([
      for record in var.input.dns_records : can(regex("^([a-z0-9\\-]+\\.)+[a-z]+$", record.name))
    ])
    error_message = "All DNS record names must be valid domain names."
  }

  # Other useful validations...
}

output "output" {
  value       = var.input
  description = "Schema validated dns_records"
}
```

---

I've added both an `optional(...)` type to the `object` definition and a `validation` block. This means the `type` parameter is no longer required and the `name` parameter must be a valid DNS domain name (without the trailing `.` 😉).

Note that for the YAML validation sub-modules, I like to always name the `variable` `input` and the `output` `output`. These sub-modules don't have any other logic.

---

### ▶️ Applying the Configuration

The following YAML configuration file has been added at `configuration/dns_records.yaml` and contains:

```yaml
---
dns_records:
  - name: test3.pmaier.at
    # type: A
    content: "1.2.3.4"

  - name: test4.pmaier.at
    # type: A
    content: "4.3.2.1"
    omitted: "Really?"
```

---

This time, providing the parameter `type` is no longer mandatory because we've defined a default for it using `type = optional(string, "A")`.

When using `object` definitions in the `type` of a `variable`, additional keys can be added as long as the required ones are satisfied. However, they will be omitted after passing through the `yaml_validation_dns_records` module. This is standard behavior unrelated to YAML configurations.

---

Content of the decoded YAML configuration:

```bash
~ tofu console
> local.yaml_configurations[local.configuration_name_dns_records]
{
  "dns_records" = [
    {
      "content" = "1.2.3.4"
      "name" = "test3.pmaier.at"
    },
    {
      "content" = "4.3.2.1"
      "name" = "test4.pmaier.at"
      "omitted" = "Really?"
    },
  ]
}
```

---

Content of the `output` after passing through the `yaml_validation_dns_records` module:

```bash
~ tofu console
> module.yaml_validation_dns_records
{
  "output" = {
    "dns_records" = tolist([
      {
        "content" = "1.2.3.4"
        "name" = "test3.pmaier.at"
        "type" = "A"
      },
      {
        "content" = "4.3.2.1"
        "name" = "test4.pmaier.at"
        "type" = "A"
      },
    ])
  }
}
```

---

### 🔥 Let's break it

The repository includes a line you can uncomment to switch to the alternative input file `dns_records_with_errors.yaml` with errors:

```terraform
# TRYME: Will throw an error
# configuration_name_dns_records = "dns_records_with_error.yaml"
```

---

The conflicting file content:

```yaml
---
dns_records:
  - name: test3.pmaier.at
    # type: A
    # content: "1.2.3.4"

  - name: test4.pmaier.00at
    # type: A
    content: "1.2.3.4"
```

---

When running with this file, OpenTofu will complain that `content` is required for `element 1`. This is enforced by the `type` definition in the `input` variable:

```bash
~ tofu plan
╷
│ Error: Invalid value for input variable
│
│   on yaml-validation.tf line 20, in module "yaml_validation_dns_records":
│   20:   input = local.yaml_configurations[local.configuration_name_dns_records]
│
│ The given value is not suitable for module.yaml_validation_dns_records.var.input declared at yaml-validation/dns_records/main.tf:1,1-17: attribute "dns_records": element 1: attribute "content" is required.
╵
```

---

After fixing the first error, the `validation` block takes over and OpenTofu will complain again. That's because it doesn't allow anything other than lowercase letters (per regex `[a-z]+$`) for the final segment of `name`:

```bash
~ tofu plan
╷
│ Error: Invalid value for variable
│
│   on yaml-validation.tf line 20, in module "yaml_validation_dns_records":
│   20:   input = local.yaml_configurations[local.configuration_name_dns_records]
│     ├────────────────
│     │ var.input.dns_records is list of object with 3 elements
│
│ All DNS record names must be valid domain names.
│
│ This was checked by the validation rule at yaml-validation/dns_records/main.tf:13,3-13.
╵
```

## 🧠 Additional Thoughts

### 🪢 Tight Coupling

I've considered creating a `yaml-validation` sub-module which itself has another layer of sub-modules e.g. `dns-records`. This felt very cumbersome and wrong.

In my opinion, the root module should always handle the gathering of configuration data and helper modules like `yaml-validation/dns-records` should validate the input. If a sub-module handles configuration gathering, it introduces hardcoded code and therefore complexity because it's tightly bound to its root module.

Called modules by the root module deploying infrastructure (not the `yaml-validation` sub-modules) should not verify our YAML schema. I think this is an anti-pattern because it would reduce the reusability of modules and forces them into our YAML schema.

### 🗂 Multi File vs. `list` in Single File

It's possible to use multiple YAML configuration files instead of adding a `list`. For DNS records, I think this would've been overkill but for use-cases where a single item is, for example, 15-20 lines long, it can enhance readability.

## 🔚 Closing

Thanks once again for stopping by! I am not sure yet what will be next but I do have some ideas lined up 😄.

## 📚 References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250530_opentofu-yaml-schema-validation)
- [Part 1/2 (OpenTofu ❤️ YAML Configuration)](/posts/2/)

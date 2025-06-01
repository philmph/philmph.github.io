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

This post is part two of a two-part series. Here, I'll show how to use OpenTofu modules to define a YAML schema and validate configuration against it.

Part 1/2 can be found at [OpenTofu ❤️ YAML Configuration](/posts/2/).

### Prerequisites

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed
- A [Cloudflare](https://www.cloudflare.com/) account with an existing DNS zone
- Understanding how to use `.yaml` files as configuration source

## 🎯 Objective

TODO Update

A YAML schema will be defined to enforce configuration files to adhere to the specified content structure. This will prevent everyone from adding **structurally** wrong configuration inputs. It does not prevent wrong values (while we can now also utilize `variable` based `default` values and `validation` blocks).

We will reuse the file baseline established in part 1/2 and add logic on top to achieve that. Therefore, we will again create [Cloudflare](https://www.cloudflare.com/) based DNS records.

### Benefits

TODO Update

Unlike in part 1/1, we can now

- Add built-in features from `variable` definitions when using YAML configurations
  - Allows usage of `default` and `optional(...)` in `object`
  - Allows usage of `validation` blocks

### Drawbacks

TODO Update

- Adds code complexity by adding a sub-module structure

## 🛠️ Using YAML in OpenTofu

TODO Updates

Utilizing modules with a flat layout not following the default pathing modules/ ...

---

Layout

TODO

---

`yaml-validation.tf`

---

"module" `main.tf`

---

Errorcase (validation)

---

Successcase + implementation

## 🧠 Additional Thoughts

TODO Add thoughts about

### Layout

### Tight Coupling

I've considered creating a `yaml-validation` sub-module which itself has another layer of sub-modules e.g. `dns-records`. This felt very cumberstone because inputs are tightly coupled to a root module.

In my opinion, the root module should always handle the gathering of configuration data and helper modules like `yaml-validation/dns-records` should validate the input. If a sub-module takes care of the gathering, it will just add hardcoded code because it is bound to it's use-case. This is an anti-pattern for a module.

### Structural Enforcement

TODO Verify

Just like with complex `object` input `variable` definitions, additional keys can be added as long as the required ones are satisfied. They will just be ommited when the `yaml-validation` sub-modules verify the configuration files:

```yaml
dns_records:
  - name: test3.pmaier.at
    # type: A
    content: "1.2.3.4"
  - name: test4.pmaier.at
    # type: A
    content: "4.3.2.1"
    ommited: "Really?"
```

```bash
> opentofu console
> local.yaml_manifests[local.manifest_name_dns_records]
{
  "dns_records" = [
    {
      "content" = "1.2.3.4"
      "name" = "test3.pmaier.at"
    },
    {
      "content" = "4.3.2.1"
      "name" = "test4.pmaier.at"
      "ommited" = "Really?"
    },
  ]
}
```

```bash
> opentofu console
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

TODO Example

- Layout
- Multi-file option e.g. Subscriptions (link Nils?)
- Tightly coupled -> No need for overkill modularization
- Additional keys can be added but will be "thrown away" during module call (TODO Verify)

## 🔚 Closing

TODO Update

Thanks again for stoping by! I am not sure yet what will be the next follow-up. I have some ideas lined up 😄.

## 📚 References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250530_opentofu-yaml-schema-validation)
- [Part 1/2 (OpenTofu ❤️ YAML Configuration)](/posts/2/)

### KEEP

```bash
~/Documents/git/personal/naboo/Blog-Resources/posts/20250530_opentofu-yaml-schema-validation (post/3-add-defaults ✗) t plan
╷
│ Error: Invalid value for input variable
│
│   on yaml-validation.tf line 20, in module "yaml_validation_dns_records":
│   20:   input = local.yaml_manifests[local.manifest_name_dns_records]
│
│ The given value is not suitable for module.yaml_validation_dns_records.var.input declared at yaml-validation/dns_records/main.tf:1,1-17: attribute "dns_records": element 1: attribute "content" is required.
╵
~/Documents/git/personal/naboo/Blog-Resources/posts/20250530_opentofu-yaml-schema-validation (post/3-add-defaults ✗) t plan
╷
│ Error: Invalid value for variable
│
│   on yaml-validation.tf line 20, in module "yaml_validation_dns_records":
│   20:   input = local.yaml_manifests[local.manifest_name_dns_records]
│     ├────────────────
│     │ var.input.dns_records is list of object with 3 elements
│
│ All DNS record names must be valid domain names.
│
│ This was checked by the validation rule at yaml-validation/dns_records/main.tf:13,3-13.
```

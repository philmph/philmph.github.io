---
date: "2025-05-30T16:13:00+02:00" # TODO
modified: "2025-05-30T16:13:00+02:00" # TODO

draft: false

summary: "Creating and validating a YAML schema with OpenTofu"
title: "OpenTofu YAML Schema Validation"

params:
  author: "Philipp Maier"
categories: ["opentofu"]
tags: ["intermediate", "deep-dive", "opentofu", "terraform"]
---

## 🧭 Before we begin

This post is part two of a two-part series. Here, I'll show how to use OpenTofu modules to define and validate a YAML schema.

Part one can be found at [OpenTofu ❤️ YAML](/posts/2/).

### Prerequisites

- [OpenTofu](https://opentofu.org/docs/intro/install/) installed
- A [Cloudflare](https://www.cloudflare.com/) account with an existing DNS zone

## 🎯 Objective

TODO Update

We'll see how to use `.yaml` files instead of `locals` or `variable` definitions with `.tfvars` files to allow YAML-based, Git-tracked configuration. I've found this useful when working with modules that deploy lots of similar `resource` definitions with different values for their parameters (e.g., DNS records).

The sample shows how to create [Cloudflare](https://www.cloudflare.com/) based DNS records.

### Benefits

TODO Update

The main benefits I've seen from this approach:

- YAML is well-known, easy to read and write
- Works well with automation toolchains
- Adds an abstraction layer and enables users to _just write YAML_

### Drawbacks

TODO Update

However, without YAML schema validation (which is covered in part two), there are some downsides to consider:

- No defaults (unlike `variable` definitions with `.tfvars` files)
- No quality or sanity checks for the provided input

## 🧠 Thoughts

TODO Add thoughts about

- Layout
- Multi-file option e.g. Subscriptions (link Nils?)
- Tightly coupled -> No need for overkill modularization
- Additional keys can be added but will be "thrown away" during module call (TODO Verify)

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

## 🔚 Closing

TODO Update

Thanks again for stoping by! I am not sure yet what will be the next follow-up. I have some ideas lined up 😄.

## 📚 References

- [GitHub Code Samples (Blog-Resources)](https://github.com/philmph/Blog-Resources/tree/main/posts/20250530_opentofu-yaml-schema-validation)
- [Part One (OpenTofu ❤️ YAML)](/posts/2/)

---
date: "{{ .Date }}"
modified: "{{ .Date }}"

draft: true

summary: ""
title: '{{ replace .File.ContentBaseName "-" " " | title }}'

params:
  author: "Philipp Maier"
categories: [""]
tags: [""]
---

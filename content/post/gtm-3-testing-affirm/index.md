---
title: "draft"
draft: true
description: "draft"
date: 2025-04-06T00:00:00+02:00
image: cover.jpg
categories:
  - testing
tags:
  - testing
  - go
links:
  - title: Testing Module on GitHub
    description: Repository with my work associated with this post.
    website: https://github.com/ctx42/testing
    image: /img/github-logo.png
---

## Why Test the Test Code?

You might wonder: why bother testing the testing tools? It’s a fair question. The answer is trust. If my assertions or utilities have bugs - like `Equal` missing a type mismatch or `Panic` misreporting a crash - then every test relying on them could silently fail or give false positives. That’s a disaster for reliability. By writing tests for core and affirm (yes, even with the standard library’s verbosity), I ensure they’re rock-solid. It’s like calibrating a ruler before measuring: if the tool’s off, all your work’s suspect. Plus, these tests double as living examples, showing how the pieces fit together as I build out the module.

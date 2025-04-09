---
title: "Crafting Go Testing Module: Step 2 - Core"
description: In this second installment, we dive into the implementation of my Go Testing Module, starting with the foundational core and affirm packages to bootstrap readable tests for the package itself.
date: 2025-04-09
image: cover.jpg
categories:
  - testing
tags:
  - testing
  - go
  - gtm
links:
  - title: Testing Module on GitHub
    description: Repository with my work associated with this post.
    website: https://github.com/ctx42/testing
    image: /img/github-logo.png
---

Welcome back to my series on building a Go Testing Module from scratch! In the [first post](/p/crafting-go-testing-module-step-1-requirements), I shared my vision for this project: a dependency-free, developer-friendly toolkit designed to make writing tests in Go more readable and intuitive. I laid out the key requirements - like robust assertions, clear log messages, and mocking support - and explained why I’m tackling this, from my gripes with the standard library to my passion for great developer experience (DX). If you haven’t checked it out yet, I’d recommend giving it a read to see where this journey started. Now, in this second part, we’re rolling up our sleeves and diving into the actual work. I’ll walk you through where I’m starting the implementation, breaking down the first steps and the thinking behind them. Let’s get into it!

> Follow Testing Module development on [GitHub](https://github.com/ctx42/testing)

<!--more-->  

## The Chicken-and-Egg Problem

In the last post, I argued that test cases riddled with conditional statements - like `if want != have {}` - can be a readability headache in my opinion. But here’s the catch: how do we write clean, readable tests for a Testing Module when the assertion library we’re building doesn’t exist yet? It’s a classic chicken-and-egg dilemma, and like most challenges in software engineering, it calls for a compromise. My solution is to bootstrap the project with two very small internal packages: `core` and `affirm`. These will provide just enough functionality to write readable tests for the rest of the module, even if their own tests lean on the standard library for now.

The trade-off? The internal packages will rely on those clunky `if` statements in their tests. But since they’re small, self-contained, and tucked away as internal tools, I’m fine with that compromise. It’s a practical move to kick things off, freeing us to craft the broader, user-facing features - like the `assert` package - with the readability I’m chasing. Think of it as laying a foundation below ground: it’s not pretty, and no one will see it, but it’s sturdy and reliable to support the real beautiful construction above.

## The `core` Package

The core package is the bedrock of this module, a minimalist set of utilities that everything else will build on. It’s lean by design, with just couple of functions:

- `func IsNil(have any) bool`
- `func WillPanic(fn func()) (val any, stack string)`
- `func Same(want, have any) bool`

Let’s break these down and see why they’re essential.

### IsNil
Checking for nil in Go isn’t as simple as `want == nil`. Sure, that works for basic types like pointers or interfaces, but Go’s type system throws curveballs. An interface with a nil value and a non-nil type isn’t considered nil by a direct comparison - it’s a subtle gotcha that trips up even seasoned developers. `IsNil` handles this properly by inspecting the underlying value and type, giving us a reliable way to test for "nil-ness" across the board. In tests, this is critical: you need to know definitively whether something’s truly absent or just masquerading as nil. It will be a basis for all assertions that need to check for nil - like `assert.NoError`. For a deeper dive into the quirks of nil checking in Go, check out this great article:
    [Why Golang Nil Is Not Always Nil? Nil Explained](https://codefibershq.com/blog/golang-why-nil-is-not-always-nil). 

### WillPanic
Panics are a big deal in testing - you often want to verify that a function does panic under certain conditions (like invalid input) or doesn’t when it shouldn’t. Writing this check by hand with `recover()` is tedious and error-prone, so `WillPanic` wraps it up neatly. It runs the provided function, catches any panic, and returns what value was panicked, and a stack trace for context. This makes it a breeze to assert panic behavior - like ensuring a some call blows up as expected - without cluttering test code with boilerplate. Plus, that stack trace? Gold for debugging when things go sideways.

### Same
While equality checks (`==`) compare values, `Same` answers a different question: do `want` and `have` point to the exact same memory address? This is handy for testing pointer-heavy code, like when you’re verifying that a function returns an existing object rather than a new copy. It’s a niche but powerful tool, particularly useful when you’re verifying pointer behavior or ensuring object identity in tests. By using Go’s `any` type, it stays flexible enough to handle any pointer type we throw at it.

These three functions may seem straightforward, but they play a crucial role in driving the testing module’s development forward.

## The `affirm` Package

Next up is `affirm`, a stepping stone to the full-blown `assert` package I’ll build later. It’s deliberately minimalist, focusing on basic assertions with clean, readable log messages for a narrow set of common cases. Think of it as a starting point: it gives us usable assertions to work with while we build out the full version. Here’s what it includes:

- `func True(t *testing.T, have bool) bool`
- `func False(t *testing.T, have bool) bool`
- `func Equal[T comparable](t *testing.T, want, have T) bool`
- `func DeepEqual(t *testing.T, want, have any) bool`
- `func Nil(t *testing.T, have any) bool`
- `func NotNil(t *testing.T, have any) bool`
- `func Panic(t *testing.T, fn func()) *string`

Most of the functions return a bool to indicate success (allowing you to evaluate or act on results if needed) and logs a clear, formatted failure message using `t.Error` when things go wrong. For instance, `Equal` generates a log message when values don’t match, such as:

```text
expected values to be equal:
  want: 42
  have: 43
```

This keeps it simple but effective. Unlike the eventual `assert` package, `affirm` doesn’t handle fancy edge cases - like recursive types or `trails` mentioned in the previous article - yet. It’s just enough to make tests for the rest of the module readable, leaning on `core` for the heavy lifting (e.g., `IsNil` or `WillPanic`). The real `assert` package will take this further, adding the robustness and polish I outlined in the first post.

## Closing Thoughts
This post marks the first real step into coding my Go Testing Module. Starting with `core` and `affirm` packages solves the chicken-and-egg problem, giving me a foothold to write readable tests for the bigger features ahead - like the `assert` package, mocking, and golden files. It’s not glamorous, but it’s foundational, and I’m excited to see it take shape. In the next post, I’ll dive into how I plan to test the `affirm` package, ensuring it’s a solid stepping stone before we expand it further.

I’d love for you to check out the progress on Go Testing Module - head over to the project's [GitHub](https://github.com/ctx42/testing) to see the code and documentation in action. Feedback is always appreciated, whether it’s a suggestion, a bug report, or just your thoughts on the approach. Open an issue on GitHub or drop a message on [X](https://x.com/context42).

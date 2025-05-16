---
title: "Crafting Testing Module: Step 4 - Goldy and Must"
description: "Creating tools to tidy up test cases and build toward a fully featured testing module."
date: 2025-05-16
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

This is the fourth chapter in my blog series about building a Go Testing Module from scratch. If you’re new here, I recommend checking out the [previous post](/p/crafting-testing-module-step-3-testing-the-testers) for some context. In this post, I’m diving into two new packages I’ve created — `goldy` and `must`. These tools tackle problems you’ve probably faced in writing tests, making your test code cleaner, more readable, and a lot more enjoyable to work with. Let me walk you through how I built these packages to solve those familiar testing headaches.

## The `goldy` Package

You’ve likely faced the frustration of testing a function that generates a complex, multi-line string — like a JSON response or a formatted report. Hardcoding that string in your test code turns it into a cluttered mess that’s tough to read and even harder to maintain. I’ve been there, staring at a test file bloated with string literals. That’s why I created the `goldy` package, a lightweight tool that uses `golden files` to store expected outputs, keeping test code tidy and manageable.

### What’s a Golden File?

A _golden file_ is like a reference sheet for tests. It stores the expected output in a separate file, making it easy to compare against actual results. In `goldy`, I designed golden files to be clear and contextual. They include comments to explain their purpose, followed by the expected content. Here’s an example of a golden file, typically saved with a `.gld` extension:

```text
This comment explains the golden file's purpose.
It can span multiple lines for clarity.
---
Content line #1.
Content line #2.
```

The format is simple:

- Optional comment lines at the top to describe the file’s intent.
- A mandatory `---` separator line to mark the start of the content.
- The expected content (the "golden" output).

This structure keeps things organized and makes golden files easy to read and update.

### Using `goldy` in Tests

Here’s how you use `goldy` to test a function that generates a multi-line string:

```go
func TestSomething(t *testing.T) {
    // --- Given ---
    gld := goldy.Open(t, "testdata/test_case123.gld") 
    
    // --- When ---
    have := Something(123)
    
    // --- Then ---
    affirm.Equal(t, gld.String(), have) // Use content as string.
    affirm.Equal(t, gld.Bytes(), have) // Use content as bytes.
}
```

The `goldy.Open` function loads the golden file and returns a `Goldy` struct with fields I find useful during testing:

```go
// Goldy represents golden file.
type Goldy struct {
    Path    string // Path to the golden file.
    Comment string // Comments from the file.
    Content []byte // Content after the --- marker.
}
```

When the expected output changes, I can update the golden file by calling `gld.Save()`. This makes it simple to keep golden files in sync as the code evolves.

### Why `goldy` Matters

Building `goldy` was my answer to the chaos of managing large outputs in tests. It’s not just about cleaner code — it’s about making tests easier to maintain and understand. When a test fails, I can quickly check the golden file’s comments to grasp the context of the expected output. Storing large outputs in separate files also keeps test files focused on logic, not data.

## The `must` Package

You’ve probably dealt with the annoyance of repetitive error handling in a test setup section. I know I have — writing a setup section full of `if err != nil` checks that clutter the code and distract from the test’s purpose. It’s functional, but it feels like wading through noise. The `must` package is my solution: a set of helper functions that panic on errors, letting me focus on the test logic instead of boilerplate or setup code.

Here is an example of a test setup I used to write before `must`:

```go
func TestSomething(t *testing.T) {
    // --- Given ---
    wd, err := os.Getwd()
    if err != nil {
        t.Fatal(err)
    }

    fil, err := os.Open("/data")
    if err != nil {
        t.Fatal(err)
    }

    // --- When ---
    have := Something(wd, fil, 123)
    
    // --- Then ---
    affirm.Equal(t, true, have)
}
```

This is of course very simple example yet the `Given` section is noisy, with error checks drowning out the test’s purpose. It works, but it’s far from elegant, in my opinion. Here’s the same test using `must`:

```go
func TestSomething(t *testing.T) {
    // --- Given ---
    wd := must.Value(os.Getwd()) 
    fil := must.Value(os.Open("/data"))
    
    // --- When ---
    have := Something(wd, fil, 123)
    
    // --- Then ---
    affirm.Equal(t, true, have)
}
```

This is so much cleaner! The setup is concise, readable, and keeps the focus on preparing the test state. The `must` functions handle errors by panicking if something goes wrong, which Go’s testing framework catches and reports as a test failure. I find this approach perfect for test setup, where I want to fail fast if preconditions aren’t met.

### Exploring `must` Functions

I built the `must` package using generics to make it flexible and type-safe. Here are the key functions:

- `func Value[T any](val T, err error) T `- Returns the value or panics on error.
- `func Values[T, TT any](val0 T, val1 TT, err error) (T, TT)` - Returns two values or panics.
- `func Nil(err error)` - Panics if the error is non-nil.
- `func First[T any](s []T, err error) T` - Returns the first element of a slice or panics.
- `func Single[T any](s []T, err error) T` - Returns a single element or panics.

These functions work with any type, so I can use them in all sorts of testing scenarios. They’re especially useful in the `Given` section, where I’m setting up dependencies and want to avoid error-handling clutter.

### A Word of Caution

While `must` is great for test setup, I use it carefully. Panicking is a strong choice, so I save `must` for cases where an error means the test can’t go on. For assertions or checks in the `Then` section, I use `affirm` package to give clear failure messages.

## Closing Thoughts

Building `goldy` and `must` has been a solid step in making my Go Testing Module more practical and user-friendly. These packages come from my own struggles with test code, addressing the clutter and complexity you’ve probably faced too. They help me write tests that are not just functional but also clear and easy to maintain, letting me focus on catching bugs and ensuring code works as expected.

It’s hard to believe this is already the fourth post in the series, and I haven’t even started on the `assert` package yet. It’s a reminder of how many pieces need to come together before diving into assertions. Tools like `goldy` and `must` lay the groundwork, handling setup and comparison tasks so the `assert` package can focus on clear, expressive checks. Building a testing module is like assembling a puzzle — each piece, has to fit just right before the assertion package can take shape.

I’d love for you to check out the code for `goldy` and `must` in the [GitHub repository](https://github.com/ctx42/testing). Give them a try, poke around, and let me know what you think. If you have feedback or ideas, open an issue or drop a note on [X](https://x.com/context42). Your thoughts help improve this project, and I’m looking forward to the next steps.

Thanks for reading. Until the next post, happy testing!

---
title: "Crafting Go Testing Module: Step 1 - Requirements"
description: Post launches a series where I document my journey of developing a dependency-free Go Testing Module from the ground up.
date: 2025-04-07
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

This post marks the start of a series where I document my journey of developing a [Testing Module](https://github.com/ctx42/testing) for Go from the ground up. My goal is to share my thought process, my approach to problem-solving, and how I design software that’s not only functional but also a joy to use. A big part of this is focusing on developer experience (DX) - making sure the module feels intuitive and seamless for anyone who picks it up.

> Follow Testing Module development on [GitHub](https://github.com/ctx42/testing)

<!--more-->  

## Why?
I’ve always admired Go for its simplicity and readability - at least in most cases. However, I hold an unpopular opinion: tests written solely with the standard library can be tough to follow. All those `if` statements and manual checks clutter the code and make it harder to understand what’s being tested. For example, here’s a typical assertion using the standard library:

```go  
if want != have {  
    t.Errorf("expected %v, but got %v", want, have)
}  
```

It works, but it’s verbose and repetitive. In my view, tests become much more readable when you use assertion functions. They cut through the noise and let the intent shine through.

```go  
assert.Equal(t, want, have)  
```

That said, assertions are just one piece of the puzzle. A truly comprehensive testing module requires more than assertions. 

## Requirements
I began by asking myself: what should a robust, full-featured testing module contain? After some thought, I compiled a list of requirements. It’s an ambitious set of goals, but I’m certain I can create a module that delivers on every one. Let’s break down each requirement, digging into what it entails and my approach to making it happen.

### No External Dependencies
Without relying on someone else’s API or design choices, I can tailor the module exactly to my needs and preferences. This lets me craft a cohesive, intuitive interface that aligns with my vision, rather than adapting to the quirks of external tools. Another reason is simplicity. Dependency-free modules have fewer moving parts, making them easier to understand, debug, and maintain. External libraries often come with features you don’t use, adding unnecessary overhead. By writing a module from scratch, I can optimize it for my specific needs, keeping it lightweight and fast. It’s like crafting a custom tool instead of lugging around a bulky toolbox. Plus, it’s rewarding to build a tool tailored exactly to my vision, from features to docs. The catch is that it means writing a ton of code myself — code that itself will need rigorous testing to back it up.

### Well-Documented
Good documentation is as vital as clean, well-crafted code - it’s the bridge linking the module to its users. To encourage developers to adopt this module, I want to provide a clear, welcoming entry point that makes diving in a breeze. That begins with a detailed README filled with practical, real-world examples they can follow, complemented by thoroughly commented code with docstrings that explain how things work. I’ll also leverage Go’s documentation strengths by including runnable examples - interactive snippets users can execute directly in the Go playground or locally - to showcase the module in action. My goal is to simplify onboarding, transforming a potentially intimidating task into a smooth, enjoyable experience. When developers can quickly see the module’s value, test it hands-on, and understand its purpose, they’re much more likely to adopt it for their own projects.

### Asserting Composite and Recursive Types
Handling complex composite types-like `[]map[int]User` or other nested structures - shouldn’t be a stumbling block for this module. Equality assertions need to work seamlessly, whether you’re comparing a simple integer or a deeply recursive type with slices, maps, and structs intertwined. Personally, I can’t stand those massive, unwieldy diffs that spill out when composite values don’t match-they’re a headache to sift through. That’s why, when a comparison fails, this module won’t just dump a wall of text; it’ll provide a concise `trail` of differences that I find far more readable, pinpointing the exact spot where things went off track.

```
expected values to be equal:  
  trail: User.Addresses[0].Appartment  
   want: 42  
   have: 4
```

To me, these trails are a game-changer, making debugging faster and less frustrating.

The module will be opinionated by design, shaped by my own experiences and needs as a developer, so it’s built to tackle these pain points head-on. The result is a tool that doesn’t just work - it works the way I think it should, turning a messy process into something clear and manageable.

### Informative, Well-Formatted Log Messages
When a test fails, the error message shouldn’t leave developers guessing - it needs to tell the full story right away. That means delivering a neatly formatted log that clearly contrasts the expected value (`want`) with the actual value (`have`), so the mismatch is immediately obvious. A vague or cluttered message is a time sink, forcing developers to dig through code or rerun tests just to figure out what went wrong. On the other hand, a crisp, informative log acts like a spotlight, pointing straight to the problem and saving precious debugging time. My goal is to craft messages that not only highlight the failure but also provide enough context - like variable names or data types - to make fixing it as painless as possible. After all, a well-designed error message isn’t just a report; it’s a tool to keep workflows smooth and frustration low.

I'm thinking about messages like this:

```
// assert.Equal(42, byte(42))

expected values to be equal:  
      want: 42  
      have: 0x2a ('*')  
 want type: int  
 have type: uint8
```

### Basic Assertions
The `assert` package will be the backbone of this testing module, and it needs to cover at least 90% of the typical use cases developers run into day-to-day. In these early stages, I’m starting with the essentials - scenarios I encounter regularly in my own work, like equality checks (`assert.Equal`), nil checks (`assert.Nil`), and basic boolean assertions. These are the bread-and-butter tests that pop up everywhere, and getting them right sets a solid foundation. From there, I’ll build out the package iteratively, refining it based on real-world feedback and evolving needs. The goal isn’t just to check boxes but to make assertions so intuitive and reliable that developers can write tests quickly without second-guessing the tools. Over time, this focused approach will ensure the module grows into something that feels indispensable, handling the common cases effortlessly while leaving room for more advanced features down the line.

### Configurable
Flexibility isn’t just a nice-to-have - it’s a cornerstone of this module. I want developers to feel like they can mold it to suit their unique needs, whether that’s skipping specific fields during comparisons, registering custom checkers for their own types, or fine-tuning how those handy data trails are generated. Configuration isn’t about overwhelming users with options; it’s about handing them the reins to adapt the tool to their workflows seamlessly. For instance, if a struct has fields that don’t matter for a test, they can opt to ignore them without cluttering the output. Or if they’re working with a quirky custom type, they can plug in a tailored checker to handle it just right. To me, this kind of control is empowering - it transforms the module from a rigid framework into a versatile ally that bends to fit the way developers already work, rather than forcing them to adjust. And that flexibility sets the stage for extensibility, where developers can take this adaptability even further by building their own custom features on top.

### Extensible
If configurability is about fine-tuning the module, extensibility is about opening the door wide for customization. I want this module to serve as an open invitation—a foundation that empowers developers to craft their own test helpers and tools, tailored to their unique ideas and needs. To make that happen, I’m including a set of building blocks: helper functions, assertion templates, and value dumpers that turn any type into a readable format. With these, anyone can craft their own test utilities, whether it’s a niche assertion for a specific use case or a custom log formatter that fits their style. Extensibility isn’t just a feature - it’s a promise that the module can evolve alongside its users, growing more powerful as they add their own flair. My goal is to strike a balance: provide a solid foundation that’s ready to use out of the box, while leaving plenty of room for developers to extend it into something uniquely theirs.

### Mocking
A testing module isn’t truly complete without solid mocking support - it’s a non-negotiable for serious testing. Mocking lets developers isolate dependencies, swap out real implementations with controlled fakes, and dig into those tricky edge cases that are otherwise tough to replicate. I’ve been burned before by tests that flaked out because of tangled external systems, and I’m determined to spare others that frustration. That’s why I’m building a robust mocking library into this module, designed to make the process as painless as possible. Whether it’s stubbing out a database call or simulating a flaky network, the goal is to give developers a simple, reliable way to create mocks that just work. It’s about stripping away complexity so they can focus on testing their logic, not wrestling with setup - and that’s where a mock generator comes in to take the heavy lifting even further.

### Mock Generator
Hand-writing mocks might be manageable for a small test suite, but it quickly turns into a tedious, error-prone slog as projects grow. I’ve spent too many hours tweaking mock implementations by hand, only to miss a method or typo my way into a subtle bug-there’s got to be a better way. That’s why my module will include a mock generator that automatically creates mocks from interfaces, saving time and cutting down on human error. It’s a tool designed to churn out reliable, flexible, and ready-to-use mocks with minimal effort, so developers can focus on writing tests instead of wrestling with boilerplate. For me, this is a must-have to scale test coverage efficiently - whether you’re mocking one dependency or a dozen, it keeps the process smooth and mistake-free, building on the mocking support to make testing even more streamlined.

### Golden Files
Comparing long strings, multiline outputs, or JSON blobs manually is a hassle without golden files. Basic support for them will let developers set expected outputs and check results easily, catching regressions without extra effort. For example, when testing a function that generates a big configuration file, you can save the correct version as a golden file and compare it to the output each time. Or if you’re working on an API that returns complex JSON, a golden file ensures the structure and data stay consistent. It’s a simple way to make testing these cases faster and more reliable in day-to-day work.

### Test Kit
To wrap things up, I’m adding a `tstkit` package packed with utilities to make test writing faster and more readable. It’ll include handy helper functions to reduce repetitive code. The idea is to smooth out the rough edges of testing, making it less of a chore and more of a breeze. It’s all about giving developers practical shortcuts to keep their focus on the logic, not the busywork.

## Closing Thoughts
This is just the beginning of my journey to build a Go Testing Module from scratch, and I’m excited to see where it takes me. I’ve laid out the vision - readable tests, no dependencies, and a toolbox that grows with its users - and now it’s time to roll up my sleeves and make it real. Each step will bring its own challenges and trade-offs, and I’ll be sharing them all here as I go. Stick around for the ride! In the next post, we’ll dive into the first real hurdle: designing readable tests for the module itself when the assertion library we need doesn’t exist yet. How do we write clean assertions without the tools we’re building? It’s a classic chicken-and-egg puzzle, and I can’t wait to unpack how we’ll crack it.

I’d love for you to check out the progress on Go Testing Module - head over to the project's [GitHub](https://github.com/ctx42/testing) to see the code and documentation in action. Feedback is always appreciated, whether it’s a suggestion, a bug report, or just your thoughts on the approach. Open an issue on GitHub or drop a message on [X](https://x.com/context42).

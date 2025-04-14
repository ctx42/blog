---
title: "Crafting Testing Module: Step 3 - Testing the Testers"
description: "Exploring how to test the affirm package’s assertion helpers without triggering real failures."
date: 2025-04-14
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

In my [previous post](/p/crafting-go-testing-module-step-2-core), I introduced the `core` and `affirm` packages, laying the groundwork for a dependency-free Go testing module focused on readable, intuitive tests. Now, in this third installment, I’m diving into testing the `affirm` package - specifically, how to verify its assertion helpers. I’ll share my thought process, the challenges I faced, and how I solved them. Let’s get started!

> Follow Testing Module development on [GitHub](https://github.com/ctx42/testing)

<!--more-->

## Why Test Testing Tools?

You might ask: why go through the trouble of testing tools meant for testing? It’s a valid question, and the answer boils down to trust. If an assertion like does not work properly or misreports a crash, every test relying on it risks silent failures or false positives. That undermines reliability. By thoroughly testing, I ensure packages I create are dependable - it's like calibrating a ruler, you just created, before measuring with it. Plus, these tests serve as living examples, showing how the pieces fit together as the module grows.

## The Challenge of Testing Affirmations

Testing the `core` package was relatively straightforward, as its utilities (`IsNil`, `WillPanic`, `Same`) don’t interact directly with `*testing.T`. The `affirm` package, on the other hand, poses a challenge. Its helpers - such as `Equal`, `Nil`, and `Panic` - rely on `*testing.T` to log errors and failures, among other tasks, making them harder to test without affecting the actual test runner.

Consider the `Equal` helper:

```go
func Equal[T comparable](t *testing.T, want, have T) bool {
	t.Helper()
	if want != have {
		const format = "expected %T to be equal:\n  want: %#v\n  have: %#v"
		t.Errorf(format, want, want, have)
		return false
	}
	return true
}
```

Testing the success case is simple enough. We can use the test’s own `*testing.T` instance:

```go
func Test_Equal_success(t *testing.T) {
	// --- When ---
	have := Equal(t, 42, 42)
	
	// --- Then ---
	if !have {
		t.Error("expected true")
	}
}
```

This works because when `want` equals `have`, `Equal` doesn’t call `t.Errorf`, and the test passes quietly. But testing the failure case is problematic:

```go
func Test_Equal_failure(t *testing.T) {
	// --- When ---
	have := Equal(t, 42, 44)
	
	// --- Then ---
	if have {
		t.Error("expected false")
	}
}
```

Here’s the catch: when `want` doesn't equal `have`, `Equal` calls `t.Errorf`, which marks the test as failed in the real test runner. Even though `Equal` behaved correctly (logging the error and returning false), the test itself fails, masking the fact that we’re verifying the right behavior. This is a classic testing conundrum: how do we test a function that triggers test failures without failing the test?

## A Solution

At this stage, my Go testing module doesn’t yet have a full-fledged mocking library (that’s coming later - stay tuned!). To test `affirm` helpers, I needed a way to intercept `*testing.T` calls without affecting the actual test runner. That’s where the core package’s `T` interface and `Spy` struct come in. I introduced the `T` interface to capture a subset of `testing.TB` methods used by affirm helpers:

```go
// errors or failures.
type T interface {
	Error(args ...any)
	Errorf(format string, args ...any)
	Failed() bool
	Fatal(args ...any)
	Fatalf(format string, args ...any)
	Helper()
}
```

The `Spy` struct implements this interface, acting as a mock `*testing.T`:

```go
type Spy struct {
    HelperCalled     bool          // Tracks if Helper was called.
    ReportedError    bool          // Tracks if Error or Errorf was called.
    TriggeredFailure bool          // Tracks if Fatal or Fatalf was called.
    Messages         *bytes.Buffer // Log messages if set.
}
```

With `Spy`, I can track whether an assertion helper called `Error`, `Errorf`, or other methods, and inspect the logged messages when needed — all without failing the actual test.

### Adapting Affirm Helpers

To use `Spy`, I updated affirm helpers to accept `core.T` instead of `*testing.T`. For example, here’s the revised `Nil` helper:

```go
func Nil(t core.T, have any) bool {
	t.Helper()
	if core.IsNil(have) {
		return true
	}
	t.Errorf(expected, nil, have)
	return false
}
```

This change is subtle but powerful: by using the `core.T` interface, helpers become testable with `Spy` while still working with `*testing.T` in real tests (since `*testing.T` implements `core.T`).

### Writing Tests with Spy

Now, testing both success and failure cases becomes straightforward. Here’s how I test the `Nil` helper:

```go
func Test_Nil(t *testing.T) {
	t.Run("success", func(t *testing.T) {
		// --- Given ---
		spy := core.NewSpy()

		// --- When ---
		have := Nil(spy, nil)

		// --- Then ---
		if !have || spy.Failed() {
			t.Error("expected passed test")
		}
	})

	t.Run("error", func(t *testing.T) {
		// --- Given ---
		spy := core.NewSpy()
		err := errors.New("m0")

		// --- When ---
		have := Nil(spy, err)

		// --- Then ---
		if have || !spy.Failed() {
			t.Error("expected test error")
		}
		if !spy.ReportedError {
			t.Error("expected test error")
		}
	})
}
```

The success case verifies that `Nil` returns true for a nil input and doesn’t mark the test as failed. The failure case confirms `Nil` returns false for a non-nil input, and uses `Errorf` to mark the test as failed. The `Spy` lets me inspect the helper’s behavior - return values, error states, and log output - without interfering with the real test runner.

Another benefit is that helpers can call `t.Fatal` or `t.Fatalf` without causing the real test to panic or exit the goroutine, making `Spy` a versatile tool for testing all kinds of assertions.

### Trade-Offs and Reflections

Switching `affirm` helpers to use `core.T` instead of `*testing.T` is a pragmatic choice, but it’s not without trade-offs. On one hand, it makes testing possible without a full mocking framework. On the other, it slightly increases the package’s complexity by introducing an interface. I’m okay with this for now - it’s a small price for testability, and `T` is narrowly scoped to internal packages only.
Another consideration is `Spy`’s design. It’s minimal, capturing only the essentials (errors, failures, messages), but it’s flexible enough to grow if I add more assertion helpers later. For example, if I introduce helpers that call `Fatal` or `FailNow`, Spy’s `TriggeredFailure` field will already handle them.

## Closing Thoughts

Testing the `affirm` package pushed me to think creatively about mocking `*testing.T`, and I’m thrilled with how `Spy` and `T` turned out. They’re not just tools for testing `affirm` — they’re building blocks for a module that’s as reliable as it is intuitive. This step reinforced my belief that trustworthy tests start with trustworthy tools.

As always, I’d love for you to check out the progress on the [GitHub repository](https://github.com/ctx42/testing). The code and early docs are there - feel free to explore, test it out, or share feedback via an issue. You can also follow updates on [X](https://x.com/context42). Your thoughts help shape this journey, so don’t hold back!

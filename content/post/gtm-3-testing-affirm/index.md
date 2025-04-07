---
title: "Crafting Go Testing Module: Step 1 - Testing Affirmations"
description: "TODO"
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

In the last post two foundational packages `core` and `affirm` this time I will go over my thoughts and the way I'm going to approach the testing og the `affirm` package.

## Test Testing Tools

You might wonder: why bother testing the testing tools? It’s a fair question. The answer is trust. If my assertions or utilities have bugs - like `Equal` missing a type mismatch or `Panic` misreporting a crash - then every test relying on them could silently fail or give false positives. That’s a disaster for reliability. By writing tests for `core` and `affirm`, I ensure they’re rock-solid. It’s like calibrating a ruler before measuring: if the tool’s off, all your work’s suspect. Plus, these tests double as living examples, showing how the pieces fit together as I build out the module.

## Testing Affirmations

Having the reasoning out of he way lets focus on testing, which is not as easily in this case as you might have thought because we are testing functions which take `*testing.T` themselves.

```go
// Equal affirms two comparable types are equal. Returns true if it is,
// otherwise marks the test as failed, writes error message to the test log and
// returns false.
func Equal[T comparable](t *testing.T, want, have T) bool {
	t.Helper()
	if want != have {
		const format = "expected %T to be equal:\n" +
			"\twant: %#v\n" +
			"\thave: %#v"
		t.Errorf(format, want, want, have)
		return false
	}
	return true
}
```

Testing success cases is easy we can use the same instance of `t` our own test uses. 

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

The problem is when we want to test failure cases. If we try to do this:

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

It will fail the whole test even though the `Equal` function behaved exactly as we wanted it to. We must use different approach. 

Unfortunately in this stage of development of my Go Testing Module we have no way to mock the `testing.TB` interface which `*testing.T` implements. It will be possible but later - stay tuned. The compromise is to use our own instance of `*testing.T`. 
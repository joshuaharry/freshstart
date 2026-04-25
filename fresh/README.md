# Fresh

A safe, debuggable systems language.

## Motivation

First, let's take a brief survey of the non-garbage collected languages used for
low-level development today:

- C is a small, fast to compile, but wildly unsafe language that underpins
  virtually every piece of software on the planet. In some sense, we build
  *everything* on top of C. Writing a good C compiler is nontrivial but
  doable.
- C++ is like C, but enormously more complex.
- Zig is also small and fast to compile, but it still is not safe; it is also
  in beta and changing quite often. Zig features a lot of very sophisticated
  metaprogramming features like `comptime` that are wonderfully expressive but
  make the compiler quite difficult to write.
- Rust is virtually the only *memory safe* mainstream language (although there
  are some soundness bugs in the compiler), but it is also *wildly* complex and
  slow to compile. Moreover, async Rust is *notoriously* difficult to understand
  and debug.

What if we look for a language with different design goals? A language that is:

- Memory safe
- Secure
- Highly debuggable
- As fast to compile as is practical
- Interoperable with C (out of necessity)

## Values

When designing a new language, thinking through the language's values is a
useful exercise; watch 
[this talk](https://www.youtube.com/watch?v=Xhx970_JKX4) from Joyent
on values in programming language design.

If forced to rank order our values, here is roughly where I think we would
wind up:

- Safety
- Security
- Debuggability
- Performance
- Maintainability
- Interoperability
- Operability
- Measurability
- Transparency
- Composability
- Robustness
- Rigor
- Simplicity
- Stability
- Integrity
- Thoroughness
- Portability
- Resiliency
- Approachability
- Availability
- Compatibility
- Velocity
- Extensibility
- Expressiveness


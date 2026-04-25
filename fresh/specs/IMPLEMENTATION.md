# Implementation Strategy

We are going to build `fresh` in an *incredibly* incremental way;

1. Start with the *most minimal* version that can compile on a Mac.
   Get the frontend and backend of the compiler working together
   well.
2. Add support for x86_64 machines running Linux.
3. Add language features, repeating step 1 to 2.

We make sure we keep the core requirements the same as we grow the
language:

- Memory safety is nonoptional
- Security is nonoptional
- Debuggability, particularly DWARF, is nonoptional
- Our compiler must be fast
- We want to be able to call C

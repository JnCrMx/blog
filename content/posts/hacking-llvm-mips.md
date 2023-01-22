---
title: "Creating an LLVM MachineFunctionPass for a custom MIPS processor"
date: 2023-01-21T17:26:00+01:00
draft: false
---

When working with a custom MIPS processor, in this case one that was written as part of a university lecture, one of the challenges I faced is having a very limited instruction set.

In particular the following instructions are not supported, but can be manually "created" from other instructions:
- `SLTI` *(set less than immediate)* - can be easily built from an `ÀDDIU` and a `SLT`
- `BNEZ`/`BNE` *(branch not equals zero / branch not equal)* - a bit more tricky, but it is possible to use two `BEQ`s (more about that later)

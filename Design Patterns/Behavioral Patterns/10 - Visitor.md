---
tags: [design, pattern, behavioral, visitor]
---

## Overview

Lets you **add new operations** to existing object structures without modifying the structures themselves.

Uses double dispatch: the object "accepts" a visitor, which then performs the operation.

## When to Use

- When you have a stable class hierarchy but need to frequently add new operations on those classes (e.g., compiler AST nodes with multiple passes: type checking, code generation, optimization)

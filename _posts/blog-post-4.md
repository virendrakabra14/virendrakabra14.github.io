---
title: 'Post 4'
date: 2023-06-18
permalink: /posts/gsoc/lpython/post-4/
tags:
  - GSoC
  - LPython
---
<!--more-->
I worked on improving existing data structures

- 1932 - Support nested tuples.
  - For example, tuple[tuple[i32, f64], i32]
- 1940 - Support optional parameters in list index method.
  - Now the method signature is list.index(x[, start[, end]]), as in CPython.

In the coming week, I will continue to add and improve other data structure methods.

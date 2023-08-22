---
title: 'Post 5'
date: 2023-06-25
permalink: /posts/gsoc/lpython/post-5/
tags:
  - GSoC
  - LPython
---
<!--more-->
This week, I continued to improve list, tuple, and dictionary data structures.

Last week, I had worked on an improvement to support nested tuples. In some parts, I made use of raw memory allocation using new and delete. This was pointed in the PR comments.

I corrected this (PR 2018), using an allocator for memory management. This is used in general throughout the existing code as well, preventing memory leaks.

Further, I worked on adding some functionality to dictionaries. Issue 1881 is to support iteration through a dictionary as
```python
d: dict[i32, i32]
d = {1:2, 3:4, 5:6}
for k in d:
    print(k)
```

This code iterates over the dictionary keys. We discussed in our meet on how to implement this, and concluded that it would be good to first implement `dict.keys` and `dict.values` as in CPython. For now, we are not using views, but rather returning lists.

I implemented some of this (PR 2023), but it turns out that the keys list has some extra values. I will discuss on this further.

Next, I also started work on supporting list comparison (Issue 1832). At the moment, we only have equality comparison. I started with implementing less than operation (PR 2025). I am trying to abstract out code for various comparison operations, so that same code is not used repeatedly.

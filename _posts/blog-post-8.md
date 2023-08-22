---
title: 'Post 8'
date: 2023-07-16
permalink: /posts/gsoc/lpython/post-8/
tags:
  - GSoC
  - LPython
---
<!--more-->
I worked on adding more functionality to sets, and fixing some existing issues.

- Sets (#2122, continued)
  - After the initial implementation was completed last week, I added functions to add and remove elements from sets.
  - To add an element, its hash is computed, and we find an empty slot in the underlying array. This was done over the linear-probing implementation - if the spot corresponding to this hash is non-empty, a traversal is done starting from that position. Also, if the load factor crosses a threshold, a larger array is allocated, and all of the existing data is rehashed.
  - To remove an element, the mask value is simply set to a special value (3). This is called a *tombstone*, marking deletion. At later insertions, this spot can be reused. However, care has to be taken while reading elements - this element must not be read.
  - During the weekly meet with my mentor, we fixed an issue that wasn't reproducible for me locally, but was showing up on the workflows. This was occurring due to non-initialization of an interface member.

- Nested Dictionaries (branch nested_dict)
  - There is an existing issue (#1111) with the current dictionary implementation, which causes exceptions and segfaults when using nested dictionaries like `dict[i32, dict[i32, i32]]`. There was also an issue with assignment of empty dictionaries, i.e., `d: dict[i32, dict[i32, i32]] = {}`. This is now fixed (not merged yet). For the nesting, I found that the issue is with nested `DictItem` ASR nodes being present, which in turn try to read items that are not yet present. It seems that the solution is to have a `DictInsert` node at the innermost level, while keeping other nodes as is. I will try to fix this in the coming weeks.

- Set benchmarking
  - For the next week, I will also work on benchmarking our set implementation against C++ STL sets. We expect to perform (at least) at par with these, but if we find issues with some types (e.g. set of strings), we will also add the separate chaining implementation, as done for dictionaries.

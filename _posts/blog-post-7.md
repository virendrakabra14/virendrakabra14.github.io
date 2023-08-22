---
title: 'Post 7'
date: 2023-07-09
permalink: /posts/gsoc/lpython/post-7/
tags:
  - GSoC
  - LPython
---
<!--more-->
I worked on introducing sets and adding functionality to existing data structures.

- Set (#2122)
  - We already had initial implementation for dictionaries. Sets are similar, but unlike dictionaries, do not have key-value pairs. As with dictionaries, an abstract set interface is created, so we can use different collision-resolution techniques. There is some duplication of code such as hash computation, but for now we focus on providing basic functionality for set.
  - I used linear-probing collision-resolution, where we find the next empty spot starting from the hash position. While deleting an element, we mark it with a special marker (tombstone). This helps in ignoring it in further reads, and indicates that elements with the same hash can be present at later positions.
  - Further, I implemented functions to write items into the set. This involved resolving collisions for reading and writing, and also for rehashing the entire set while increasing the underlying list size. This technique is not effective for string elements, as in that case we just compute hashes of the character-array pointers. We will use separate chaining for this in the coming weeks.

Next, I worked on functions for deep-copying sets, and retrieving its length.

- Dictionary keys and values (#2023)
  - I corrected earlier functions to obtain lists of keys and values, when linear probing is used. This involved using the key-mask to identify whether elements are set or not.
  - Directly testing this did not work, as CPython does not return list items for these functions. Rather, it returns a view, with limited functionality; for example, we cannot index a view. Thus, I tried copying the results into a list, but it turns out that LPython does not fully support nested iterables with dictionaries yet. I will work on this in the next week.

- List comparison (#2025)
  - Earlier, I had implemented the less than operation for lists and tuples. I extended this to other comparison operations, and used LLVM's CreateCmp instructions that allow specifying a predicate. This helps prevent unnecessary replication of code, and we just switch the predicate using an overload_id.

---

Next week, I will continue adding more functions to sets, and try to resolve the nested dictionary issue.

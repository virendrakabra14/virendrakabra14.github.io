---
title: 'Post 11'
date: 2023-08-06
permalink: /posts/gsoc/lpython/post-11/
tags:
  - GSoC
  - LPython
---
<!--more-->
## Reserve function ([#2238](https://github.com/lcompilers/lpython/pull/2238))

There was an interesting observation with one of the C++ benchmarks: it sped up significantly with the use of `std::vector<T>::reserve`. To test this with LPython, we added a similar function `reserve(list, n)` that reserves space of `n` list elements. Surprisingly, the C++ gains could not be replicated with LPython, even after this addition.

This function is not available in CPython; type information is required for reserving space. For CPython tests, a function modifying the list to be `[None] * n` is used.

## Nested dictionaries ([#2253](https://github.com/lcompilers/lpython/pull/2253))

We only had support for single-level dictionaries:
```python
d: dict[i32, i32] = {1: -1}
d[3] = -3
```

We added support to handle nested dictionaries:
```python
d: dict[i32, dict[i32, i32]] = {1: {2: 4, 3: 9}, 7: {8: 64, 9: 81}}
d[3] = {4: 16, 5: 25}
d[1][10] = 100
```

This involved modification of a part of AST to ASR conversion. In particular, `Subscript` nodes with `Dict` names created `DictItem` objects. However, this leads to the following (partial ASR) for `d[1][10][100] = 1000`
```bash
(DictItem
    (DictItem
        (DictItem
            (Var 3 d)
            (IntegerConstant 1 (Integer 4))
            ()
            (Dict
                (Integer 4)
                (Dict
                    (Integer 4)
                    (Integer 4)
                )
            )
            ()
        )
        (IntegerConstant 10 (Integer 4))
        ()
        (Dict
            (Integer 4)
            (Integer 4)
        )
        ()
    )
    (IntegerConstant 100 (Integer 4))
    (IntegerConstant 1000 (Integer 4))
)
```
So `10` is searched as a key in `d[1][10]`, leading to a `KeyError`.

What we require is a nesting like
```bash
DictInsert      # here's the change
    DictItem
        ...
            DictItem
```
That is, `DictInsert` at the outer-most level.

This was done using a recursive function, that keeps track of the recursion level. Further, there were some issues with assignment of empty dictionaries, which was fixed.

Adding support of nested dictionaries also led to the following interesting issue
```python
d: dict[i32, dict[i32, i32]] = {1: {2: 3}, 4: {5: 6}}
d[1] = d[4]
print(len(d[1]))    # prints 0
```
This is because the initial assignment of `d` leads to its `capacity` and `occupancy` both equalling `2`. Now the assignment `d[1] = d[2]` triggering rehashing of the dictionary (as the current `occupancy / capacity` ratio breaches a pre-set threshold). This in turn invalidates the value pointer (to `d[2]`). This was fixed by adding a possibility of rehashing *after* writing to dictionaries.

A small improvement was also made to the way hashes are computed for `bool` keys: We take the hash modulo capacity.

## Next

I observed that there are still issues with the existing `dict` implementation. For example, popping keys does not cause `KeyError`s even though the same key has been popped right before. Similar issues exist with insertion of elements, especially with the Separate Chaining implementation. For example,
```python
d2: dict[str, i32] = {'a': 1, 'a': 2}
print(len(d2))      # 2
```

I will try to fix these in the coming week.

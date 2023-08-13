---
title: 'Post 12'
date: 2023-08-13
permalink: /posts/gsoc/lpython/post-12/
tags:
  - GSoC
  - LPython
---
<!--more-->
## Corrections to `dict` ([#2261](https://github.com/lcompilers/lpython/pull/2261))

There were issues with `dict` collision resolution, insertion, and popping; both in Linear Probing and Separate Chaining implementations. For example,
- Linear Probing

```python
from lpython import i32

def test():
    d: dict[str, i32] = {'a': 1, 'a': 1}
    print(len(d))   # 2
    d.pop('a')
    print(len(d))   # 1
    d.pop('a')
    print(len(d))   # 0
    d.pop('a')
    print(len(d))   # -1

test()
```
- Separate Chaining

```python
from lpython import i32

def test():
    d: dict[i32, i32] = {2: 1, 2: 1}
    print(len(d))   # 1
    d.pop(2)
    print(len(d))   # 0
    d.pop(2)
    print(len(d))   # -1
    d.pop(2)
    print(len(d))   # -2

test()
```

I fixed these and several related issues, and would document the `dict` implementation here. The `set` implementation is very similar.

In LPython, we have implemented hash tables similar to CPython. CPython uses linear probing for collision resolution, and we provide both that and separate chaining options.

- Hash computation: Similar to both CPython and C++, for integers it is the integer itself (modulo dictionary capacity). For strings, we use Polynomial Rolling Hash.

- Linear Probing
    - Two lists are maintained - one for keys and one for values. Further, there is a list storing keymasks, that indicates if an element is present or not. The following mask values are used
        - 0: no key set
        - 1: key set without probing (i.e., hash value and position are same)
        - 2: key set with probing
        - 3: tombstone (i.e., key was deleted, and now this position is empty)
    - Insertion and deletion occur by linear probing scheme.

- Separate Chaining
    - Here, a chain (linked list) of key-value pairs is maintained at each hash location. For example, to access the `i`th linked list
        ```cpp
        llvm::Value* key_value_pairs = LLVM::CreateLoad(*builder, get_pointer_to_key_value_pairs(dict));
        llvm::Value* dict_i = llvm_utils->create_ptr_gep(key_value_pairs, idx);
        ```
    - This linked list has elements of the form `key, value, next`, where `next` is a pointer to the next element in this LL (or null if none exists)
    - Dictionary insertion and deletion occur via these lists.

- Benchmarks
    - Dict: [Gist](https://gist.github.com/czgdp1807/e7f7b6ca52c57b16b27ec8d0259c6d4a)  by [Gagandeep Singh](https://github.com/czgdp1807)
    - Set: [Gist](https://gist.github.com/kabra1110/fe86159c81a7cfed7c4f52e11e7fa3c6) by me

This marks the end of my official GSoC contribution period. I had a lot of fun in the entire period. I learnt a lot of cool things: getting introduced to LLVM and implementing data structures for an actual compiler! Hope to keep contributing here and to open-source in general.

I would like to thank [Ondřej Čertík](https://github.com/certik/) for the conception of this wonderful compiler. My mentors [Gagandeep Singh](https://github.com/czgdp1807) and [Thirumalai Shaktivel](https://github.com/Thirumalai-Shaktivel) were immensely helpful before and during this entire project. Thank you! Finally, thanks to all LPython contributors on whose work I could build on.

---
title: 'Post 9'
date: 2023-07-23
permalink: /posts/gsoc/lpython/post-9/
tags:
  - GSoC
  - LPython
---
<!--more-->
In the past couple of weeks, an implementation of sets was completed. This week, I worked on benchmarking this. I also started work on another implementation, and progressed on some earlier dictionary issues.

## Set Benchmark ([GitHub Gist](https://gist.github.com/kabra1110/fe86159c81a7cfed7c4f52e11e7fa3c6))

- This benchmarks the recent hash-set implementation that uses linear-probing for collision-resolution. We compare performance of this LPython data-structure with equivalent structures in Python (`set`) and C++ (`unordered_set`).
- Performance of functions set.add and set.remove is tested.
  - One way to do this is to simply insert and remove elements in order (say, from 1 to 1e7). However, this would not lead to collisions, as the set would be rehashed after reaching a size threshold (in our implementation, this is 0.6 of the capacity at any point).
  - To add some more complexity, we use random numbers. To generate pseudo-random numbers without adding major performance overheads, [Lehmer random number generator](https://en.wikipedia.org/wiki/Lehmer_random_number_generator) is used. In particular, the tests use [Schrage's method](https://en.wikipedia.org/wiki/Lehmer_random_number_generator#Schrage's_method) to avoid 64-bit division. This is a straightforward algorithm that uses linear modular arithmetic to generate random numbers.
- As can be seen in the results, LPython outperforms both Python and C++. Surprisingly, C++ was slower than the equivalent Python code.
  - When dealing with integers, LPython is more than 4.5 times faster than C++, and more than 3.6 times faster than Python.
  - With strings, these gains are about 1.2 and 1.02, respectively.
- For C++, we also tested with custom hash functions, to mimic those used in our LPython implementation. This did not affect the results much.

## ​​​​​​​Set (Separate Chaining) (#2198)

​​​- ​​​​This is towards adding the separate-chaining collision-resolution technique. In contrast to linear probing, when collisions happen, we extend the pre-existing linked list of elements at that hash value. We already have it in dictionaries, but there are some issues in that implementation.
- I created all the basic functions, and will fix remaining bugs in the coming week.
​
## ​​​​​​Dictionary (Keys and Values, continued) (#2023)

- ​​​​​​​I had implemented this in a previous week to work with the linear-probing implementation of dictionaries. Now, I extended these functions to also work with the separate-chaining implementation.
- We iterate over all the hash values, and if there is a corresponding linked list, we iterate over that, putting keys/values into a list, the latter returned at the end.

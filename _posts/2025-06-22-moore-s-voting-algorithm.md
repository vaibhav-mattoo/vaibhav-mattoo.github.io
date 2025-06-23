---
title: Moore's Voting Algorithm
description: Array algorithms to find the majority element or number of elements which take over some fraction of space in the array
layout: post
categories: [DSA, Array Algorithms]
tags: [Named Algorithms]
date: 2025-06-22 20:55 -0700
math: true
---

# Moore's Voting Algorithm problem

- Given an integer k and an array of size n, find the number of integers in the array with frequency greater than n/k.
- Other ways it can come up are find the number of elements which take up more than 1/5th of the space in the array, find the majority element in the array (which takes more than 1/2 space) and so on.

### Naive solution

- Worst possible idea is to have two for loops and for each key, count the number of times it shows up. Then store the key in a set so that if a new key is encountered first we check if set.find(key) != set.end() and if not then no need to run inner loop since we already counted once. Due to the two loops, this is already O(n<sup>2</sup>).
- A more natural idea is that since frequency is mentioned, we count use a map where key is the element and value is frequency. This is $O(n \log n)$ since C++ maps are red-black trees, which have $\log n$ update and we need to do this for every element in a single pass of the array.

```cpp
#include <iostream>
#include <map>
#include <vector>

int checking_space_greater(std::vector<int> &arr, int minfreq) {
  std::map<int, int> freq;
  for (const int &values : arr) {
    freq[values]++;
  }
  int count = 0;
  for (const auto &pair : freq) {
    if (pair.second > minfreq) {
      count++;
    }
  }
  return count;
}

int main(int argc, char *argv[]) {
  std::vector<int> arr = {1, 2, 1, 0, 1, 2, 0, 1, 0, 0, 2, 1, 0, 2, 0};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;

  std::cout << "Enter a k for size " << arr.size() << " : ";
  int k;
  std::cin >> k;
  int minfreq = arr.size() / k;

  std::cout << "Number of elements with frequency greater than " << minfreq
            << " : " << checking_space_greater(arr, minfreq);
  return 0;
}

```


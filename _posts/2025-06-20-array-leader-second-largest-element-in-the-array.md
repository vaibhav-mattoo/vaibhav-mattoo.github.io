---
title: Array leaders, Second largest element in the array
description: Array algorithms related to finding some sort of maximum in the array
layout: post
categories: [DSA, Array Algorithms]
tags: []
date: 2025-06-19 05:25 -0700
---

# Array leaders problem

- Find all elements of the array such that they are greater than or equal to every element to their right.

### Bad approach (O(n<sup>2</sup>))

- Basically brute force from left side checking if every element is greater than every element from the right side.

```cpp

#include <iostream>
#include <vector>

void print_array_leaders(std::vector<int> &arr) {
  int key;
  for (int i = 0; i < arr.size(); i++) {
    key = arr[i];
    bool key_is_leader = true;
    for (int j = i + 1; j < arr.size(); j++) {
      if (key < arr[j]) {
        key_is_leader = false;
      }
    }
    if (key_is_leader == true) {
      std::cout << key << " ";
    }
  }
}

int main(int argc, char *argv[]) {
  std::vector<int> arr = {5, 4, 3, 10, 5, 9, 8, 4, 7, 6};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  print_array_leaders(arr);
  return 0;
}
```

### Ideal approach (O(n) in one pass)

- We know that the last element will always be in the leaders array, we maintain a maximum starting from the last element and whenever a new maximum is encountered, it must be a leader.

```cpp

#include <iostream>
#include <vector>

void print_array_leaders(std::vector<int> &arr) {
  int max_index = arr.size() - 1;
  for (int i = arr.size() - 1; i >= 0; i--) {
    if (arr[i] >= arr[max_index]) {
      std::cout << arr[i] << " ";
      max_index = i;
    }
  }
}

int main(int argc, char *argv[]) {
  std::vector<int> arr = {5, 4, 3, 10, 5, 9, 8, 4, 7, 6};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  print_array_leaders(arr);
  return 0;
}
```

# Second largest element in the array problem

- Find the second largest value in the array

### Bad solution (O(n) but two passes)

- Basically brute force, find the max in the array and then pass through the array again and find the new max which is less than the old max.

```cpp

#include <iostream>
#include <vector>

int second_largest_value(std::vector<int> &arr) {
  int max = arr[0];
  for (int &num : arr) {
    if (num > max) {
      max = num;
    }
  }
  int secmax = arr[0];
  for (int &num : arr) {
    if (num > secmax && num < max) {
      secmax = num;
    }
  }
  return secmax;
}

int main(int argc, char *argv[]) {
  std::vector<int> arr = {10, 4, 3, 10, 5, 9, 8, 4, 7, 6};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  std::cout << "Sec largest val: " << second_largest_value(arr) << std::endl;
  return 0;
}
```
### Optimal solution (O(n) but takes 1 pass only)

- Idea is that whenever we assign max variable as new max, the old max that we are discarding is the second largest value in the subarray considered till that point in the loop. However need to be careful that we consider the case where a second max lies is larger than the old second max but is not large enough to trigger a max check like in {10, 9}. To handle this, we add another check where we check if the value is > secmax.

```cpp

#include <iostream>
#include <vector>

int second_largest_value(std::vector<int> &arr) {
  int max = arr[0];
  int secmax = arr[0];
  for (int &value : arr) {
    if (value > max) {
      secmax = max;
      max = value;
    } else if (value > secmax) {
      secmax = value;
    }
  }
  return secmax;
}

int main(int argc, char *argv[]) {
  std::vector<int> arr = {10, 4, 3, 10, 5, 9, 8, 4, 7, 6};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  std::cout << "Sec largest val: " << second_largest_value(arr) << std::endl;
  return 0;
}
```

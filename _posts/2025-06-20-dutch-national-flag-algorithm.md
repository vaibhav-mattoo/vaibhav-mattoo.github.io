---
title: Dutch National Flag Algorithm
description: Array algorithms to sort an array of three elements in O(n) and O(1) space
layout: post
categories: [DSA, Array Algorithms]
tags: [Named Algorithms, Sorting]
date: 2025-06-20 08:20 -0700
---

# Dutch National Flag problem

- Sort an array of just three elements (0, 1, 2) in the most efficient way possible.

### Bad solution (O(n log n) and space O(n))

- we have to sort so the brute force method is just to Mergesort as lower bound of sorting and we ignore the fixed range.

```cpp

#include <iostream>
#include <vector>

void merge(std::vector<int> &arr, int left, int mid, int right) {
  int n1 = mid - left + 1;
  int n2 = right - mid;
  std::vector<int> leftarr(n1);
  std::vector<int> rightarr(n2);
  for (int i = 0; i < n1; i++) {
    leftarr[i] = arr[left + i];
  }
  for (int i = 0; i < n2; i++) {
    rightarr[i] = arr[mid + 1 + i];
  }
  int i = 0, j = 0, k = left;
  while (i < n1 && j < n2) {
    if (leftarr[i] < rightarr[j]) {
      arr[k] = leftarr[i];
      i++;
    } else {
      arr[k] = rightarr[j];
      j++;
    }
    k++;
  }
  while (i < n1) {
    arr[k] = leftarr[i];
    i++;
    k++;
  }
  while (j < n2) {
    arr[k] = rightarr[j];
    j++;
    k++;
  }
}

void mergesort(std::vector<int> &arr, int left, int right) {
  if (left < right) {
    int mid = left + (right - left) / 2;
    mergesort(arr, left, mid);
    mergesort(arr, mid + 1, right);
    merge(arr, left, mid, right);
  }
}

void mergesort(std::vector<int> &arr) { mergesort(arr, 0, arr.size() - 1); }

int main(int argc, char *argv[]) {
  std::vector<int> arr = {1, 2, 1, 0, 1, 2, 0, 1, 0, 0, 2, 1, 0, 2, 0};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  mergesort(arr);
  std::cout << "Sorted arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}
```


### Better solution (O(n) with two passes)

- Since we know the total number and the kinds of elements in advance we can maintain three counts count0, count1, and count2.
- In first pass if we encounter 0 then we do count0++ and same for 1, 2. Then in second pass we populate the array again based on the counts we stored.

```cpp
#include <iostream>
#include <vector>

// we can just remove count 2 from everywhere and it would still work perfectly
void counting_method(std::vector<int> &arr) {
  int count0 = 0, count1 = 0, count2 = 0;
  for (int &num : arr) {
    if (num == 0) {
      count0++;
    } else if (num == 1) {
      count1++;
    } else {
      count2++;
    }
  }
  for (int &num : arr) {
    if (count0 != 0) {
      num = 0;
      count0--;
    } else if (count1 != 0) {
      num = 1;
      count1--;
    } else {
      num = 2;
      count2--;
    }
  }
}

int main(int argc, char *argv[]) {
  std::vector<int> arr = {1, 2, 1, 0, 1, 2, 0, 1, 0, 0, 2, 1, 0, 2, 0};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  counting_method(arr);
  std::cout << "Sorted arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}

```

### Optimal solution (O(n) in 1 pass)

![Desktop View](/assets/img/dnf_algo_pointers.png)
- Main value is to push the low value to the left and high values to the right, and middle values left in middle.
- We maintain a low, mid and high pointer.
  - The low pointer points to one ahead of the last low value, so on the first mid value. Everything to left of the low pointer is low.
  - The mid pointer points to one ahead of the last confirmed mid value, so on the first unknown value. Everything betweem the mid pointer and high pointer (inclusive) is unknown.
  - Everything to the right of the high pointer is high.

```cpp
#include <iostream>
#include <vector>

void dnf_algo(std::vector<int> &arr) {
  int low = 0, mid = 0, high = arr.size() - 1;
  while (high >= mid) {
    if (arr[mid] == 0) {
      std::swap(arr[low], arr[mid]);
      low++;
      mid++;
    } else if (arr[mid] == 1) {
      mid++;
    } else {
      std::swap(arr[mid], arr[high]);
      high--;
    }
  }
}

int main(int argc, char *argv[]) {
  std::vector<int> arr = {1, 2, 1, 0, 1, 2, 0, 1, 0, 0, 2, 1, 0, 2, 0};
  std::cout << "Input arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  dnf_algo(arr);
  std::cout << "Sorted arr : ";
  for (const auto &num : arr) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}

```

---
title: Sorting algorithms
description: Some theory and implementation of major array sorting algorithms in C++
layout: post
math: true
categories: [DSA, Array Algorithms]
tags: [Named Algorithms]
date: 2025-06-19 05:25 -0700
---

## Fundamental lower bound

- Any deterministic comparison based sorting algorithm must perform at least Ω(n log n) comparisons in the worst case to sort n elements.
  - Any such algorithm requires at least log(n!) comparisons for some input.

### Decision Tree Model

- This bound is derived from decision tree analysis, which represents the execution of any comparison based sorting algorithm as a binary tree where:
  - Each internal node represents a comparison between two elements
  - Each edge represents the outcome of that comparison
  - Each leaf represents a final sorted permutation of the input
- The algorithm then follows a path from root to leaf based on the comparison results, and the **height of this tree** represents to maximum number of comparisons the algo makes in worst case.

#### Permutation requirement

- For a sorting algorithm to be correct, the decision tree must have **at least n! leaves**, one for each possible permutation that could be the correct answer.
  - This is because there are n! input combinations and each of them requires a potentiallt different sequence of comparisons to determine the correct sorted output.

### Mathematical lower bound

- Each comparison provides exactly one bit of information, and answers a yes or no question. In the best case, each comparison should eliminate roughly half of the remaining possibilities so we need enough comparisons to narrow down from n! possibilities to exactly 1.
- Since a binary tree of height h has at most 2<sup>h</sup> leaves, we need at least n! leaves:

$$
 2^h \geq n! 
$$

- Taking logarithm both sides:

$$
 h \geq \log_2(n!) 
$$

- So any comparison based sorting algorithm must take at least log (n!) comparisons in worst case.

- **Stirling's approximation**: $ n! \approx \sqrt{2\pi n} \left(\frac{n}{e}\right)^n 
$

$$
\log (n!) = \frac{1}{2} \log (2 \pi n) + n \log (n) - n \log (e) 
$$


$$
= n \log (n) - 1.44n + O(\log n)
$$

- So log (n!) = θ(n log n) and this lower bound is tight meaning we cannot do better than Ω(n log n) and we have algorithms like mergesort and heapsort which achieve this lower bound.

## Classifying sorting algorithms

- By comparison:
  - **Comparison based algorithms**: Use element comparisons to determine order (mergesort, quicksort, heapsort)
  - **Non comparison based algorithms**: Exploit specific properties of data like knowing that input numbers have a range or some distribution properties and use this to break the comparison barrier and achieve linear time (counting sort, radix sort, bucket sort)
- By memory classification usage:
  - **In place algorithms**: Sort the array within the original memory space without requiring additional arrays proportional to the input size (require only constant extra memory beyond input array). Examples : Selection Sort , Heap sort ,Quick sort (pseudo in-place)
  - Out of place algorithms: O(n) or more space than the input size as temporary storage. Examples: Merge sort, counting sort, radix sort
- Usage:
  - **Embedded systems**: Prefer in place comparison based due to memory constraints.
  - **Database systems**: Use non comparison if data limited range (non comparison becomes better as dataset grows), and out of place if they maintain relative order of equal elements.
  - **Real time systems**: Choose algo based on worst case always, not on average performance.

### Heirarchy of efficiency

- Simple algorithms (O(n<sup>2</sup>)) : Bubble sort, selection sort, insertion sort - good for small dataset as usually low overhead
- Efficient algorithms (O (n log n)) : Mergesort, quicksort, heapsort - achieve optimal comparison based bound, merge sort guarantees worst case performance, quicksort has better average case constant and heap sort is in place
- Specialized algorithms (O(n) or O(n+k)) : counting sort, radix sort, bucket sort - can beat comparison bounf by exploiting some property

## Bubble sort

- Takes multiple passes through the array, and in each pass compares adjacent elements and swap them if they are in the wrong order, largest element bubbles up to correct position in each pass hence the name.
  - In each pass the largest unsorted element moves to the right side of the array, so we need a total of n-1 passes through the array in the worst case to guarantee complete sorting. Here i controls what the current pass is starting from 0, which also equals the current size of the sorted section of largest elements on the right side.
  - So after pass i, the last i elements are sorted so n-i elements are left to sort. Since we are swapping with one element ahead j only needs to go from 0 to n-i-1 since the swapping of j and j+1 if needed handles the (n-i-1)th element (last in the unsorted part).
  - In each swapping pair we only swap if the left one is bigger than the right one.
  - If we ever have a pass in the unsorted section where nothing is swapped, this means the unsorted section is actually already sorted, so we have a flag to detect that.

```cpp

#include <iostream>
#include <sstream>
#include <string>
#include <vector>

void bubblesort(std::vector<int> &arr) {
  int n = arr.size();
  for (int i = 0; i < n - 1; i++) {
    bool swapped = false;
    for (int j = 0; j < n - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        std::swap(arr[j], arr[j + 1]);
        swapped = true;
      }
    }
    if (!swapped) {
      break;
    }
  }
}

int main(int argc, char *argv[]) {
  std::vector<int> input;
  std::string line;
  std::cout << "Enter space-separated numbers: ";
  std::getline(std::cin, line);
  std::istringstream iss(line);
  int num;
  while (iss >> num) {
    input.push_back(num);
  }
  std::cout << "Input array : ";
  for (auto &num : input) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  bubblesort(input);

  std::cout << "Sorted array : ";
  for (auto &num : input) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}

```

### Bubble sort complexity analysis

- **Best case is O(n)** : If array is already sorted, algo makes one pass through the array, sees that swapped flag remains false and exits out of outer loop after n-1 comparisons with no swaps. Hence, O(n).
- **Average case is O(n<sup>2</sup>)** : If elements are in random order, algorithm performs multiple passes with each element having a probability of being out of order.
  - On average, half the adjacent pairs will be out of order
  - Each pass reduces the unsorted portion by one element
  - Expected number of swaps per pass: approximately $\frac{n-i}{2}$ for pass i
  - Total passes needed: $n-1$ (worst case, since we can't predict early termination)
  - So comparisons are $\frac{n(n-1)}{2} \approx \frac{n^2}{2}$
  - Swaps are approximately $\frac{n^2}{4}$
- **Worst case is O(n<sup>2</sup>)**: If array is sorted in reverse order, every adjacent pair is out of order and every comparison results in a swap with no early termination. So $\frac{n(n-1)}{2}$ comparisons and swaps.

## Selection sort
 - Divides the array into sorted and unsorted portions, repeatedly finds the minimum element from the unsorted portion and places it at the beginning of the unsorted portion.
  - In each pass i, we find the minimum element in the unsorted portion (from index i to n-1) and swap it with the element at position i.
  - After pass i, the first i+1 elements are sorted in their correct positions, so the sorted portion grows from left to right.
  - Unlike bubble sort, selection sort always performs the same number of comparisons regardless of input order, it must scan the entire unsorted portion to find the minimum.
  - The algorithm makes at most n-1 swaps total (one per pass), which can be beneficial when swap operations are expensive.
  - No early termination optimization is possible because we must always verify that each position contains the true minimum of the remaining elements.

```cpp

#include <iostream>
#include <sstream>
#include <string>
#include <vector>

void selectionsort(std::vector<int> &arr) {
  int n = arr.size();
  for (int i = 0; i < n - 1; i++) {
    int min_idx = i;
    for (int j = i + 1; j < n; j++) {
      if (arr[j] < arr[min_idx]) {
        min_idx = j;
      }
    }
    if (min_idx != i) {
      std::swap(arr[i], arr[min_idx]);
    }
  }
}

int main(int argc, char *argv[]) {
  std::vector<int> input;
  std::string line;
  std::cout << "Enter space-separated numbers: ";
  std::getline(std::cin, line);
  std::istringstream iss(line);
  int num;
  while (iss >> num) {
    input.push_back(num);
  }
  std::cout << "Input array : ";
  for (auto &num : input) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  selectionsort(input);

  std::cout << "Sorted array : ";
  for (auto &num : input) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}
```

### Selection sort complexity analysis

- **All cases are O(n<sup>2</sup>)**: Even when array is already sorted, selection sort must verify that each position contains the minimum by scanning all remaining elements. For position i, it scans positions i+1 to n-1, making $(n-1) + (n-2) + ... + 1 = \frac{n(n-1)}{2}$ comparisons total. Hence, O(n<sup>2</sup>). The algorithm does not adapt to existing order in data and has consistent performance.

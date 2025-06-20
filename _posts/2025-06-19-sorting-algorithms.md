---
title: Sorting algorithms
description: Some theory and implementation of major array sorting algorithms in C++
layout: post
math: true
categories: [DSA, Array Algorithms]
tags: [Named Algorithms, Sorting]
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

## Insertion sort

- Builds the sorted array one element at a time by taking each element from the unsorted portion and inserting it into its correct position in the sorted portion.
  - In each pass i (starting from 1), we take the element at position i (key) and find its correct position in the already sorted portion (indices 0 to i-1).
  - We shift all elements in the sorted portion that are greater than the key one position to the right, then insert the key at the correct position.
  - After pass i, the first i+1 elements are sorted relative to each other, so the sorted portion grows from left to right.
  - The algorithm is adaptive - it performs well on nearly sorted arrays because elements that are already in correct positions require minimal shifting.
  - Similar to how you might sort playing cards in your hand by picking up each card and inserting it in the right position among the cards you've already sorted.

```cpp
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

void insertionsort(std::vector<int> &arr) {
  int n = arr.size();
  for (int i = 1; i < n; i++) {
    int key = arr[i];
    int j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
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
  insertionsort(input);

  std::cout << "Sorted array : ";
  for (auto &num : input) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}

```


### Insertion sort complexity analysis

- **Best case is O(n)** : When array is already sorted, each element at position i only needs to be compared with the element at position i-1. Since arr[i-1] ≤ arr[i], no shifting is required and the while loop executes 0 times. Total of n-1 comparisons with 0 shifts. Hence, O(n).
- **Average case is O(n<sup>2</sup>)** : For randomly ordered elements, each element at position i needs to be compared with approximately half of the preceding elements.
  - Element at position i has i preceding elements in the sorted portion
  - On average, need to compare with $\frac{i}{2}$ elements and shift $\frac{i}{2}$ positions
  - Expected comparisons for element i: $\frac{i}{2}$
  - Total comparisons: $\sum_{i=1}^{n-1} \frac{i}{2} = \frac{1}{2} \cdot \frac{(n-1)n}{2} \approx \frac{n^2}{4}$
  - Total shifts: Similar magnitude to comparisons
- **Worst case is O(n<sup>2</sup>)**: When array is sorted in reverse order, each element must be compared with and shifted past all preceding elements. Element at position i requires i comparisons and i shifts. Total comparisons and shifts: $1 + 2 + 3 + ... + (n-1) = \frac{n(n-1)}{2}$. Total operations: $n(n-1) \approx n^2$.

## Merge sort

- A divide-and-conquer algorithm that recursively divides the array into two halves, sorts each half separately, and then merges the sorted halves back together.
  - **Divide phase**: For array segment from index `left` to `right`, calculate midpoint as `mid = left + (right - left) / 2` to avoid integer overflow. Split into two subarrays: `[left...mid]` and `[mid+1...right]`.
  - **Base case**: When `left >= right`, the subarray has 0 or 1 element and is trivially sorted, so return immediately.
  - **Recursive calls**: First sort left half `mergesort(arr, left, mid)`, then sort right half `mergesort(arr, mid+1, right)`.
  - **Merge operation**: After both halves are sorted, merge them back together. Create temporary arrays: left subarray has size `n1 = mid - left + 1` elements, right subarray has size `n2 = right - mid` elements.
  - **Why these sizes**: Left subarray spans from `left` to `mid` inclusive, so size is `mid - left + 1`. Right subarray spans from `mid+1` to `right` inclusive, so size is `right - (mid+1) + 1 = right - mid`.
  - **Merge process**: Use three pointers - `i` for left array, `j` for right array, `k` for original array starting at `left`. Compare `leftArr[i]` with `rightArr[j]`, place smaller element at `arr[k]`, advance corresponding pointer. Continue until one array is exhausted, then copy remaining elements.
  - **Stability**: When elements are equal, always take from left array first (`leftArr[i] <= rightArr[j]`) to maintain relative order.

```cpp

#include <iostream>
#include <sstream>
#include <string>
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
  // i = left array index
  // j = right array index
  // k = merged array index
  while (i < n1 && j < n2) {
    if (leftarr[i] <= rightarr[j]) {
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
  mergesort(input);

  std::cout << "Sorted array : ";
  for (auto &num : input) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}

```


### Merge sort complexity analysis

- **Best case is O(n log n)** : Even when array is already sorted, merge sort still divides the array into log n levels and performs n comparisons at each level. The recursive structure ensures consistent depth of log n regardless of input order. At each level, exactly n elements are processed during merging operations.
- **Average case is O(n log n)** : The divide-and-conquer approach guarantees balanced recursion regardless of input distribution.
  - Array is always divided exactly in half at midpoint, creating a balanced binary tree of recursive calls
  - Tree depth is always ⌊log₂ n⌋ + 1 levels due to binary division
  - Each level processes exactly n elements during the merge operations across all subarrays at that level
  - Total work: n elements × log n levels = n log n comparisons and movements
  - Space complexity: O(n) for temporary arrays during merging, plus O(log n) for recursion stack
- **Worst case is O(n log n)** : Unlike quicksort, merge sort has no worst input cases because the division strategy is deterministic. The recursive division is always balanced since we split at the calculated midpoint `left + (right - left) / 2`. Maximum recursion depth is exactly log n, and each level requires exactly n operations for merging all subarrays at that level. This makes merge sort optimal among comparison-based sorting algorithms with guaranteed performance.

## Quick sort

- A divide-and-conquer algorithm that selects a 'pivot' element, partitions the array around the pivot, and recursively sorts the subarrays on either side of the pivot.
  - **Pivot selection**: Choose an element from the array as the pivot. Common strategies include first element, last element, random element, or median-of-three.
  - **Partition phase**: Rearrange the array so that all elements smaller than the pivot come before it, and all elements greater than the pivot come after it. The pivot is now in its final sorted position.
  - **Partition process**: Use two pointers - `i` starting from `low-1` (before the subarray) and `j` iterating from `low` to `high-1`. When `arr[j] <= pivot`, increment `i` and swap `arr[i]` with `arr[j]`. This maintains the invariant that elements from `low` to `i` are ≤ pivot.
  - **Why i starts at low-1**: The variable `i` tracks the boundary of the "smaller than pivot" region. Starting at `low-1` means initially no elements are in the smaller region. When we find an element ≤ pivot, we first increment `i` then swap, effectively expanding the smaller region.
  - **Final pivot placement**: After partitioning, swap the pivot (at `high`) with `arr[i+1]` to place the pivot in its correct position. Return `i+1` as the partition index.
  - **Recursive calls**: Sort left subarray `quicksort(arr, low, partitionIndex-1)` and right subarray `quicksort(arr, partitionIndex+1, high)`. The pivot at `partitionIndex` is already in correct position.
  - **Base case**: When `low >= high`, the subarray has 0 or 1 element and is trivially sorted.
  - **In-place sorting**: Unlike merge sort, quicksort requires only O(1) additional space for partitioning, making it an in-place algorithm.

```cpp
#include <iostream>
#include <sstream>
#include <string>
#include <utility>
#include <vector>

int partition(std::vector<int> &arr, int low, int high) {
  int pivot = arr[high];
  int i = low - 1;
  for (int j = low; j < high; j++) {
    if (arr[j] <= pivot) {
      i++;
      std::swap(arr[i], arr[j]);
    }
  }
  std::swap(arr[i + 1], arr[high]);
  return i + 1;
}

void quicksort(std::vector<int> &arr, int low, int high) {
  if (low < high) {
    int partindex = partition(arr, low, high);
    quicksort(arr, low, partindex - 1);
    quicksort(arr, partindex + 1, high);
  }
}

void quicksort(std::vector<int> &arr) { quicksort(arr, 0, arr.size() - 1); }

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
  quicksort(input);

  std::cout << "Sorted array : ";
  for (auto &num : input) {
    std::cout << num << " ";
  }
  std::cout << std::endl;
  return 0;
}

```

### Quick sort complexity analysis

- **Best case is O(n log n)** : When the pivot always divides the array into two equal halves, creating a balanced recursion tree. Each level processes n elements for partitioning, and there are log n levels. This happens when the pivot is always the median element of the subarray.
- **Average case is O(n log n)** : For randomly distributed elements, the pivot on average divides the array into reasonably balanced partitions.
  - Even if partitions are not exactly equal (e.g., 1/4 and 3/4 split), the recursion depth remains O(log n)
  - Each level still processes n elements total across all partitions
  - Expected number of comparisons: approximately 1.39 n log n
  - Space complexity: O(log n) for recursion stack in average case
  - Performs better than merge sort in practice due to better cache locality and fewer data movements
- **Worst case is O(n²)** : When the pivot is always the smallest or largest element, creating maximally unbalanced partitions. This happens with already sorted arrays (ascending or descending) when using first/last element as pivot. Each recursive call reduces array size by only 1, leading to n levels of recursion with n, n-1, n-2, ... comparisons respectively. Total: n + (n-1) + (n-2) + ... + 1 = n(n+1)/2 ≈ n²/2 comparisons. Space complexity becomes O(n) due to deep recursion stack.

## Heap sort

- A comparison-based sorting algorithm that uses a binary heap data structure to sort elements by repeatedly extracting the maximum element from a max-heap.
  - **Heap property**: A complete binary tree where each parent node is greater than or equal to its children (max-heap). For array representation, parent of element at index `i` is at `(i-1)/2`, left child at `2*i+1`, right child at `2*i+2`.
  - **Build heap phase**: Convert the unsorted array into a max-heap by calling heapify on all non-leaf nodes from bottom to top. Start from index `n/2-1` (last non-leaf node) down to 0.
  - **Heapify operation**: For a node at index `i`, compare it with its children and swap with the largest child if necessary. Recursively heapify the affected subtree to maintain heap property throughout.
  - **Why start from n/2-1**: In a complete binary tree with n nodes, nodes from index `n/2` to `n-1` are leaf nodes. Non-leaf nodes are from 0 to `n/2-1`, so we heapify from the last non-leaf node upward.
  - **Sorting phase**: Repeatedly extract the maximum element (root) by swapping it with the last element, reduce heap size by 1, and heapify the root to restore heap property. The extracted elements accumulate at the end in sorted order.
  - **In-place sorting**: The algorithm sorts within the original array - the heap occupies the unsorted portion while sorted elements accumulate at the end.
  - **Unstable sorting**: Due to long-distance swaps during heapify operations, equal elements may change their relative order.



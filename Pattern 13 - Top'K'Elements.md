# Pattern 13: Top 'K' Elements

Any problem that asks us to find the <b>top/smallest/frequent K</b> elements among a given set falls under this pattern.

<s>The best data structure that comes to mind to keep track of <b>K</b> elements is <s>Heap</s>. This pattern will make use of the <s><b>Heap</b></s> to solve multiple problems dealing with <b>K</b> elements at a time from a set of given elements.</s>

### ❗ NOTE

Although this course uses <b>Heaps</b> to solve <b>Top 'K' Elements</b> problems, <b>JavaScript</b> does not have a built in method for <b>Heaps/Priority Queues</b>. It can be very time consuming to implement a <b>Heap class</b> from scratch, especially during an interview. After reviewing the <i>JavaScript</i> solutions on <i>Leetcode</i> the most effecient way to solve a <b>Top 'K' Elements</b> problem is usually with <b>[QuickSort](https://github.com/Chanda-Abdul/leetcode/blob/master/0%20%E2%9D%97Sort%20Algorithms.md#-quick-sort)</b>, <b>[BinarySearch](https://github.com/Chanda-Abdul/leetcode/blob/master/0%20%E2%9D%97Sort%20Algorithms.md#binary-search)</b>, <b>[BucketSort](https://initjs.org/bucket-sort-in-javascript-dc040b8f0058)</b>, <b>[Greedy Algorithms](https://github.com/Chanda-Abdul/Grokking-Algorithm-Book-Notes/blob/main/8.%20Greedy%20Algoritms.md)</b>, or <b>[HashMaps](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)</b>. For more information take a look at these

- [js heap implementation](https://dandkim.com/js-heap-implementation/)
- [implementing heaps in javascript](https://blog.bitsrc.io/implementing-heaps-in-javascript-c3fbf1cb2e65)
- [heap data structure in javascript](https://learnersbucket.com/tutorials/array/heap-data-structure-in-javascript/)

## Top 'K' Numbers (easy)

> Given an unsorted array of numbers, find the `K` largest numbers in it.

<b>Note:</b> For a detailed discussion about different approaches to solve this problem, take a look at [Kth Smallest Number](#kth-smallest-number-easy).

A <b>brute force solution</b> could be to sort the array and return the <b>largest K numbers</b>. The time complexity of such an algorithm will be `O(N*logN)` as we need to use a sorting algorithm like <b>[Quicksort](https://github.com/Chanda-Abdul/leetcode/blob/master/%E2%9D%97Sort%20Algorithms.md#-quick-sort)</b>. Can we do better than that?

<!-- <s>The best data structure that comes to mind to keep track of top `K` elements is Heap. Let's see if we can use a heap to find a better algorithm.

If we iterate through the array one element at a time and keep `K` largest numbers in a heap such that each time we find a larger number than the smallest number in the heap, we do two things:
1. Take out the smallest number from the heap, and
2. Insert the larger number into the heap.

This will ensure that we always have `K` largest numbers in the heap. The most efficient way to repeatedly find the smallest number among a set of numbers will be to use a min-heap. As we know, we can find the smallest number in a min-heap in constant time `O(1)`, since the smallest number is always at the root of the heap. Extracting the smallest number from a min-heap will take `O(logN)` (if the heap has `N` elements) as the heap needs to readjust after the removal of an element.</s>

Let's take <b>Example 1</b> to go through each step of our algorithm:

Given array: `[3, 1, 5, 12, 2, 11]`, and `K=3`
<s>

1. First, let's insert `K` elements in the min-heap.
2. After the insertion, the heap will have three numbers `[3, 1, 5]` with `1` being the root as it is the smallest element.
3. We`ll iterate through the remaining numbers and perform the above-mentioned two steps if we find a number larger than the root of the heap.
4. The 4th number is `12` which is larger than the root (which is `1`), so let's take out `1` and insert `12`. Now the heap will have `[3, 5, 12]` with `3` being the root as it is the smallest element.
5. The 5th number is `2` which is not bigger than the root of the heap (`3`), so we can skip this as we already have top three numbers in the heap.
6. The last number is `11` which is bigger than the root (which is `3`), so let's take out `3` and insert `11`. Finally, the heap has the largest three numbers: [5, 12, 11]

As discussed above, it will take us `O(logK)` to extract the minimum number from the min-heap. So the overall time complexity of our algorithm will be `O(K*logK+(N-K)*logK)` since, first, we insert `K` numbers in the heap and then iterate through the remaining numbers and at every step, in the worst case, we need to extract the minimum number and insert a new number in the heap. This algorithm is better than `O(N*logN)`.</s> -->

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

vector<int> quickSort(vector<int> array) {
  // recursive base case
  if (array.size() == 1) return array;

  // set pivot to end value
  int pivotPoint = array[array.size() - 1];
  vector<int> firstHalf;
  vector<int> secondHalf;

  for (int i = 0; i < array.size() - 1; i++) {
    if (array[i] < pivotPoint) {
      firstHalf.push_back(array[i]);
    } else {
      secondHalf.push_back(array[i]);
    }
  }

  // recursively sort
  vector<int> result;
  if (firstHalf.size() > 0 && secondHalf.size() > 0) {
    vector<int> sortedFirst = quickSort(firstHalf);
    vector<int> sortedSecond = quickSort(secondHalf);
    result.insert(result.end(), sortedFirst.begin(), sortedFirst.end());
    result.push_back(pivotPoint);
    result.insert(result.end(), sortedSecond.begin(), sortedSecond.end());
  } else if (firstHalf.size() > 0) {
    vector<int> sortedFirst = quickSort(firstHalf);
    result.insert(result.end(), sortedFirst.begin(), sortedFirst.end());
    result.push_back(pivotPoint);
  } else {
    result.push_back(pivotPoint);
    vector<int> sortedSecond = quickSort(secondHalf);
    result.insert(result.end(), sortedSecond.begin(), sortedSecond.end());
  }
  return result;
}

vector<int> findKLargestNumbers(vector<int> nums, int k) {
  vector<int> sorted = quickSort(nums);
  return vector<int>(sorted.begin() + (sorted.size() - k), sorted.end());
}

int main() {
  vector<int> result1 = findKLargestNumbers({3, 1, 5, 12, 2, 11}, 3);
  cout << "Here are the top K numbers: ";
  for (int n : result1) cout << n << " ";
  cout << endl;

  vector<int> result2 = findKLargestNumbers({5, 12, 11, -1, 12}, 3);
  cout << "Here are the top K numbers: ";
  for (int n : result2) cout << n << " ";
  cout << endl;

  return 0;
}
```

- As discussed above, the time complexity of this algorithm is `O(K * log K +(N - K) * logK)`, which is asymptotically equal to `O(N*logK)`.
- The space complexity will be `O(K)` since we need to store the top `K` numbers in an array.

## Kth Smallest Number (easy)

https://leetcode.com/problems/kth-largest-element-in-an-array/

> Given an unsorted array of numbers, find `Kth` smallest number in it.
>
> Please note that it is the `Kth` smallest number in the sorted order, not the `Kth` distinct element.

```cpp
#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>
using namespace std;

int sortPartition(vector<int>& array, int pivot, int start, int end) {
  // swap the pivot to the end
  swap(array[pivot], array[end]);

  int i = start;
  int j = start;

  while (j < end) {
    if (array[j] <= array[end]) {
      swap(array[i], array[j]);
      i++;
    }
    j++;
  }

  // swap pivot to its final position
  swap(array[i], array[end]);
  return i;
}

int findKthSmallest(vector<int>& nums, int k) {
  int finalIndex = nums.size() - k;
  int start = 0;
  int end = nums.size() - 1;

  while (start <= end) {
    // random number between start and end for pivot
    int pivot = start + rand() % (end - start + 1);
    // final position of the pivot in a sorted array
    int pivotIndex = sortPartition(nums, pivot, start, end);
    if (pivotIndex == finalIndex) return nums[finalIndex];

    // if pivotIndex is smaller we undershot, so look only on the second half
    if (pivotIndex < finalIndex) start = pivotIndex + 1;
    // if pivotIndex is larger we overshot, so look only on the first half
    else end = pivotIndex - 1;
  }
  return -1;
}

int main() {
  srand(time(0));

  vector<int> arr1 = {3, 2, 3, 1, 2, 4, 5, 5, 6};
  cout << "Here is the Kth number: " << findKthSmallest(arr1, 4) << endl;  // 4

  vector<int> arr2 = {3, 1, 5, 12, 2, 11};
  cout << "Here is the Kth number: " << findKthSmallest(arr2, 3) << endl;  // 5

  vector<int> arr3 = {5, 12, 11, -1, 12};
  cout << "Here is the Kth number: " << findKthSmallest(arr3, 3) << endl;  // 11

  return 0;
}
```

- As discussed above, the time complexity of this algorithm is `O(K * log K +(N - K) * logK)`, which is asymptotically equal to `O(N*logK)`.
- The space complexity will be `O(K)` since we need to store the top `K` numbers in an array.

## 'K' Closest Points to the Origin (easy)

https://leetcode.com/problems/k-closest-points-to-origin/

> Given an array of points in a 2D plane, find `K` closest points to the origin.

<b>Note:</b> For a detailed discussion about different approaches to solve this problem, take a look at [Kth Smallest Number](#kth-smallest-number-easy).

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>
using namespace std;

vector<vector<int>> findClosestPoints(vector<vector<int>>& points, int k) {
    sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) {
        int distA = a[0] * a[0] + a[1] * a[1];
        int distB = b[0] * b[0] + b[1] * b[1];
        return distA < distB;
    });

    vector<vector<int>> result(points.begin(), points.begin() + k);
    return result;
}

int main() {
    // The Euclidean distance between (1, 2) and the origin is sqrt(5).
    // The Euclidean distance between (1, 3) and the origin is sqrt(10).
    // Since sqrt(5) < sqrt(10), therefore (1, 2) is closer to the origin.
    vector<vector<int>> points1 = {{1, 2}, {1, 3}};
    vector<vector<int>> result1 = findClosestPoints(points1, 1);
    cout << "Here are the k points closest the origin: ";
    for (auto& p : result1) cout << "[" << p[0] << "," << p[1] << "] ";
    cout << endl;

    // [[1, 3], [2, -1]]
    vector<vector<int>> points2 = {{1, 3}, {3, 4}, {2, -1}};
    vector<vector<int>> result2 = findClosestPoints(points2, 2);
    cout << "Here are the k points closest the origin: ";
    for (auto& p : result2) cout << "[" << p[0] << "," << p[1] << "] ";
    cout << endl;

    return 0;
}
```

## Connect Ropes (easy)

https://leetcode.com/problems/minimum-cost-to-connect-sticks/

> Given `N` ropes with different lengths, we need to connect these ropes into one big rope with minimum cost. The cost of connecting two ropes is equal to the sum of their lengths.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int minimumCostToConnectRopes(vector<int>& ropeLengths) {
    if (ropeLengths.size() == 1) return 0;

    sort(ropeLengths.begin(), ropeLengths.end());

    int totalCost = 0;

    while (ropeLengths.size() > 1) {
        int firstRope = ropeLengths[0];
        int secondRope = ropeLengths[1];
        ropeLengths.erase(ropeLengths.begin());
        ropeLengths.erase(ropeLengths.begin());

        int currentCost = firstRope + secondRope;
        totalCost += currentCost;

        if (ropeLengths.size() == 0) return totalCost;

        // Binary search for insertion position
        int start = 0;
        int end = ropeLengths.size();

        while (start < end) {
            int mid = start + (end - start) / 2;
            if (currentCost < ropeLengths[mid]) {
                end = mid;
            } else {
                start = mid + 1;
            }
        }
        ropeLengths.insert(ropeLengths.begin() + start, currentCost);
    }

    return totalCost;
}

int main() {
    // 33
    // First connect 1+3(=4), then 4+5(=9), and then 9+11(=20). So the total cost is 33 (4+9+20)
    vector<int> ropes1 = {1, 3, 11, 5};
    cout << "Minimum cost to connect ropes: " << minimumCostToConnectRopes(ropes1) << endl;

    // 36
    // First connect 3+4(=7), then 5+6(=11), 7+11(=18). Total cost is 36 (7+11+18)
    vector<int> ropes2 = {3, 4, 5, 6};
    cout << "Minimum cost to connect ropes: " << minimumCostToConnectRopes(ropes2) << endl;

    // 42
    // First connect 1+2(=3), then 3+3(=6), 6+5(=11), 11+11(=22). Total cost is 42 (3+6+11+22)
    vector<int> ropes3 = {1, 3, 11, 5, 2};
    cout << "Minimum cost to connect ropes: " << minimumCostToConnectRopes(ropes3) << endl;

    return 0;
}
```

- Given `N` ropes, we need `O(N^2)` for the <b>Binary Search</b>.
- The space complexity will be `O(1)` .

## 👩🏽‍🦯 Top 'K' Frequent Numbers (medium)

https://leetcode.com/problems/top-k-frequent-elements/

> Given an unsorted array of numbers, find the top `K` frequently occurring numbers in it.

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>
#include <cstdlib>
#include <ctime>
using namespace std;

int sortPartitionForFreq(vector<int>& mapKeys, unordered_map<int, int>& numMap, int pivot, int start, int end) {
    swap(mapKeys[pivot], mapKeys[end]);
    int swapIndex = start;

    for (int i = start; i < end; i++) {
        if (numMap[mapKeys[i]] < numMap[mapKeys[end]]) {
            swap(mapKeys[i], mapKeys[swapIndex]);
            swapIndex++;
        }
    }
    swap(mapKeys[end], mapKeys[swapIndex]);
    return swapIndex;
}

vector<int> findKFrequentNumbers(vector<int>& nums, int k) {
    unordered_map<int, int> numMap;
    for (int num : nums) {
        numMap[num]++;
    }

    vector<int> mapKeys;
    for (auto& p : numMap) {
        mapKeys.push_back(p.first);
    }

    int finalIndex = mapKeys.size() - k;
    int start = 0;
    int end = mapKeys.size() - 1;

    // Quicksort with partition
    while (start <= end) {
        int pivotPoint = start + rand() % (end - start + 1);
        int pivotIndex = sortPartitionForFreq(mapKeys, numMap, pivotPoint, start, end);

        if (pivotIndex == finalIndex) {
            return vector<int>(mapKeys.begin() + finalIndex, mapKeys.end());
        }
        if (pivotIndex < finalIndex) {
            start = pivotIndex + 1;
        } else {
            end = pivotIndex - 1;
        }
    }

    return vector<int>();
}

int main() {
    srand(time(0));

    vector<int> nums1 = {1, 3, 5, 12, 11, 12, 11};
    vector<int> result1 = findKFrequentNumbers(nums1, 2);
    cout << "Here are the K frequent numbers: ";
    for (int n : result1) cout << n << " ";
    cout << endl;

    vector<int> nums2 = {5, 12, 11, 3, 11};
    vector<int> result2 = findKFrequentNumbers(nums2, 2);
    cout << "Here are the K frequent numbers: ";
    for (int n : result2) cout << n << " ";
    cout << endl;

    return 0;
}
```

## Frequency Sort (medium)

https://leetcode.com/problems/sort-characters-by-frequency/

> Given a string, sort it based on the decreasing frequency of its characters.

```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>
#include <algorithm>
using namespace std;

string sortCharacterByFrequency(string str) {
    unordered_map<char, int> counts;
    for (char c : str) {
        counts[c]++;
    }

    vector<pair<int, char>> freqArray;
    for (auto& p : counts) {
        freqArray.push_back({p.second, p.first});
    }

    // Sort by frequency in descending order
    sort(freqArray.begin(), freqArray.end(), greater<pair<int, char>>());

    string sortedString = "";
    for (auto& p : freqArray) {
        for (int i = 0; i < p.first; i++) {
            sortedString += p.second;
        }
    }

    return sortedString;
}

int main() {
    cout << "String after sorting characters by frequency: " << sortCharacterByFrequency("Programming") << endl;
    // 'r', 'g', and 'm' appeared twice, so they need to appear before any other character.

    cout << "String after sorting characters by frequency: " << sortCharacterByFrequency("abcbab") << endl;
    // 'b' appeared three times, 'a' appeared twice, and 'c' appeared only once.

    return 0;
}
```

## Kth Largest Number in a Stream (medium)

https://leetcode.com/problems/kth-largest-element-in-a-stream/

> Design a class to efficiently find the `Kth` largest element in a stream of numbers.
>
> The class should have the following two things:
>
> 1. The constructor of the class should accept an integer array containing initial numbers from the stream and an integer `K`.
> 2. The class should expose a function `add(num)` which will store the given number and return the <b>Kth largest</b> number.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

class KthLargest {
public:
    vector<int> nums;
    int k;

    KthLargest(int k, vector<int> nums) {
        this->k = k;
        this->nums = nums;
    }

    int add(int val) {
        sort(nums.begin(), nums.end());
        nums.erase(nums.end() - k, nums.end());

        if (val > nums[0] || nums.size() < k) {
            // Binary search for insertion position
            int start = 0;
            int end = k;
            int insertPos = start;

            while (start < end) {
                int mid = start + (end - start) / 2;
                if (nums[mid] == val) insertPos = mid;
                else if (nums[mid] < val) {
                    start = mid + 1;
                } else {
                    end = mid;
                }
            }
            insertPos = start;

            while (nums.size() > k) nums.erase(nums.begin());
            nums.insert(nums.begin() + insertPos, val);
        }

        if (nums.size() >= k) return nums[nums.size() - k];
        else return -1;
    }
};

int main() {
    // Input: [3, 1, 5, 12, 2, 11], K = 4
    KthLargest kthLargest(4, {3, 1, 5, 12, 2, 11});
    cout << kthLargest.add(6) << endl;    // return 5
    cout << kthLargest.add(13) << endl;   // return 6
    cout << kthLargest.add(4) << endl;    // return 6

    return 0;
}
```

- The time complexity of the above algorithm is `O(NlogN)`.
- The space complexity of the above algorithm is `O(1)`

## 'K' Closest Numbers (medium)

https://leetcode.com/problems/find-k-closest-elements/

> Given a sorted number array and two integers `K` and `X`, find `K` closest numbers to `X` in the array. Return the numbers in the sorted order. `X` is not necessarily present in the array.

This problem follows the [Top `K` Numbers](#top-k-numbers-easy) pattern. The biggest difference in this problem is that we need to find the closest (to `X`) numbers compared to finding the overall largest numbers. Another difference is that the given array is sorted.

Utilizing a similar approach, we can find the numbers closest to `X` through the following algorithm:

1. Since the array is sorted, we can first find the number closest to `X` through <b>Binary Search</b>. Let's say that number is `Y`.
2. The `K` closest numbers to `Y` will be adjacent to `Y` in the array. We can search in both directions of `Y` to find the closest numbers.
3. We can use a <s>heap</s> to efficiently search for the closest numbers. We will take `K` numbers in both directions of `Y` and push them in a <s>Min Heap</s> sorted by their absolute difference from `X`. This will ensure that the numbers with the smallest difference from `X` (i.e., closest to `X`) can be extracted easily from <s>Min Heap</s>.
4. Finally, we will extract the top `K` numbers from the <s>Min Heap</s> to find the required numbers.

After finding the number closest to `X` through <b>Binary Search</b>, we can use the <b>[Two Pointers](https://github.com/Chanda-Abdul/Several-Coding-Patterns-for-Solving-Data-Structures-and-Algorithms-Problems-during-Interviews/blob/main/%E2%9C%85%20%20Pattern%2002:%20Two%20Pointers.md)</b> approach to find the `K` closest numbers. Let’s say the closest number is `Y`. We can have a left pointer to move back from `Y` and a right pointer to move forward from `Y`. At any stage, whichever number pointed out by the left or the right pointer gives the smaller difference from `X` will be added to our result list.

To keep the resultant list sorted we can use a <b>Queue</b>. So whenever we take the number pointed out by the left pointer, we will append it at the beginning of the list and whenever we take the number pointed out by the right pointer we will append it at the end of the list.

Here is what our algorithm will look like:

```cpp
#include <iostream>
#include <vector>
#include <deque>
#include <algorithm>
using namespace std;

int binarySearchForClosest(vector<int>& arr, int X) {
    int lo = 0, hi = arr.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == X) return mid;
        if (arr[mid] < X) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    if (lo > 0) return lo - 1;
    return lo;
}

vector<int> findClosestElements(vector<int>& arr, int K, int X) {
    deque<int> windowOfKClosest;
    int idx = binarySearchForClosest(arr, X);
    int windowStart = idx;
    int windowEnd = idx + 1;
    int n = arr.size();

    for (int i = 0; i < K; i++) {
        if (windowStart >= 0 && windowEnd < n) {
            int diffFromStart = abs(X - arr[windowStart]);
            int diffFromEnd = abs(X - arr[windowEnd]);

            if (diffFromStart <= diffFromEnd) {
                windowOfKClosest.push_front(arr[windowStart]);
                windowStart--;
            } else {
                windowOfKClosest.push_back(arr[windowEnd]);
                windowEnd++;
            }
        } else if (windowStart >= 0) {
            windowOfKClosest.push_front(arr[windowStart]);
            windowStart--;
        } else if (windowEnd < n) {
            windowOfKClosest.push_back(arr[windowEnd]);
            windowEnd++;
        }
    }

    vector<int> result(windowOfKClosest.begin(), windowOfKClosest.end());
    return result;
}

int main() {
    vector<int> arr1 = {5, 6, 7, 8, 9};
    vector<int> result1 = findClosestElements(arr1, 3, 7);
    cout << "'K' closest numbers to 'X' are: ";
    for (int x : result1) cout << x << " ";
    cout << endl;  // [6, 7, 8]

    vector<int> arr2 = {2, 4, 5, 6, 9};
    vector<int> result2 = findClosestElements(arr2, 3, 6);
    cout << "'K' closest numbers to 'X' are: ";
    for (int x : result2) cout << x << " ";
    cout << endl;  // [4, 5, 6]

    return 0;
}
```

console.log(
`'K' closest numbers to 'X' are: ${findClosestElements(
    [5, 6, 7, 8, 9],
    3,
    7
  )}`
);
//Output: [6, 7, 8]

console.log(
`'K' closest numbers to 'X' are: ${findClosestElements(
    [2, 4, 5, 6, 9],
    3,
    6
  )}`
);
//Output: [4, 5, 6]

console.log(
`'K' closest numbers to 'X' are: ${findClosestElements(
    [2, 4, 5, 6, 9],
    3,
    10
  )}`
);
//Output: [5, 6, 9]

console.log(
`'K' closest numbers to 'X' are: ${findClosestElements(
    [1, 2, 3, 4, 5],
    4,
    3
  )}`
);
//Output: [1,2,3,4]

console.log(
`'K' closest numbers to 'X' are: ${findClosestElements(
    [1, 2, 3, 4, 5],
    4,
    -1
  )}`
);
//Output: [1,2,3,4]

````

- The time complexity of the above algorithm is `O(logN + K)`. We need `O(logN)` for <b>Binary Search</b> and `O(K)`for finding the `K` closest numbers using the two pointers.
- If we ignoring the space required for the output list, the algorithm runs in constant space `O(1)`.

## Maximum Distinct Elements (medium)

https://leetcode.com/problems/least-number-of-unique-integers-after-k-removals/

> Given an array of numbers and a number `K`, we need to remove `K` numbers from the array such that we are left with maximum distinct numbers.

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>
using namespace std;

int findMaximumDistinctElements(vector<int>& nums, int k) {
    unordered_map<int, int> freqMap;

    for (int num : nums) {
        freqMap[num]++;
    }

    vector<int> freq;
    for (auto& p : freqMap) {
        freq.push_back(p.second);
    }

    sort(freq.begin(), freq.end());

    int results = freq.size();
    for (int n : freq) {
        if (k >= n) {
            k -= n;
            results--;
        } else {
            return results;
        }
    }

    return results;
}

int main() {
    vector<int> arr1 = {5, 5, 4};
    cout << "Maximum distinct numbers after removing K numbers: "
         << findMaximumDistinctElements(arr1, 1) << endl;
    // 1, Remove the single 4, only 5 is left.

    vector<int> arr2 = {4, 3, 1, 1, 3, 3, 2};
    cout << "Maximum distinct numbers after removing K numbers: "
         << findMaximumDistinctElements(arr2, 3) << endl;
    // 2

    vector<int> arr3 = {3, 5, 12, 11, 12};
    cout << "Maximum distinct numbers after removing K numbers: "
         << findMaximumDistinctElements(arr3, 3) << endl;
    // 2

    return 0;
}
````

console.log(`Maximum distinct numbers after removing K numbers: 
${findMaximumDistinctElements([5, 5, 4], 1)}`);
//1, Remove the single 4, only 5 is left.

console.log(`Maximum distinct numbers after removing K numbers: 
${findMaximumDistinctElements([4, 3, 1, 1, 3, 3, 2], 3)}`);
//2, Remove 4, 2 and either one of the two 1s or three 3s. 1 and 3 will be left.

console.log(`Maximum distinct numbers after removing K numbers: 
${findMaximumDistinctElements([7, 3, 5, 8, 5, 3, 3], 2)}`);
//3, We can remove two occurrences of 3 to be left with 3 distinct numbers [7, 3, 8],
//we have to skip 5 because it is not distinct and appeared twice.
// Another solution could be to remove one instance of '5' and '3' each to be left with
//three distinct numbers [7, 5, 8], in this case, we have to skip 3 because it appeared twice.

console.log(`Maximum distinct numbers after removing K numbers: 
${findMaximumDistinctElements([3, 5, 12, 11, 12], 3)}`);
//2, We can remove one occurrence of 12, after which all numbers will become distinct.
//Then we can delete any two numbers which will leave us 2 distinct numbers in the result.

console.log(`Maximum distinct numbers after removing K numbers: 
${findMaximumDistinctElements([1, 2, 3, 3, 3, 3, 4, 4, 5, 5, 5], 2)}`); //3, We can remove one occurrence of '4' to get three distinct numbers.

````

- Since we will insert all numbers in a <b>HashMap</b>, this will take `O(N*logN)` where `N` is the total input numbers. While extracting numbers from the map, in the worst case, we will need to take out `K` numbers. This will happen when we have at least `K` numbers with a frequency of two. Since the <s>heap</s> can have a maximum of `N/2` numbers, therefore, extracting an element from the <s>heap</s> will take `O(logN)` and extracting `K` numbers will take `O(KlogN)`. So overall, the time complexity of our algorithm will be `O(N*logN + KlogN)`. We can optimize the above algorithm and only push `K` elements in the <s>heap</s>, as in the worst case we will be extracting `K` elements from the <s>heap</s>. This optimization will reduce the overall time complexity to `O(N*logK + KlogK)`.
- The space complexity will be `O(N)` as, in the worst case, we need to store all the `N` characters in the <b>HashMap</b>.

## Sum of Elements (medium)

https://www.geeksforgeeks.org/sum-elements-k1th-k2th-smallest-elements/

> Given an array, find the sum of all numbers between the `K1th` and `K2th` smallest elements of that array.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findSumOfElements(vector<int>& nums, int k1, int k2) {
    sort(nums.begin(), nums.end());

    vector<int> kSlice(nums.begin() + k1, nums.begin() + k2 - 1);
    int kSumBetween = 0;

    for (int n : kSlice) {
        kSumBetween += n;
    }

    return kSumBetween;
}

int main() {
    vector<int> arr1 = {1, 3, 12, 5, 15, 11};
    cout << "Sum of all numbers between k1 and k2 smallest numbers: "
         << findSumOfElements(arr1, 3, 6) << endl;
    // 23, The 3rd smallest number is 5 and 6th smallest number 15.

    vector<int> arr2 = {3, 5, 8, 7};
    cout << "Sum of all numbers between k1 and k2 smallest numbers: "
         << findSumOfElements(arr2, 1, 4) << endl;
    // 12

    return 0;
}
````

console.log(
`Sum of all numbers between k1 and k2 smallest numbers: ${findSumOfElements(
    [1, 3, 12, 5, 15, 11],
    3,
    6
  )}`
);
//23
//The 3rd smallest number is 5 and 6th smallest number 15.
// The sum of numbers coming between 5 and 15 is 23 (11+12).

console.log(
`Sum of all numbers between k1 and k2 smallest numbers: ${findSumOfElements(
    [3, 5, 8, 7],
    1,
    4
  )}`
);
//12
//The sum of the numbers between the 1st smallest number (3)
//and the 4th smallest number (8) is 12 (5+7).

````

## Rearrange String (hard)

https://leetcode.com/problems/reorganize-string/

> Given a string, find if its letters can be rearranged in such a way that no two same characters come next to each other.

```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <vector>
#include <algorithm>
using namespace std;

string rearrangeString(string str) {
    unordered_map<char, int> strMap;

    for (char c : str) {
        strMap[c]++;
    }

    // Convert map to vector and sort by frequency
    vector<pair<int, char>> freq;
    for (auto& p : strMap) {
        freq.push_back({p.second, p.first});
    }
    sort(freq.begin(), freq.end(), greater<pair<int, char>>());

    // Check if first character count > half of string length
    if (freq[0].first > (str.length() + 1) / 2) return "";

    string result(str.length(), ' ');
    int index = 0;

    for (auto& p : freq) {
        int value = p.first;
        char key = p.second;

        while (value--) {
            result[index] = key;
            index += 2;
            if (index >= str.length()) index = 1;
        }
    }

    return result;
}

int main() {
    cout << "Rearranged string: " << rearrangeString("aappp") << endl;
    cout << "Rearranged string: " << rearrangeString("Programming") << endl;
    cout << "Rearranged string: " << rearrangeString("aapa") << endl;

    return 0;
}
````

console.log(`Rearranged string: ${rearrangeString("aappp")}`);
console.log(`Rearranged string: ${rearrangeString("Programming")}`);
console.log(`Rearranged string: ${rearrangeString("aapa")}`);

````

## 🌟 Rearrange String K Distance Apart (hard)

https://leetcode.com/problems/rearrange-string-k-distance-apart/

> Given a string and a number `K`, find if the string can be rearranged such that the same characters are at least `K` distance apart from each other.

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

string rearrangeStringKDistance(string str, int k) {
    if (str.length() < 2 || k == 0) return str;

    vector<int> buckets(26, 0);
    for (char c : str) {
        buckets[c - 'a']++;
    }

    string res = "";
    vector<bool> added(26, false);

    while (res.length() < str.length()) {
        int maxIndex = -1;
        for (int i = 0; i < 26; i++) {
            if (buckets[i] && !added[i] &&
                (maxIndex == -1 || buckets[i] > buckets[maxIndex])) {
                maxIndex = i;
            }
        }

        if (maxIndex == -1) return "";

        res += (char)('a' + maxIndex);
        buckets[maxIndex]--;

        // Reset added array every k iterations
        fill(added.begin(), added.end(), false);
        for (int i = max(0, (int)res.length() - k); i < res.length() - 1; i++) {
            added[res[i] - 'a'] = true;
        }
    }

    return res;
}

int main() {
    cout << "Reorganized string: " << rearrangeStringKDistance("aabbcc", 3) << endl;
    // "abcabc" or similar

    cout << "Reorganized string: " << rearrangeStringKDistance("aaabc", 3) << endl;
    // ""

    cout << "Reorganized string: " << rearrangeStringKDistance("aaadbbcc", 2) << endl;
    // "abacabcd" or similar

    return 0;
}
````

console.log(`Reorganized string: ${rearrangeString('aabbcc', 3)}`);
//"abcabc"
//The same letters are at least a distance of 3 from each other.

console.log(`Reorganized string: ${rearrangeString('aaabc', 3)}`);
//""
//It is not possible to rearrange the string.

console.log(`Reorganized string: ${rearrangeString('aaadbbcc', 2)}`);
//"abacabcd"
//The same letters are at least a distance of 2 from each other.

console.log(`Reorganized string: ${rearrangeString('Programming', 3)}`);
//"rgmPrgmiano" or "gmringmrPoa" or "gmrPagimnor" and a few more
//All same characters are 3 distance apart.

console.log(`Reorganized string: ${rearrangeString('mmpp', 2)}`);
//"mpmp" or "pmpm"
//All same characters are 2 distance apart.

console.log(`Reorganized string: ${rearrangeString('aab', 2)}`);
//"aba"
//All same characters are 2 distance apart.

`console.log(`Reorganized string: ${rearrangeString('aapa', 3)}`);
//""
//We cannot find an arrangement of the string where any two 'a' are 3 distance apart.

```

## 🌟 🔎 Scheduling Tasks (hard)

https://leetcode.com/problems/task-scheduler/

> You are given a list of `tasks` that need to be run, in any order, on a server. Each `task` will take one CPU interval to execute but once a `task` has finished, it has a cooling period during which it cant be run again. If the cooling period for all tasks is `K` intervals, find the minimum number of CPU intervals that the server needs to finish all `tasks`.
>
> If at any time the server can't execute any `task` then it must stay `idle`.

This problem follows the Top `K` Elements pattern and is quite similar to Rearrange [String K Distance Apart](#-rearrange-string-k-distance-apart-hard). We need to rearrange tasks such that same tasks are `K` distance apart.


### A mental model for solving this problem

❗ Explaination referenced [here](https://leetcode.com/problems/task-scheduler/discuss/1874475/Easy-Solution-with-Writeup)

Consider the following input:

```

tasks = ["A", "A", "A", "B", "B", "B", "C", "C", "C", "D", "D", "E"]
n = 2

```

- First let's find the most frequently occuring task and lay it out like so (an underscore represents a cooldown period):

```

A \_ _ A _ \_ A

```

- Our goal is to insert all the other tasks into this sequence by filling the cooldown periods first.

- We know any other task can be scheduled into this sequence without creating additional cooldown slots because:

1. The number of occurrences of any task `x` is less than or equal to the number occurences of the most frequrntly occuring task `A`; and
2. We can always find a sequence of `k` different tasks to schedule between any pair of tasks `x`.

- Let's insert in the second most frequently occuring task `B` (we fill the two cooldown slots first and append the third task to the end):

```

A B _ A B _ A B

```

- Next let's insert in the third most frequently occuring task `C` (we fill in the two cooldown slots first and append the third task to the end):

```

A B C A B C A B C

```

- Next let's insert the fourth most frequently occuring task `D`. At this point we've used up all our cooldown slots so we can insert these tasks pretty much anywhere we like as long as there are at least `k` tasks between every pair of `D` tasks.

```

D A B D C A B C A B C

```

- Lastly, the least frequently occuring task `E` can be inserted literally anywhere we like. Because it only occurs once there is no cooldown period we need to respect.

```

D A B D C A B C A E B C

```

The sequence above is the shortest possible sequence these tasks can be scheduled in.
<b>Note</b> that there are multiple possible sequences of this length.

The answer to the problem is the number of `tasks` + the number of cooldown periods.

### Greedy HashMap Solution (Recommended)
}

console.log(
  `Minimum intervals needed to execute all tasks: ${scheduleTasks(
    ["a", "a", "a", "b", "c", "c"],
    2
  )}`
);
//7
//a -> c -> b -> a -> c -> idle -> a
console.log(
  `Minimum intervals needed to execute all tasks: ${scheduleTasks(
    ["a", "b", "a"],
    3
  )}`
);
//5
//a -> b -> idle -> idle -> a
```

- The time complexity of the above algorithm is `O(N∗logN)`
  where `N` is the number of tasks. Our while loop will iterate once for each occurrence of the task in the input (i.e. `N`) and in each iteration we will remove a task from the <b>heap</b> which will take `O(logN)`time. Hence the overall time complexity of our algorithm is `O(N*logN)`.
- The space complexity will be `O(N)`, as in the worst case, we need to store all the `N` tasks in the <b>HashMap</b>.

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <algorithm>
using namespace std;

int scheduleTasks(vector<char>& tasks, int k) {
    unordered_map<char, int> taskFreqMap;

    for (char c : tasks) {
        taskFreqMap[c]++;
    }

    int intervalMax = 0;
    int taskCountMax = 0;

    for (auto& p : taskFreqMap) {
        if (p.second > intervalMax) {
            intervalMax = p.second;
            taskCountMax = 1;
        } else if (p.second == intervalMax) {
            taskCountMax++;
        }
    }

    return max((int)tasks.size(), (intervalMax - 1) * (k + 1) + taskCountMax);
}

int main() {
    vector<char> tasks1 = {'A', 'A', 'A', 'B', 'B', 'B'};
    cout << "Minimum intervals needed to execute all tasks: "
         << scheduleTasks(tasks1, 2) << endl;
    // 8

    vector<char> tasks2 = {'A', 'A', 'A', 'B', 'B', 'B'};
    cout << "Minimum intervals needed to execute all tasks: "
         << scheduleTasks(tasks2, 0) << endl;
    // 6

    vector<char> tasks3 = {'A', 'A', 'A', 'A', 'A', 'A', 'B', 'C', 'D', 'E', 'F', 'G'};
    cout << "Minimum intervals needed to execute all tasks: "
         << scheduleTasks(tasks3, 2) << endl;
    // 16

    return 0;
}
```

console.log(
`Minimum intervals needed to execute all tasks: ${scheduleTasks(
    ["A", "A", "A", "B", "B", "B"],
    2
  )}`
);
// 8
// A -> B -> idle -> A -> B -> idle -> A -> B
// There is at least 2 units of time between any two same tasks.

console.log(
`Minimum intervals needed to execute all tasks: ${scheduleTasks(
    ["A", "A", "A", "B", "B", "B"],
    0
  )}`
);
// 6
// On this case any permutation of size 6 would work since n = 0.
// ["A","A","A","B","B","B"]
// ["A","B","A","B","A","B"]
// ["B","B","B","A","A","A"]

console.log(
`Minimum intervals needed to execute all tasks: ${scheduleTasks(
    ["A", "A", "A", "A", "A", "A", "B", "C", "D", "E", "F", "G"],
    2
  )}`
);
// 16
// One possible solution is
// A -> B -> C -> A -> D -> E -> A -> F -> G -> A
// -> idle -> idle -> A -> idle -> idle -> A

````

## 🌟Frequency Stack (hard)

https://leetcode.com/problems/maximum-frequency-stack/

> Design a class that simulates a Stack data structure, implementing the following two operations:
>
> - `push(int num)`: Pushes the number `num` on the stack.
> - `pop()`: Returns the most frequent number in the stack. If there is a tie, return the number which was pushed later.

### Frequency Map & Stack Solution

❗ Explaination referenced [here](https://leetcode.com/problems/maximum-frequency-stack/discuss/1086543/JS-Python-Java-C%2B%2B-or-Frequency-Map-and-Stack-Solution-w-Explanation)

There are many ways to solve this problem, but the description gives us two clues as to the most efficient way to do so.

- First, any time the word <i>"frequency"</i> is used, we're most likely going to need to make a <b>frequency map</b>.
- Second, they use the word <i>"stack"</i> in the title, so we should look at the possibility of a <b>stack</b> solution.

In this instance, we should consider a <b>2D stack</b>, with frequency on one side and input order on the other. This <b>stack</b> will hold each individual instance of a `value` pushed separately by what the frequency was at the time of insertion.

`freqStack` will work here because it starts at <b>1</b> and will increment from there. If we remember to `pop()` off unused frequencies, then the top of the frequency dimension of our <b>stack</b> `stack[stack.length-1]` will always represent the most frequent element, while the top of the input order dimension will represent the most recently seen value.

Our frequency map `freqMap()` will be used to keep track of the current frequencies of seen elements, so we know where to enter new ones into our stack.

### Implementation:

Since our frequencies are <b>1-indexed</b> and the <b>stack</b> is <b>0-indexed</b>, we have to insert a dummy <b>0-index</b> for all languages except <i>Javascript</i>, which lets you directly access even undefined array elements by index.

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <stack>
using namespace std;

class FreqStack {
private:
    unordered_map<int, int> freqMap;
    vector<stack<int>> freqStack;

public:
    FreqStack() {
        freqStack.push_back(stack<int>());  // dummy stack at index 0
    }

    void push(int value) {
        int freq = freqMap[value] + 1;
        freqMap[value] = freq;

        // Resize freqStack if needed
        while (freqStack.size() <= freq) {
            freqStack.push_back(stack<int>());
        }

        freqStack[freq].push(value);
    }

    int pop() {
        int freq = freqStack.size() - 1;
        int value = freqStack[freq].top();
        freqStack[freq].pop();

        if (freqStack[freq].empty()) {
            freqStack.pop_back();
        }

        freqMap[value]--;
        return value;
    }
};

int main() {
    FreqStack freqStack;
    freqStack.push(5);  // The stack is [5]
    freqStack.push(7);  // The stack is [5,7]
    freqStack.push(5);  // The stack is [5,7,5]
    freqStack.push(7);  // The stack is [5,7,5,7]
    freqStack.push(4);  // The stack is [5,7,5,7,4]
    freqStack.push(5);  // The stack is [5,7,5,7,4,5]

    cout << freqStack.pop() << endl;  // 5
    cout << freqStack.pop() << endl;  // 7
    cout << freqStack.pop() << endl;  // 5
    cout << freqStack.pop() << endl;  // 4

    return 0;
}
````

// Input
// ["FreqStack", "push", "push", "push", "push", "push", "push", "pop", "pop", "pop", "pop"]
// [[], [5], [7], [5], [7], [4], [5], [], [], [], []]
// Output
// [null, null, null, null, null, null, null, 5, 7, 5, 4]

// Explanation
let freqStack = new FreqStack();
freqStack.push(5); // The stack is [5]
freqStack.push(7); // The stack is [5,7]
freqStack.push(5); // The stack is [5,7,5]
freqStack.push(7); // The stack is [5,7,5,7]
freqStack.push(4); // The stack is [5,7,5,7,4]
freqStack.push(5); // The stack is [5,7,5,7,4,5]

freqStack.pop();
// return 5, as 5 is the most frequent. The stack becomes [5,7,5,7,4].
freqStack.pop();
// return 7, as 5 and 7 is the most frequent, but 7 is closest to the top.
//The stack becomes [5,7,5,4].
freqStack.pop();
// return 5, as 5 is the most frequent. The stack becomes [5,7,4].
freqStack.pop();
// return 4, as 4, 5 and 7 is the most frequent, but 4 is closest to the top. The stack becomes [5,7].

let frequencyStack = new FreqStack();
frequencyStack.push(1);
frequencyStack.push(2);
frequencyStack.push(3);
frequencyStack.push(2);
frequencyStack.push(1);
frequencyStack.push(2);
frequencyStack.push(5);
frequencyStack.pop();
// return 2, as it is the most frequent number
frequencyStack.pop();
// should return 1
frequencyStack.pop();
// should return 2

```

- <b>Time Complexity</b> of `O(1)` for both `push()` and `pop()` operations.
- <b>Space Complexity</b> of `O(N)`, where `N` is the number of elements in the `FreqStack()`.
```

# Pattern 3: Fast & Slow pointers

The <b>Fast & Slow</b> pointer approach, also known as the <b>Hare & Tortoise algorithm</b>, is a pointer algorithm that uses two pointers which move through the array (or sequence/<b>LinkedList</b>) at different speeds. This approach is quite useful when dealing with cyclic <b>LinkedLists</b> or arrays.

By moving at different speeds (say, in a cyclic <b>LinkedList</b>), the algorithm proves that the two pointers are bound to meet. The <i>fast pointer</i> should catch the <i>slow pointer</i> once both the pointers are in a cyclic loop.

One of the famous problems solved using this technique was <b>Finding a <i>cycle</i> in a LinkedList</b>. Let’s jump onto this problem to understand the <b>Fast & Slow</b> pattern.

## LinkedList Cycle (easy)

https://leetcode.com/problems/linked-list-cycle/

> Given the head of a <b>Singly LinkedList</b>, write a function to determine if the <b>LinkedList</b> has a </b>cycle</b> in it or not.

Imagine two racers running in a circular racing track. If one racer is faster than the other, the faster racer is bound to catch up and cross the slower racer from behind. We can use this fact to devise an algorithm to determine if a <b>LinkedList</b> has a <i>cycle</i> in it or not.

Imagine we have a slow and a <i>fast pointer</i> to traverse the <b>LinkedList</b>. In each iteration, the <i>slow pointer</i> moves one step and the <i>fast pointer</i> moves two steps. This gives us two conclusions:

1. If the <b>LinkedList</b> doesn’t have a <i>cycle</i> in it, the <i>fast pointer</i> will reach the end of the <b>LinkedList</b> before the <i>slow pointer</i> to reveal that there is no <i>cycle</i> in the <b>LinkedList</b>.
2. The <i>slow pointer</i> will never be able to catch up to the <i>fast pointer</i> if there is no <i>cycle</i> in the <b>LinkedList</b>.

If the <b>LinkedList</b> has a cycle, the <i>fast pointer</i> enters the <i>cycle</i> first, followed by the <i>slow pointer</i>. After this, both pointers will keep moving in the <i>cycle</i> infinitely. If at any stage both of these pointers meet, we can conclude that the <b>LinkedList</b> has a <i>cycle</i> in it. Let’s analyze if it is possible for the two pointers to meet. When the <i>fast pointer</i> is approaching the <i>slow pointer</i> from behind we have two possibilities:

1. The <i>fast pointer</i> is one step behind the <i>slow pointer</i>.
2. The <i>fast pointer</i> is two steps behind the <i>slow pointer</i>.

All other distances between the fast and <i>slow pointers</i> will reduce to one of these two possibilities. Let’s analyze these scenarios, considering the <i>fast pointer</i> always moves first:

1. If the <i>fast pointer</i> is one step behind the <i>slow pointer</i>: The <i>fast pointer</i> moves two steps and the <i>slow pointer</i> moves one step, and they both meet.
2. If the <i>fast pointer</i> is two steps behind the <i>slow pointer</i>: The <i>fast pointer</i> moves two steps and the <i>slow pointer</i> moves one step. After the moves, the <i>fast pointer</i> will be one step behind the <i>slow pointer</i>, which reduces this scenario to the first scenario. This means that the two pointers will meet in the next iteration.

This concludes that the two pointers will definitely meet if the <b>LinkedList</b> has a cycle.

```cpp
#include <iostream>
using namespace std;

struct Node {
  int value;
  Node* next;
  explicit Node(int v, Node* n = nullptr) : value(v), next(n) {}
};

bool hasCycle(Node* head) {
  Node* slow = head;
  Node* fast = head;
  while (fast != nullptr && fast->next != nullptr) {
    fast = fast->next->next;
    slow = slow->next;
    if (slow == fast) {
      return true;
    }
  }
  return false;
}

int main() {
  Node* head = new Node(1);
  head->next = new Node(2);
  head->next->next = new Node(3);
  head->next->next->next = new Node(4);
  head->next->next->next->next = new Node(5);
  head->next->next->next->next->next = new Node(6);
  cout << "LinkedList has cycle: " << boolalpha << hasCycle(head) << "\n";

  head->next->next->next->next->next->next = head->next->next;
  cout << "LinkedList has cycle: " << boolalpha << hasCycle(head) << "\n";

  head->next->next->next->next->next->next = head->next->next->next;
  cout << "LinkedList has cycle: " << boolalpha << hasCycle(head) << "\n";
  return 0;
}
```

- Once the <i>slow pointer</i> enters the cycle, the <i>fast pointer</i> will meet the <i><i>slow pointer</i></i> in the same loop. Therefore, the time complexity of our algorithm will be `O(N)` where `N` is the total number of nodes in the <b>LinkedList</b>.
- The algorithm runs in constant space `O(1)`.

> Given the head of a LinkedList with a cycle, find the length of the cycle.

Once the fast and <i>slow pointers</i> meet, we can save the <i>slow pointer</i> and iterate the whole <i>cycle</i> with another pointer until we see the <i>slow pointer</i> again to find the length of the cycle.

```cpp
#include <iostream>
using namespace std;

struct Node {
  int value;
  Node* next;
  explicit Node(int v, Node* n = nullptr) : value(v), next(n) {}
};

int calculateCycleLength(Node* meet) {
  Node* current = meet;
  int cycleLength = 0;
  do {
    current = current->next;
    cycleLength++;
  } while (current != meet);
  return cycleLength;
}

int findCycleLength(Node* head) {
  Node* slow = head;
  Node* fast = head;
  while (fast != nullptr && fast->next != nullptr) {
    fast = fast->next->next;
    slow = slow->next;
    if (slow == fast) {
      return calculateCycleLength(slow);
    }
  }
  return 0;
}

int main() {
  Node* head = new Node(1);
  head->next = new Node(2);
  head->next->next = new Node(3);
  head->next->next->next = new Node(4);
  head->next->next->next->next = new Node(5);
  head->next->next->next->next->next = new Node(6);
  cout << "LinkedList has cycle length of: " << findCycleLength(head) << "\n";

  head->next->next->next->next->next->next = head->next->next;
  cout << "LinkedList has cycle length of: " << findCycleLength(head) << "\n";

  head->next->next->next->next->next->next = head->next->next->next;
  cout << "LinkedList has cycle length of: " << findCycleLength(head) << "\n";
  return 0;
}
```

- The above algorithm runs in `O(N)` time complexity and `O(1)` space complexity.

## Start of LinkedList Cycle (medium)

https://leetcode.com/problems/linked-list-cycle-ii/

> Given the head of a <b>Singly LinkedList</b> that contains a cycle, write a function to find the <b>starting node of the cycle</b>.

If we know the length of the <b>LinkedList</b> cycle, we can find the start of the <i>cycle</i> through the following steps:

1. Take two pointers. Let’s call them `pointer1` and `pointer2`.
2. Initialize both pointers to point to the start of the <b>LinkedList</b>.
3. We can find the length of the <b>LinkedList</b> <i>cycle</i> using the approach discussed in <b>LinkedList Cycle</b>. Let’s assume that the length of the <i>cycle</i> is `K` nodes.
4. Move `pointer2` ahead by `K` nodes.
5. Now, keep incrementing `pointer1` and `pointer2` until they both meet.
6. As `pointer2` is `K` nodes ahead of `pointer1`, which means, `pointer2` must have completed one loop in the <i>cycle</i> when both pointers meet. Their meeting point will be the start of the cycle.

```cpp
#include <iostream>
using namespace std;

struct Node {
  int value;
  Node* next;
  explicit Node(int v, Node* n = nullptr) : value(v), next(n) {}
};

int calculateCycleLength(Node* meet) {
  Node* current = meet;
  int length = 0;
  do {
    current = current->next;
    length++;
  } while (current != meet);
  return length;
}

Node* findStart(Node* head, int cycleLength) {
  Node* pointer1 = head;
  Node* pointer2 = head;
  while (cycleLength-- > 0) {
    pointer2 = pointer2->next;
  }
  while (pointer1 != pointer2) {
    pointer1 = pointer1->next;
    pointer2 = pointer2->next;
  }
  return pointer1;
}

Node* findCycleStart(Node* head) {
  int cycleLength = 0;
  Node* slow = head;
  Node* fast = head;
  while (fast != nullptr && fast->next != nullptr) {
    fast = fast->next->next;
    slow = slow->next;
    if (slow == fast) {
      cycleLength = calculateCycleLength(slow);
      break;
    }
  }
  if (cycleLength == 0) {
    return nullptr;
  }
  return findStart(head, cycleLength);
}

int main() {
  Node* head = new Node(1);
  head->next = new Node(2);
  head->next->next = new Node(3);
  head->next->next->next = new Node(4);
  head->next->next->next->next = new Node(5);
  head->next->next->next->next->next = new Node(6);

  head->next->next->next->next->next->next = head->next->next;
  cout << "LinkedList cycle start: " << findCycleStart(head)->value << "\n";

  head->next->next->next->next->next->next = head->next->next->next;
  cout << "LinkedList cycle start: " << findCycleStart(head)->value << "\n";

  head->next->next->next->next->next->next = head;
  cout << "LinkedList cycle start: " << findCycleStart(head)->value << "\n";
  return 0;
}
```

- As we know, finding the <i>cycle</i> in a <b>LinkedList</b> with `N` nodes and also finding the length of the <i>cycle</i> requires `O(N)`. Also, as we saw in the above algorithm, we will need `O(N)` to find the start of the cycle. Therefore, the overall time complexity of our algorithm will be `O(N)`.
- The algorithm runs in constant space `O(1)`.

## Happy Number (medium)

https://leetcode.com/problems/happy-number/

Any number will be called a <b>happy number</b> if, after repeatedly replacing it with a number equal to the <b>sum of the square of all of its digits, leads us to number `1`</b>. All other <b>(not-happy)</b> numbers will never reach `1`. Instead, they will be stuck in a <i>cycle</i> of numbers which does not include `1`.

The process, defined above, to find out if a number is a <b>happy number</b> or not, always ends in a cycle. If the number is a <b>happy number</b>, the process will be stuck in a <i>cycle</i> on number `1`, and if the number is not a <b>happy number</b> then the process will be stuck in a <i>cycle</i> with a set of numbers. As we saw in Example-2 while determining if `12` is a <b>happy number</b> or not, our process will get stuck in a <i>cycle</i> with the following numbers: `89 -> 145 -> 42 -> 20 -> 4 -> 16 -> 37 -> 58 -> 89`

We saw in the <b>LinkedList Cycle</b> problem that we can use the <b>Fast & Slow</b> pointers method to find a <i>cycle</i> among a set of elements. As we have described above, each number will definitely have a cycle. Therefore, we will use the same <i>fast</i> & <i>slow pointer</i> strategy to find the <i>cycle</i> and once the <i>cycle</i> is found, we will see if the <i>cycle</i> is stuck on number `1` to find out if the number is happy or not.

```cpp
#include <iostream>
using namespace std;

int findSquareSum(int num) {
  int sum = 0;
  while (num > 0) {
    int digit = num % 10;
    sum += digit * digit;
    num /= 10;
  }
  return sum;
}

bool findHappyNumber(int num) {
  int slow = num;
  int fast = num;
  while (true) {
    slow = findSquareSum(slow);
    fast = findSquareSum(findSquareSum(fast));
    if (slow == fast) {
      break;
    }
  }
  return slow == 1;
}
```

`findHappyNumber(23) // true`

`23` is a <b>happy number</b>, Here are the steps to find out that `23` is a <b>happy number</b>:

1. 2² + 3² = 4 + 9 = 13
2. 1² + 3² = 1 + 9 = 10
3. 1² + 0² = 1 + 0 = 1

`findHappyNumber(12)//false`

`12` is not a <b>happy number</b>, Here are the steps to find out that `12` is not a <b>happy number</b>:

1. 1²+2²= 1 + 4 = 5
2. 5² = 25
3. 2² + 5² = 4 + 25 = 29
4. 2² + 9² = 4 + 81 = 85
5. 8² + 5² = 64 + 25 = 89
6. 8² + 9² = 64 + 81 = 145
7. 1² + 4² + 5²= 1 + 16 + 25 = 42
8. 4² + 2² = 16 + 4 = 20
9. 2² + 0² = 4 + 0 = 4
10. 4²= 16
11. 1² + 6² = 1 + 36 = 37
12. 3² + 7² = 9 + 49 = 58
13. 5² + 8²= 25 + 64 = 89
    Step `13` leads us back to step `5` as the number becomes equal to `89’, this means that we can never reach `1`, therefore, `12` is not a <b>happy number</b>.

`findHappyNumber(19)//true`

`19` is a <b>happy number</b>, Here are the steps to find out that 19 is a <b>happy number</b>:

1. 1² + 9² = 82
2. 8² + 2² = 68
3. 6² + 8² = 100
4. 1² + 0² + 0² = 1

`findHappyNumber(2)//false`

`2` is not a <b>happy number</b>

- The time complexity of the algorithm is difficult to determine. However we know the fact that all <b>unhappy number</b>s eventually get stuck in the cycle: 4 -> 16 -> 37 -> 58 -> 89 -> 145 -> 42 -> 20 -> 4

This sequence behavior tells us two things:

1. If the number `N` is less than or equal to `1000`, then we reach the <i>cycle</i> or `1` in at most `1001` steps.
2. For `N > 1000`, suppose the number has `M` digits and the next number is `N1`. From the above Wikipedia link, we know that the sum of the squares of the digits of `N` is at most `9²M`, or `81M`(this will happen when all digits of `N` are `9`).

This means:

1. `N1 < 81M`
2. As we know `M = log(N+1)`
3. Therefore: `N1 < 81 * log(N+1) => N1 = O(logN)`

- This concludes that the above algorithm will have a time complexity of `O(logN)`.
- The algorithm runs in constant space `O(1)`.

## Middle of the LinkedList (easy)

https://leetcode.com/problems/middle-of-the-linked-list/

> Given the head of a <b>Singly LinkedList</b>, write a method to return the <b>middle node</b> of the <b>LinkedList</b>.
>
> If the total number of nodes in the <b>LinkedList</b> is even, return the second middle node.

One brute force strategy could be to first count the number of nodes in the <b>LinkedList</b> and then find the middle node in the second iteration. Can we do this in one iteration?

We can use the <b>Fast & Slow</b> pointers method such that the <i>fast pointer</i> is always twice the nodes ahead of the <i>slow pointer</i>. This way, when the <i>fast pointer</i> reaches the end of the <b>LinkedList</b>, the <i>slow pointer</i> will be pointing at the middle node.

```cpp
#include <iostream>
using namespace std;

struct Node {
  int value;
  Node* next;
  explicit Node(int v, Node* n = nullptr) : value(v), next(n) {}
};

Node* findMiddleOfLinkedList(Node* head) {
  Node* slow = head;
  Node* fast = head;
  while (fast != nullptr && fast->next != nullptr) {
    slow = slow->next;
    fast = fast->next->next;
  }
  return slow;
}

int main() {
  Node* head = new Node(1);
  head->next = new Node(2);
  head->next->next = new Node(3);
  head->next->next->next = new Node(4);
  head->next->next->next->next = new Node(5);
  cout << "Middle Node: " << findMiddleOfLinkedList(head)->value << "\n";

  head->next->next->next->next->next = new Node(6);
  cout << "Middle Node: " << findMiddleOfLinkedList(head)->value << "\n";

  head->next->next->next->next->next->next = new Node(7);
  cout << "Middle Node: " << findMiddleOfLinkedList(head)->value << "\n";
  return 0;
}
```

- The above algorithm will have a time complexity of `O(N)` where `N` is the number of nodes in the <b>LinkedList</b>.
- The algorithm runs in constant space `O(1)`.

## 🌟 Palindrome LinkedList (medium)

https://leetcode.com/problems/palindrome-linked-list/

> Given the head of a <b>Singly LinkedList</b>, write a method to check if the <b>LinkedList is a palindrome</b> or not.
>
> Your algorithm should use <b>constant space</b> and the input <b>LinkedList</b> should be in the original form once the algorithm is finished. The algorithm should have `O(N)` time complexity where `N` is the number of nodes in the <b>LinkedList</b>.

### Example 1:

```
Input: 2 -> 4 -> 6 -> 4 -> 2 -> null
Output: true
```

### Example 2:

```
Input: 2 -> 4 -> 6 -> 4 -> 2 -> 2 -> null
Output: false
```

As we know, a palindrome <b>LinkedList</b> will have nodes values that read the same backward or forward. This means that if we divide the <b>LinkedList</b> into two halves, the node values of the first half in the forward direction should be similar to the node values of the second half in the backward direction. As we have been given a Singly <b>LinkedList</b>, we can’t move in the backward direction. To handle this, we will perform the following steps:

1. We can use the <b>Fast & Slow pointers</b> method similar to <b>Middle of the LinkedList</b> to find the middle node of the <b>LinkedList</b>.
2. Once we have the middle of the <b>LinkedList</b>, we will reverse the second half.
3. Then, we will compare the first half with the reversed second half to see if the <b>LinkedList</b> represents a palindrome.
4. Finally, we will reverse the second half of the <b>LinkedList</b> again to revert and bring the <b>LinkedList</b> back to its original form.

```cpp
#include <iostream>
using namespace std;

struct Node {
  int value;
  Node* next;
  explicit Node(int v, Node* n = nullptr) : value(v), next(n) {}
};

Node* reverse(Node* head) {
  Node* prev = nullptr;
  while (head != nullptr) {
    Node* next = head->next;
    head->next = prev;
    prev = head;
    head = next;
  }
  return prev;
}

bool isPalindromicLinkedList(Node* head) {
  if (head == nullptr || head->next == nullptr) {
    return true;
  }

  Node* slow = head;
  Node* fast = head;
  while (fast != nullptr && fast->next != nullptr) {
    slow = slow->next;
    fast = fast->next->next;
  }

  Node* headSecondHalf = reverse(slow);
  Node* copyHeadSecondHalf = headSecondHalf;

  while (head != nullptr && headSecondHalf != nullptr) {
    if (head->value != headSecondHalf->value) {
      break;
    }
    head = head->next;
    headSecondHalf = headSecondHalf->next;
  }

  reverse(copyHeadSecondHalf);
  return head == nullptr || headSecondHalf == nullptr;
}

int main() {
  Node* head = new Node(2);
  head->next = new Node(4);
  head->next->next = new Node(6);
  head->next->next->next = new Node(4);
  head->next->next->next->next = new Node(2);
  cout << "Is palindrome: " << boolalpha << isPalindromicLinkedList(head) << "\n";

  head->next->next->next->next->next = new Node(2);
  cout << "Is palindrome: " << boolalpha << isPalindromicLinkedList(head) << "\n";
  return 0;
}
```

- The above algorithm will have a time complexity of `O(N)` where `N` is the number of nodes in the <b>LinkedList</b>.
- The algorithm runs in constant space `O(1)`.

## 🌟 Rearrange a LinkedList (medium)

https://leetcode.com/problems/reorder-list/

> Given the head of a Singly <b>LinkedList</b>, write a method to modify the <b>LinkedList</b> such that the <b>nodes from the second half of the <b>LinkedList</b> are inserted alternately to the nodes from the first half in reverse order</b>. So if the <b>LinkedList</b> has nodes `1 -> 2 -> 3 -> 4 -> 5 -> 6 -> null`, your method should return `1 -> 6 -> 2 -> 5 -> 3 -> 4 -> null`.
>
> Your algorithm should not use any extra space and the input <b>LinkedList</b> should be modified </i>in-place</i>.

### Example 1:

```
Input: 2 -> 4 -> 6 -> 8 -> 10 -> 12 -> null
Output: 2 -> 12 -> 4 -> 10 -> 6 -> 8 -> null
```

### Example 2:

```
Input: 2 -> 4 -> 6 -> 8 -> 10 -> null
Output: 2 -> 10 -> 4 -> 8 -> 6 -> null
```

This problem shares similarities with <b>Palindrome LinkedList</b>. To rearrange the given <b>LinkedList</b> we will follow the following steps:

1. We can use the <b>Fast & Slow pointers</b> method similar to <b>Middle of the <b>LinkedList</b></b> to find the middle node of the <b>LinkedList</b>.
2. Once we have the middle of the <b>LinkedList</b>, we will reverse the second half of the <b>LinkedList</b>.
3. Finally, we’ll iterate through the first half and the reversed second half to produce a <b>LinkedList</b> in the required order.

```cpp
#include <iostream>
using namespace std;

struct Node {
  int val;
  Node* next;
  explicit Node(int v, Node* n = nullptr) : val(v), next(n) {}

  void printList() {
    Node* temp = this;
    while (temp != nullptr) {
      cout << temp->val << " ";
      temp = temp->next;
    }
    cout << "\n";
  }
};

Node* reverse(Node* head) {
  Node* prev = nullptr;
  while (head != nullptr) {
    Node* next = head->next;
    head->next = prev;
    prev = head;
    head = next;
  }
  return prev;
}

void reorder(Node* head) {
  if (head == nullptr || head->next == nullptr) {
    return;
  }

  Node* slow = head;
  Node* fast = head;
  while (fast != nullptr && fast->next != nullptr) {
    slow = slow->next;
    fast = fast->next->next;
  }

  Node* headSecondHalf = reverse(slow);
  Node* headFirstHalf = head;

  while (headFirstHalf != nullptr && headSecondHalf != nullptr) {
    Node* temp = headFirstHalf->next;
    headFirstHalf->next = headSecondHalf;
    headFirstHalf = temp;

    temp = headSecondHalf->next;
    headSecondHalf->next = headFirstHalf;
    headSecondHalf = temp;
  }

  if (headFirstHalf != nullptr) {
    headFirstHalf->next = nullptr;
  }
}

int main() {
  Node* head = new Node(2);
  head->next = new Node(4);
  head->next->next = new Node(6);
  head->next->next->next = new Node(8);
  head->next->next->next->next = new Node(10);
  head->next->next->next->next->next = new Node(12);
  reorder(head);
  head->printList();
  return 0;
}
```

- The above algorithm will have a time complexity of `O(N)` where `N` is the number of nodes in the <b>LinkedList</b>.
- The algorithm runs in constant space `O(1)`.

## 🌟 Cycle in a Circular Array (hard)

https://leetcode.com/problems/circular-array-loop/

We are given an array containing positive and negative numbers. Suppose the array contains a number `M` at a particular index. Now, if `M` is positive we will move forward `M` indices and if `M` is negative move backwards `M` indices. You should assume that the <b>array is circular</b> which means two things:

1. If, while moving forward, we reach the end of the array, we will jump to the first element to continue the movement.
2. If, while moving backward, we reach the beginning of the array, we will jump to the last element to continue the movement.
   Write a method to determine <b>if the array has a cycle</b>. The <i>cycle</i> should have more than one element and should follow one direction which means the <i>cycle</i> should not contain both forward and backward movements.

### Example 1:

```
Input: [1, 2, -1, 2, 2]
Output: true
Explanation: The array has a cycle among indices: 0 -> 1 -> 3 -> 0
```

### Example 2:

```
Input: [2, 2, -1, 2]
Output: true
Explanation: The array has a cycle among indices: 1 -> 3 -> 1
```

### Example 3:

```
Input: [2, 1, -1, -2]
Output: false
Explanation: The array does not have any cycle.
```

This problem involves finding a <i>cycle</i> in the array and, as we know, the <b>Fast & Slow pointer</b> method is an efficient way to do that. We can start from each index of the array to find the cycle. If a number does not have a <i>cycle</i> we will move forward to the next element. There are a couple of additional things we need to take care of:

1. As mentioned in the problem, the <i>cycle</i> should have more than one element. This means that when we move a pointer forward, if the pointer points to the same element after the move, we have a one-element cycle. Therefore, we can finish our <i>cycle</i> search for the current element.

2. The other requirement mentioned in the problem is that the <i>cycle</i> should not contain both forward and backward movements. We will handle this by remembering the direction of each element while searching for the cycle. If the number is positive, the direction will be forward and if the number is negative, the direction will be backward. So whenever we move a pointer forward, if there is a change in the direction, we will finish our <i>cycle</i> search right there for the current element.

```cpp
#include <vector>
using namespace std;

int findNextIndex(const vector<int>& arr, bool isForward, int currentIndex) {
  bool direction = arr[currentIndex] >= 0;
  if (isForward != direction) {
    return -1;
  }

  int nextIndex = (currentIndex + arr[currentIndex]) % static_cast<int>(arr.size());
  if (nextIndex < 0) {
    nextIndex += static_cast<int>(arr.size());
  }
  if (nextIndex == currentIndex) {
    return -1;
  }
  return nextIndex;
}

bool circularArrayLoopExists(const vector<int>& arr) {
  for (int i = 0; i < static_cast<int>(arr.size()); i++) {
    bool isForward = arr[i] >= 0;
    int slow = i;
    int fast = i;

    while (true) {
      slow = findNextIndex(arr, isForward, slow);
      fast = findNextIndex(arr, isForward, fast);
      if (fast != -1) {
        fast = findNextIndex(arr, isForward, fast);
      }
      if (slow == -1 || fast == -1 || slow == fast) {
        break;
      }
    }

    if (slow != -1 && slow == fast) {
      return true;
    }
  }
  return false;
}

// circularArrayLoopExists({1, 2, -1, 2, 2})
// circularArrayLoopExists({2, 2, -1, 2})
// circularArrayLoopExists({2, 1, -1, -2})
```

- The above algorithm will have a time complexity of `O(N²)` where `N` is the number of elements in the array. This complexity is due to the fact that we are iterating all elements of the array and trying to find a <i>cycle</i> for each element.
- The algorithm runs in constant space `O(1)`.

#### An Alternate Approach

In our algorithm, we don’t keep a record of all the numbers that have been evaluated for <i>cycle</i> . We know that all such numbers will not produce a <i>cycle</i> for any other instance as well. If we can remember all the numbers that have been visited, our algorithm will improve to `O(N)` as, then, each number will be evaluated for <i>cycle</i> only once. We can keep track of this by creating a separate array, however, in this case, the space complexity of our algorithm will increase to `O(N)`.

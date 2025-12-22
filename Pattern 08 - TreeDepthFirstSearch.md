# Pattern 8: Tree Depth First Search (DFS)

This pattern is based on the <b>Depth First Search (DFS)</b> technique to traverse a tree.

We will be using <b>recursion</b> (or we can also use a stack for the iterative approach) to keep track of all the previous (parent) nodes while traversing. This also means that the space complexity of the algorithm will be `O(H)`, where `H` is the maximum height of the tree.

## Binary Tree Path Sum (easy)

https://leetcode.com/problems/path-sum/

> Given a binary tree and a number `S`, find if the tree has a path from root-to-leaf such that the sum of all the node values of that path equals `S`.

As we are trying to search for a <i>root-to-leaf path</i> , we can use the <b>Depth First Search (DFS)</b> technique to solve this problem.

To recursively traverse a binary tree in a <b>DFS</b> fashion, we can start from the root and at every step, make two recursive calls one for the left and one for the right child.

Here are the steps for our [Binary Tree Path Sum](#binary-tree-path-sum-easy) problem:

1. Start <b>DFS</b> with the root of the tree.
2. If the current node is not a leaf node, do two things:
   - Subtract the value of the current node from the given number to get a new <i>sum</i> => `S = S - node.value`
   - Make two recursive calls for both the children of the current node with the new number calculated in the previous step.
3. At every step, see if the current node being visited is a leaf node and if its value is equal to the given number `S`. If both these conditions are true, we have found the required <i>root-to-leaf path</i> , therefore return `true`.
4. If the current node is a leaf but its value is not equal to the given number `S`, return `false`.

```cpp
#include <iostream>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

bool hasPath(TreeNode* root, int sum) {
  if (root == nullptr) {
    return false;
  }

  if (root->value == sum && root->left == nullptr && root->right == nullptr) {
    return true;
  }

  return hasPath(root->left, sum - root->value) ||
         hasPath(root->right, sum - root->value);
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  cout << "Tree has path: " << (hasPath(root, 23) ? "true" : "false") << "\n";
  cout << "Tree has path: " << (hasPath(root, 16) ? "true" : "false") << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` in the worst case. This space will be used to store the <b>recursion</b> stack. The worst case will happen when the given tree is a <i>linked list</i> (i.e., every node has only one child).

## All Paths for a Sum (medium)

https://leetcode.com/problems/path-sum-ii/

> Given a binary tree and a number `S`, find all paths from root-to-leaf such that the sum of all the node values of each path equals `S`.

This problem follows the <b>[Binary Tree Path Sum](#binary-tree-path-sum-easy)</b> pattern. We can follow the same <b>DFS</b> approach. There will be two differences:

1. Every time we find a <i>root-to-leaf path</i> , we will store it in a list.
2. We will traverse all paths and will not stop processing after finding the first path.

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

void findAPath(TreeNode* currentNode, int sum, vector<int>& currentPath,
               vector<vector<int>>& allPaths) {
  if (currentNode == nullptr) {
    return;
  }

  currentPath.push_back(currentNode->value);

  if (currentNode->value == sum && currentNode->left == nullptr &&
      currentNode->right == nullptr) {
    allPaths.push_back(currentPath);
  } else {
    findAPath(currentNode->left, sum - currentNode->value, currentPath,
              allPaths);
    findAPath(currentNode->right, sum - currentNode->value, currentPath,
              allPaths);
  }

  currentPath.pop_back();
}

vector<vector<int>> findPaths(TreeNode* root, int sum) {
  vector<vector<int>> allPaths;
  vector<int> currentPath;
  findAPath(root, sum, currentPath, allPaths);
  return allPaths;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(4);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  findPaths(root, 23);
}
```

- The time complexity of the above algorithm is `O(N^2)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once (which will take `O(N)`), and for every leaf node, we might have to store its path (by making a copy of the current path) which will take `O(N)`.
  - We can calculate a tighter time complexity of `O(NlogN)` from the space complexity discussion below.
- If we ignore the space required for the `allPaths` list, the space complexity of the above algorithm will be `O(N)` in the worst case. This space will be used to store the <b>recursion</b> stack. The worst-case will happen when the given tree is a <i>linked list</i> (i.e., every node has only one child).

> 🌟 Given a binary tree, return all <i>root-to-leaf</i> paths.

https://leetcode.com/problems/binary-tree-paths/

We can follow a similar approach. We just need to remove the “check for the path sum.”

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

void findAPath(TreeNode* currentNode, vector<int>& currentPath,
               vector<vector<int>>& allPaths) {
  if (currentNode == nullptr) {
    return;
  }

  currentPath.push_back(currentNode->value);

  if (currentNode->left == nullptr && currentNode->right == nullptr) {
    allPaths.push_back(currentPath);
  } else {
    findAPath(currentNode->left, currentPath, allPaths);
    findAPath(currentNode->right, currentPath, allPaths);
  }

  currentPath.pop_back();
}

vector<vector<int>> findPaths(TreeNode* root) {
  vector<vector<int>> allPaths;
  vector<int> currentPath;
  findAPath(root, currentPath, allPaths);
  return allPaths;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(4);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  findPaths(root);
}
```

> 🌟 Given a binary tree, find the <i>root-to-leaf</i> path with the maximum sum.

We need to find the path with the maximum sum. As we traverse all paths, we can keep track of the path with the maximum sum.

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

long long maxSum = LLONG_MIN;

void findAPath(TreeNode* currentNode, vector<int>& currentPath) {
  if (currentNode == nullptr) {
    return;
  }

  currentPath.push_back(currentNode->value);

  if (currentNode->left == nullptr && currentNode->right == nullptr) {
    long long pathMax = 0;
    for (int val : currentPath) {
      pathMax += val;
    }
    maxSum = max(maxSum, pathMax);
  } else {
    findAPath(currentNode->left, currentPath);
    findAPath(currentNode->right, currentPath);
  }

  currentPath.pop_back();
}

long long maxPathSum(TreeNode* root) {
  vector<int> currentPath;
  findAPath(root, currentPath);
  return maxSum;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(4);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  maxPathSum(root);
}
```

## Sum of Path Numbers (medium)

https://leetcode.com/problems/sum-root-to-leaf-numbers/

> Given a binary tree where each node can only have a digit (0-9) value, each <i>root-to-leaf path</i> will represent a number. Find the total sum of all the numbers represented by all paths.

This problem follows the <b>[Binary Tree Path Sum](#binary-tree-path-sum-easy)</b> pattern. We can follow the same <b>DFS</b> approach. The additional thing we need to do is to keep track of the number representing the current path.

How do we calculate the path number for a node? Taking the first example mentioned above, say we are at node `7`. As we know, the path number for this node is `17`, which was calculated by: `1 * 10 + 7 => 17`. We will follow the same approach to calculate the path number of each node.

```cpp
#include <iostream>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v) : value(v), left(nullptr), right(nullptr) {}
};

long long findRootToLeafPathNumbers(TreeNode* currentNode,
                                     long long pathSum) {
  if (currentNode == nullptr) {
    return 0;
  }

  pathSum = 10 * pathSum + currentNode->value;

  if (currentNode->left == nullptr && currentNode->right == nullptr) {
    return pathSum;
  }

  return findRootToLeafPathNumbers(currentNode->left, pathSum) +
         findRootToLeafPathNumbers(currentNode->right, pathSum);
}

long long findSumOfPathNumbers(TreeNode* root) {
  return findRootToLeafPathNumbers(root, 0);
}

int main() {
  TreeNode* root = new TreeNode(1);
  root->left = new TreeNode(0);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(1);
  root->right->left = new TreeNode(6);
  root->right->right = new TreeNode(5);
  cout << "Total Sum of Path Numbers: " << findSumOfPathNumbers(root) << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` in the worst case. This space will be used to store the <b>recursion</b> stack. The worst case will happen when the given tree is a <i>linked list</i> (i.e., every node has only one child).

## Path With Given Sequence (medium)

> Given a binary tree and a number sequence, find if the sequence is present as a <i>root-to-leaf path</i> in the given tree.

This problem follows the <b>[Binary Tree Path Sum](#binary-tree-path-sum-easy)</b> pattern. We can follow the same <b>DFS</b> approach and additionally, track the element of the given sequence that we should match with the current node. Also, we can return false as soon as we find a mismatch between the sequence and the node value.

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v) : value(v), left(nullptr), right(nullptr) {}
};

bool findPathRecursive(TreeNode* currentNode, const vector<int>& sequence,
                       int sequenceIndex) {
  if (currentNode == nullptr) return false;

  const int sequenceLength = sequence.size();
  if (sequenceIndex >= sequenceLength ||
      currentNode->value != sequence[sequenceIndex])
    return false;

  if (currentNode->left == nullptr && currentNode->right == nullptr &&
      sequenceIndex == sequenceLength - 1)
    return true;

  return findPathRecursive(currentNode->left, sequence, sequenceIndex + 1) ||
         findPathRecursive(currentNode->right, sequence, sequenceIndex + 1);
}

bool findPath(TreeNode* root, const vector<int>& sequence) {
  if (root == nullptr) {
    return sequence.size() == 0;
  }

  return findPathRecursive(root, sequence, 0);
}

int main() {
  TreeNode* root = new TreeNode(1);
  root->left = new TreeNode(0);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(1);
  root->right->left = new TreeNode(6);
  root->right->right = new TreeNode(5);

  cout << "Tree has path sequence: "
       << (findPath(root, {1, 0, 7}) ? "true" : "false") << "\n";
  cout << "Tree has path sequence: "
       << (findPath(root, {1, 1, 6}) ? "true" : "false") << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` in the worst case. This space will be used to store the <b>recursion</b> stack. The worst case will happen when the given tree is a <i>linked list</i> (i.e., every node has only one child).

## Count Paths for a Sum (medium)

https://leetcode.com/problems/path-sum-iii/

> Given a binary tree and a number `S`, find all paths in the tree such that the sum of all the node values of each path equals `S`. Please note that the paths can start or end at any node but all paths must follow direction from parent to child (top to bottom).

This problem follows the <b>[Binary Tree Path Sum](#binary-tree-path-sum-easy)</b> pattern. We can follow the same <b>DFS</b> approach. But there will be four differences:

1. We will keep track of the current path in a list which will be passed to every recursive call.
2. Whenever we traverse a node we will do two things:

   - Add the current node to the current path.
   - As we added a new node to the current path, we should find the sums of all sub-paths ending at the current node. If the sum of any sub-path is equal to `S` we will increment our path count.

3. We will traverse all paths and will not stop processing after finding the first path.
4. Remove the current node from the current path before returning from the function. This is needed to <i>backtrack</i> while we are going up the recursive call stack to process other paths.

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

int countPathsRecursive(TreeNode* currentNode, int S,
                        vector<int>& currentPath) {
  if (currentNode == nullptr) return 0;

  currentPath.push_back(currentNode->value);

  int pathCount = 0;
  long long pathSum = 0;

  for (int i = static_cast<int>(currentPath.size()) - 1; i >= 0; i--) {
    pathSum += currentPath[i];
    if (pathSum == S) {
      pathCount++;
    }
  }

  pathCount +=
      countPathsRecursive(currentNode->left, S, currentPath) +
      countPathsRecursive(currentNode->right, S, currentPath);

  currentPath.pop_back();
  return pathCount;
}

int countPaths(TreeNode* root, int S) {
  vector<int> currentPath;
  return countPathsRecursive(root, S, currentPath);
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(4);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  cout << "Tree has " << countPaths(root, 11) << " paths\n";
}
```

- The time complexity of the above algorithm is `O(N²)` in the worst case, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once, but for every node, we iterate the current path. The current path, in the worst case, can be `O(N)` (in the case of a skewed tree). But, if the tree is balanced, then the current path will be equal to the height of the tree, i.e., `O(logN)`. So the best case of our algorithm will be `O(NlogN)`.
- The space complexity of the above algorithm will be `O(N)`. This space will be used to store the <b>recursion</b> stack. The worst case will happen when the given tree is a <i>linked list</i> (i.e., every node has only one child). We also need `O(N)` space for storing the currentPath in the worst case. Overall space complexity of our algorithm is `O(N)`.

## 🌟 Tree Diameter (medium)

https://leetcode.com/problems/diameter-of-binary-tree/

> Given a binary tree, find the length of its diameter. The diameter of a tree is the number of nodes on the <b>longest path between any two leaf nodes</b>. The diameter of a tree may or may not pass through the root.
>
> Note: You can always assume that there are at least two leaf nodes in the given tree.

This problem follows the <b>Binary Tree Path Sum</b> pattern. We can follow the same <b>DFS</b> approach. There will be a few differences:

1. At every step, we need to find the height of both children of the current node. For this, we will make two recursive calls similar to <b>DFS</b>.
2. The height of the current node will be equal to the maximum of the heights of its left or right children, plus `1` for the `currentNode`.
3. The tree diameter at the `currentNode` will be equal to the height of the left child plus the height of the right child plus `1` for the current node: `diameter = leftTreeHeight + rightTreeHeight + 1`. To find the overall tree diameter, we will use a class level variable. This variable will store the maximum `diameter` of all the nodes visited so far, hence, eventually, it will have the final tree diameter.

```cpp
#include <algorithm>
#include <iostream>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

class TreeDiameter {
 public:
  int treeDiameter = 0;

  int findDiameter(TreeNode* root) {
    calculateHeight(root);
    return treeDiameter;
  }

 private:
  int calculateHeight(TreeNode* currentNode) {
    if (currentNode == nullptr) return 0;

    int leftTreeHeight = calculateHeight(currentNode->left);
    int rightTreeHeight = calculateHeight(currentNode->right);

    if (leftTreeHeight != 0 && rightTreeHeight != 0) {
      int diameter = leftTreeHeight + rightTreeHeight + 1;
      treeDiameter = max(treeDiameter, diameter);
    }

    return max(leftTreeHeight, rightTreeHeight) + 1;
  }
};

int main() {
  TreeDiameter td;
  TreeNode* root = new TreeNode(1);
  root->left = new TreeNode(2);
  root->right = new TreeNode(3);
  root->left->left = new TreeNode(4);
  root->right->left = new TreeNode(5);
  root->right->right = new TreeNode(6);
  cout << "Tree Diameter: " << td.findDiameter(root) << "\n";

  root->left->left = nullptr;
  root->right->left->left = new TreeNode(7);
  root->right->left->right = new TreeNode(8);
  root->right->right->left = new TreeNode(9);
  root->right->left->right->left = new TreeNode(10);
  root->right->right->left->left = new TreeNode(11);
  cout << "Tree Diameter: " << td.findDiameter(root) << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` in the worst case. This space will be used to store the <b>recursion</b> stack. The worst case will happen when the given tree is a <i>linked list</i> (i.e., every node has only one child).

## 🌟 Path with Maximum Sum (hard)

https://leetcode.com/problems/binary-tree-maximum-path-sum/

> Find the path with the maximum sum in a given binary tree. Write a function that returns the maximum sum.
>
> A path can be defined as a <b>sequence of nodes between any two nodes</b> and doesn’t necessarily pass through the root. The path must contain at least one node.

This problem follows the <b>Binary Tree Path Sum</b> pattern and shares the algorithmic logic with <b>Tree Diameter</b>. We can follow the same <b>DFS</b> approach. The only difference will be to ignore the paths with negative sums. Since we need to find the overall maximum sum, we should ignore any path which has an overall negative sum.

```cpp
#include <algorithm>
#include <iostream>
#include <limits>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

long long globalMaximumSum = numeric_limits<long long>::min();

long long findMaximumPathSumRecursive(TreeNode* currentNode) {
  if (currentNode == nullptr) return 0;

  long long maxPathSumLeft = findMaximumPathSumRecursive(currentNode->left);
  long long maxPathSumRight = findMaximumPathSumRecursive(currentNode->right);

  maxPathSumLeft = max(maxPathSumLeft, 0LL);
  maxPathSumRight = max(maxPathSumRight, 0LL);

  long long localMaximumSum =
      maxPathSumLeft + maxPathSumRight + currentNode->value;

  globalMaximumSum = max(globalMaximumSum, localMaximumSum);

  return max(maxPathSumLeft, maxPathSumRight) + currentNode->value;
}

long long findMaximumPathSum(TreeNode* root) {
  globalMaximumSum = numeric_limits<long long>::min();
  findMaximumPathSumRecursive(root);
  return globalMaximumSum;
}

int main() {
  TreeNode* root = new TreeNode(1);
  root->left = new TreeNode(2);
  root->right = new TreeNode(3);
  cout << "Maximum Path Sum: " << findMaximumPathSum(root) << "\n";

  root->left->left = new TreeNode(1);
  root->left->right = new TreeNode(3);
  root->right->left = new TreeNode(5);
  root->right->right = new TreeNode(6);
  root->right->left->left = new TreeNode(7);
  root->right->left->right = new TreeNode(8);
  root->right->right->left = new TreeNode(9);
  cout << "Maximum Path Sum: " << findMaximumPathSum(root) << "\n";

  root = new TreeNode(-1);
  root->left = new TreeNode(-3);
  cout << "Maximum Path Sum: " << findMaximumPathSum(root) << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` in the worst case. This space will be used to store the <b>recursion</b> stack. The worst case will happen when the given tree is a <i>linked list</i> (i.e., every node has only one child).

###### #DFS #DepthFirstSearch #JavaScript #GrokkingTheCodingInterviewPatterns #LeetCode #DataStructures #Algorithms

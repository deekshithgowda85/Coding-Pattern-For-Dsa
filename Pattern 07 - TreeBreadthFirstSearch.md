# Pattern 7: Tree Breadth First Search

This pattern is based on the <b>Breadth First Search (BFS)</b> technique to traverse a tree.

Any problem involving the traversal of a tree in a level-by-level order can be efficiently solved using this approach. We will use a <b>Queue</b> to keep track of all the nodes of a level before we jump onto the next level. This also means that the space complexity of the algorithm will be `O(W)`, where `W` is the maximum number of nodes on any level.

## Binary Tree Level Order Traversal (easy)

https://leetcode.com/problems/binary-tree-level-order-traversal/

> Given a binary tree, populate an array to represent its level-by-level traversal. You should populate the values of all <b>nodes of each level from left to right</b> in separate sub-arrays.

Since we need to traverse all nodes of each level before moving onto the next level, we can use the <b>Breadth First Search (BFS)</b> technique to solve this problem.

We can use a <b>Queue</b> to efficiently traverse in <b>BFS</b> fashion. Here are the steps of our algorithm:

1. Start by pushing the `root` node to the queue.
2. Keep iterating until the <b>queue</b> is empty.
3. In each iteration, first count the elements in the <b>queue</b> (let’s call it `levelSize`). We will have these many nodes in the current level.
4. Next, remove `levelSize` nodes from the <b>queue</b> and push their `value` in an array to represent the current level.
5. After removing each node from the queue, insert both of its children into the queue.
6. If the <b>queue</b> is not empty, repeat from <i>step 3</i> for the next level.

```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v) : value(v), left(nullptr), right(nullptr) {}
};

vector<vector<int>> traverse(TreeNode* root) {
  vector<vector<int>> result;
  if (root == nullptr) {
    return result;
  }

  queue<TreeNode*> q;
  q.push(root);
  while (!q.empty()) {
    int levelSize = static_cast<int>(q.size());
    vector<int> currentLevel;
    currentLevel.reserve(levelSize);

    for (int i = 0; i < levelSize; i++) {
      TreeNode* node = q.front();
      q.pop();
      currentLevel.push_back(node->value);
      if (node->left != nullptr) {
        q.push(node->left);
      }
      if (node->right != nullptr) {
        q.push(node->right);
      }
    }
    result.push_back(currentLevel);
  }

  return result;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);

  vector<vector<int>> levels = traverse(root);
  cout << "Level order traversal:";
  for (const auto& level : levels) {
    cout << " [";
    for (size_t i = 0; i < level.size(); i++) {
      if (i > 0) cout << ",";
      cout << level[i];
    }
    cout << "]";
  }
  cout << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` as we need to return a list containing the level order traversal. We will also need `O(N)` space for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

### Easier to understand solution w/o `Deque()`

```cpp
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v = 0, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

vector<vector<int>> levelOrder(TreeNode* root) {
  if (root == nullptr) return {};

  vector<vector<int>> levels;
  queue<TreeNode*> q;
  q.push(root);

  while (!q.empty()) {
    int queueLength = static_cast<int>(q.size());
    vector<int> currLevel;
    currLevel.reserve(queueLength);

    for (int i = 0; i < queueLength; i++) {
      TreeNode* currNode = q.front();
      q.pop();

      if (currNode->left) q.push(currNode->left);
      if (currNode->right) q.push(currNode->right);
      currLevel.push_back(currNode->value);
    }
    levels.push_back(currLevel);
  }

  return levels;
}

int main() {
  TreeNode* root = new TreeNode(3);
  root->left = new TreeNode(9);
  root->right = new TreeNode(20);
  root->right->left = new TreeNode(15);
  root->right->right = new TreeNode(7);
  levelOrder(root);

  root = new TreeNode(1);
  levelOrder(root);

  root = new TreeNode();
  levelOrder(root);
}
```

## Reverse Level Order Traversal (easy)

https://leetcode.com/problems/binary-tree-level-order-traversal-ii/

> Given a binary tree, populate an array to represent its level-by-level traversal in reverse order, i.e., <b>the lowest level comes first</b>. You should populate the values of all nodes in each level from left to right in separate sub-arrays.

This problem follows the <b>Binary Tree Level Order Traversal</b> pattern. We can follow the same <b>BFS</b> approach. The only difference will be that instead of appending the current level at the end, we will append the current level at the beginning of the result list.

```cpp
#include <deque>
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

vector<vector<int>> reverseLevelOrder(TreeNode* root) {
  if (root == nullptr) return {};

  deque<vector<int>> levels;
  queue<TreeNode*> q;
  q.push(root);

  while (!q.empty()) {
    int queueLength = static_cast<int>(q.size());
    vector<int> currLevel;
    currLevel.reserve(queueLength);

    for (int i = 0; i < queueLength; i++) {
      TreeNode* currNode = q.front();
      q.pop();
      if (currNode->left) q.push(currNode->left);
      if (currNode->right) q.push(currNode->right);
      currLevel.push_back(currNode->value);
    }
    levels.push_front(currLevel);
  }

  return vector<vector<int>>(levels.begin(), levels.end());
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->left->right = new TreeNode(10);
  root->right->right = new TreeNode(5);
  reverseLevelOrder(root);
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` as we need to return a list containing the level order traversal. We will also need `O(N)` space for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

## 🌴 Zigzag Traversal (medium)

https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/

> Given a binary tree, populate an array to represent its zigzag level order traversal. You should populate the values of all <b>nodes of the first level from left to right</b>, then <b>right to left for the next level</b> and keep alternating in the same manner for the following levels.

This problem follows the <b>Binary Tree Level Order Traversal</b> pattern. We can follow the same <b>BFS</b> approach. The only additional step we have to keep in mind is to alternate the level order traversal, which means that for every other level, we will traverse similar to <b>[Reverse Level Order Traversal](#reverse-level-order-traversal-easy)</b>.

```cpp
#include <deque>
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v = 0, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
  if (root == nullptr) return {};

  vector<vector<int>> levels;
  queue<TreeNode*> q;
  q.push(root);
  bool leftToRight = true;

  while (!q.empty()) {
    int queueLength = static_cast<int>(q.size());
    deque<int> currLevel;

    for (int i = 0; i < queueLength; i++) {
      TreeNode* currNode = q.front();
      q.pop();

      if (leftToRight) {
        currLevel.push_back(currNode->value);
      } else {
        currLevel.push_front(currNode->value);
      }

      if (currNode->left) q.push(currNode->left);
      if (currNode->right) q.push(currNode->right);
    }

    levels.emplace_back(currLevel.begin(), currLevel.end());
    leftToRight = !leftToRight;
  }
  return levels;
}

int main() {
  TreeNode* root = new TreeNode(1);
  root->left = new TreeNode(2);
  root->right = new TreeNode(3);
  root->left->left = new TreeNode(4);
  root->left->right = new TreeNode(5);
  root->right->left = new TreeNode(6);
  root->right->right = new TreeNode(7);
  zigzagLevelOrder(root);

  root = new TreeNode(3);
  root->left = new TreeNode(9);
  root->right = new TreeNode(20);
  root->right->left = new TreeNode(15);
  root->right->right = new TreeNode(7);
  zigzagLevelOrder(root);

  root = new TreeNode(1);
  zigzagLevelOrder(root);

  root = new TreeNode();
  zigzagLevelOrder(root);
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` as we need to return a list containing the level order traversal. We will also need `O(N)` space for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

## Level Averages in a Binary Tree (easy)

https://leetcode.com/problems/average-of-levels-in-binary-tree/

> Given a binary tree, populate an array to represent the <b>averages of all of its levels</b>

This problem follows the <b>Binary Tree Level Order Traversal</b> pattern. We can follow the same <b>BFS</b> approach. The only difference will be that instead of keeping track of all nodes of a level, we will only track the running sum of the values of all nodes in each level. In the end, we will append the average of the current level to the result array.

```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

vector<double> findLevelAverages(TreeNode* root) {
  if (root == nullptr) return {};

  vector<double> result;
  queue<TreeNode*> q;
  q.push(root);

  while (!q.empty()) {
    int levelSize = static_cast<int>(q.size());
    long long levelSum = 0;

    for (int i = 0; i < levelSize; i++) {
      TreeNode* currNode = q.front();
      q.pop();
      levelSum += currNode->value;
      if (currNode->left) q.push(currNode->left);
      if (currNode->right) q.push(currNode->right);
    }
    result.push_back(static_cast<double>(levelSum) / levelSize);
  }
  return result;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->left->right = new TreeNode(2);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  cout << "Level averages are: ";
  for (double v : findLevelAverages(root)) cout << v << " ";
  cout << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` which is required for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue

### Level Maximum in a Binary Tree

https://leetcode.com/problems/maximum-level-sum-of-a-binary-tree/

> 🌟 Find the largest value on each level of a binary tree.

We will follow a similar approach, but instead of having a running sum we will track the maximum value of each level.

`maxValue = Math.max(maxValue, currentNode.val)`

```cpp
#include <algorithm>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

vector<int> largestValue(TreeNode* root) {
  if (root == nullptr) return {};

  vector<int> result;
  queue<TreeNode*> queueNodes;
  queueNodes.push(root);

  while (!queueNodes.empty()) {
    int levelSize = static_cast<int>(queueNodes.size());
    int maxValue = numeric_limits<int>::min();

    for (int i = 0; i < levelSize; i++) {
      TreeNode* currNode = queueNodes.front();
      queueNodes.pop();

      maxValue = max(maxValue, currNode->value);

      if (currNode->left) queueNodes.push(currNode->left);
      if (currNode->right) queueNodes.push(currNode->right);
    }
    result.push_back(maxValue);
  }
  return result;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->left->right = new TreeNode(2);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);

  cout << "Max values for each level: ";
  for (int v : largestValue(root)) cout << v << " ";
  cout << "\n";
}
```

## Minimum Depth of a Binary Tree (easy)

https://leetcode.com/problems/minimum-depth-of-binary-tree/

> Find the minimum depth of a binary tree. The minimum depth is the number of nodes along the <b>shortest path from the root node to the nearest leaf node</b>.

This problem follows the <b>Binary Tree Level Order Traversal</b> pattern. We can follow the same <b>BFS</b> approach. The only difference will be, instead of keeping track of all the nodes in a level, we will only track the depth of the tree. As soon as we find our first leaf node, that level will represent the minimum depth of the tree.

```cpp
#include <iostream>
#include <queue>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

int findMinimumDepth(TreeNode* root) {
  if (root == nullptr) return 0;

  queue<TreeNode*> q;
  q.push(root);
  int minimumTreeDepth = 0;

  while (!q.empty()) {
    minimumTreeDepth++;
    int levelSize = static_cast<int>(q.size());

    for (int i = 0; i < levelSize; i++) {
      TreeNode* currNode = q.front();
      q.pop();

      if (currNode->left == nullptr && currNode->right == nullptr) {
        return minimumTreeDepth;
      }

      if (currNode->left) q.push(currNode->left);
      if (currNode->right) q.push(currNode->right);
    }
  }
  return minimumTreeDepth;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  cout << "Tree Minimum Depth: " << findMinimumDepth(root) << "\n";
  root->left->left = new TreeNode(9);
  root->right->left->left = new TreeNode(11);
  cout << "Tree Minimum Depth: " << findMinimumDepth(root) << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` which is required for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

### Maximum Depth of a Binary Tree

https://leetcode.com/problems/maximum-depth-of-binary-tree/

> Given a binary tree, find its maximum depth (or height).

We will follow a similar approach. Instead of returning as soon as we find a leaf node, we will keep traversing for all the levels, incrementing `maximumDepth` each time we complete a level.

```cpp
#include <iostream>
#include <queue>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

int findMaximumDepth(TreeNode* root) {
  if (root == nullptr) return 0;

  queue<TreeNode*> q;
  q.push(root);
  int maximumTreeDepth = 0;

  while (!q.empty()) {
    maximumTreeDepth++;
    int levelSize = static_cast<int>(q.size());

    for (int i = 0; i < levelSize; i++) {
      TreeNode* currNode = q.front();
      q.pop();
      if (currNode->left) q.push(currNode->left);
      if (currNode->right) q.push(currNode->right);
    }
  }
  return maximumTreeDepth;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  cout << "Tree Maximum Depth: " << findMaximumDepth(root) << "\n";
  root->left->left = new TreeNode(9);
  root->right->left->left = new TreeNode(11);
  cout << "Tree Maximum Depth: " << findMaximumDepth(root) << "\n";
}
```

## Level Order Successor (easy)

> Given a binary tree and a node, find the level order successor of the given node in the tree. The level order successor is the node that appears right after the given node in the level order traversal.

This problem follows the <b>Binary Tree Level Order Traversal</b> pattern. We can follow the same <b>BFS</b> approach. The only difference will be that we will not keep track of all the levels. Instead we will keep inserting child nodes to the queue. As soon as we find the given node, we will return the next node from the <b>queue</b> as the level order successor.

```cpp
#include <iostream>
#include <queue>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v, TreeNode* l = nullptr, TreeNode* r = nullptr)
      : value(v), left(l), right(r) {}
};

TreeNode* findSuccessor(TreeNode* root, int key) {
  if (root == nullptr) return nullptr;

  queue<TreeNode*> q;
  q.push(root);

  while (!q.empty()) {
    TreeNode* currNode = q.front();
    q.pop();

    if (currNode->left) q.push(currNode->left);
    if (currNode->right) q.push(currNode->right);

    if (currNode->value == key) break;
  }
  if (!q.empty()) return q.front();

  return nullptr;
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);

  TreeNode* result = findSuccessor(root, 12);
  if (result != nullptr) cout << result->value << "\n";

  result = findSuccessor(root, 9);
  if (result != nullptr) cout << result->value << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` which is required for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

## 😕 Connect Level Order Siblings (medium)

https://leetcode.com/problems/populating-next-right-pointers-in-each-node/

> Given a binary tree, connect each node with its level order successor. The last node of each level should point to a `null` node.

This problem follows the <b>Binary Tree Level Order Traversal</b> pattern. We can follow the same <b>BFS</b> approach. The only difference is that while traversing a level we will remember the previous node to connect it with the current node.

```cpp
#include <iostream>
#include <queue>
using namespace std;

struct TreeNode {
  int val;
  TreeNode* left;
  TreeNode* right;
  TreeNode* next;
  explicit TreeNode(int v)
      : val(v), left(nullptr), right(nullptr), next(nullptr) {}
};

void connectLevelOrderSiblings(TreeNode* root) {
  if (root == nullptr) return;

  queue<TreeNode*> q;
  q.push(root);

  while (!q.empty()) {
    TreeNode* previousNode = nullptr;
    int levelSize = static_cast<int>(q.size());

    for (int i = 0; i < levelSize; i++) {
      TreeNode* currentNode = q.front();
      q.pop();
      if (previousNode != nullptr) {
        previousNode->next = currentNode;
      }
      previousNode = currentNode;

      if (currentNode->left) q.push(currentNode->left);
      if (currentNode->right) q.push(currentNode->right);
    }
  }
}

void printLevelOrder(TreeNode* root) {
  cout << "Level order traversal using 'next' pointer:\n";
  TreeNode* nextLevelRoot = root;
  while (nextLevelRoot != nullptr) {
    TreeNode* currentNode = nextLevelRoot;
    nextLevelRoot = nullptr;
    while (currentNode != nullptr) {
      cout << currentNode->val << ' ';
      if (nextLevelRoot == nullptr) {
        if (currentNode->left != nullptr) {
          nextLevelRoot = currentNode->left;
        } else if (currentNode->right != nullptr) {
          nextLevelRoot = currentNode->right;
        }
      }
      currentNode = currentNode->next;
    }
    cout << "\n";
  }
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  connectLevelOrderSiblings(root);
  printLevelOrder(root);
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)`, which is required for the queue. Since we can have a maximum of `N/2`nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

## 🌟 Connect All Level Order Siblings (medium)

> Given a binary tree, connect each node with its level order successor. The last node of each level should point to the first node of the next level.

This problem follows the <b>Binary Tree Level Order Traversal</b> pattern. We can follow the same <b>BFS</b> approach. The only difference will be that while traversing we will remember (irrespective of the level) the previous node to connect it with the current node.

```cpp
#include <iostream>
#include <queue>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  TreeNode* next;
  explicit TreeNode(int v)
      : value(v), left(nullptr), right(nullptr), next(nullptr) {}

  void printTree() {
    cout << "Traversal using 'next' pointer: ";
    TreeNode* current = this;
    while (current != nullptr) {
      cout << current->value << ' ';
      current = current->next;
    }
    cout << "\n";
  }
};

void connectAllSiblings(TreeNode* root) {
  if (root == nullptr) return;

  queue<TreeNode*> q;
  q.push(root);
  TreeNode* previousNode = nullptr;

  while (!q.empty()) {
    TreeNode* currentNode = q.front();
    q.pop();

    if (previousNode != nullptr) {
      previousNode->next = currentNode;
    }
    previousNode = currentNode;

    if (currentNode->left) q.push(currentNode->left);
    if (currentNode->right) q.push(currentNode->right);
  }
}

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  connectAllSiblings(root);
  root->printTree();
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once.
- The space complexity of the above algorithm will be `O(N)` which is required for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

## 🌟 Right View of a Binary Tree (easy)

https://leetcode.com/problems/binary-tree-right-side-view/

> Given a binary tree, return an array containing nodes in its right view. The right view of a binary tree is the set of <b>nodes visible when the tree is seen from the right side</b>.

```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v) : value(v), left(nullptr), right(nullptr) {}
};

vector<int> treeRightView(TreeNode* root) {
  vector<int> result;
  if (root == nullptr) {
    return result;
  }

  queue<TreeNode*> q;
  q.push(root);

  while (!q.empty()) {
    int levelSize = static_cast<int>(q.size());

    for (int i = 0; i < levelSize; i++) {
      TreeNode* currentNode = q.front();
      q.pop();

      if (i == levelSize - 1) {
        result.push_back(currentNode->value);
      }
      if (currentNode->left) q.push(currentNode->left);
      if (currentNode->right) q.push(currentNode->right);
    }
  }

  return result;
};

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  root->left->left->left = new TreeNode(3);
  vector<int> view = treeRightView(root);
  cout << "Tree right view: ";
  for (int v : view) cout << v << ' ';
  cout << "\n";
}
```

- The time complexity of the above algorithm is `O(N)`, where `N` is the total number of nodes in the tree. This is due to the fact that we traverse each node once
- The space complexity of the above algorithm will be `O(N)` as we need to return a list containing the level order traversal. We will also need `O(N)` space for the queue. Since we can have a maximum of `N/2` nodes at any level (this could happen only at the lowest level), therefore we will need `O(N)` space to store them in the queue.

### Similar Questions

> Given a binary tree, return an array containing nodes in its left view. The left view of a binary tree is the set of nodes visible when the tree is seen from the left side.

We will be following a similar approach, but instead of appending the last element of each level, we will be appending the first element of each level to the output array.

```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

struct TreeNode {
  int value;
  TreeNode* left;
  TreeNode* right;
  explicit TreeNode(int v) : value(v), left(nullptr), right(nullptr) {}
};

vector<int> treeLeftView(TreeNode* root) {
  vector<int> result;
  if (root == nullptr) {
    return result;
  }

  queue<TreeNode*> q;
  q.push(root);

  while (!q.empty()) {
    int levelSize = static_cast<int>(q.size());

    for (int i = 0; i < levelSize; i++) {
      TreeNode* currentNode = q.front();
      q.pop();

      if (i == 0) {
        result.push_back(currentNode->value);
      }
      if (currentNode->left) q.push(currentNode->left);
      if (currentNode->right) q.push(currentNode->right);
    }
  }

  return result;
};

int main() {
  TreeNode* root = new TreeNode(12);
  root->left = new TreeNode(7);
  root->right = new TreeNode(1);
  root->left->left = new TreeNode(9);
  root->right->left = new TreeNode(10);
  root->right->right = new TreeNode(5);
  root->left->left->left = new TreeNode(3);
  vector<int> view = treeLeftView(root);
  cout << "Tree left view: ";
  for (int v : view) cout << v << ' ';
  cout << "\n";
}
```

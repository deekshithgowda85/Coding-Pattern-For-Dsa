### Similar problems

Here are a couple of similar problems:

## 1. Minimum insertions in a string to make it a palindrome

https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/

Will the above approach work if we make insertions instead of deletions?

Yes, the length of the <b>Longest Palindromic Subsequence</b> is the best <i>palindromic subsequence</i> we can have. Let’s take a few examples:

### Example 1:

```text
Input: "abdbca"
Output: 1
Explanation: By inserting “c”, we get a palindrome “acbdbca”.
```

### Example 2:

```text
Input: = "cddpd"
Output: 2
Explanation: Inserting “cp”, we get a palindrome “cdpdpdc”. We can also get a palindrome by inserting “dc”: “cddpddc”
```

### Example 3:

```text
Input: = "pqr"
Output: 2
Explanation: We have to insert any two characters to get a palindrome (e.g. if we insert “pq”, we get a palindrome “pqrqp”).
```

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int lpsLength(const string& s) {
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; ++i) dp[i][i] = 1;
    for (int start = n - 1; start >= 0; --start) {
        for (int end = start + 1; end < n; ++end) {
            if (s[start] == s[end]) dp[start][end] = 2 + dp[start + 1][end - 1];
            else dp[start][end] = max(dp[start + 1][end], dp[start][end - 1]);
        }
    }
    return dp[0][n - 1];
}

int minInsertions(const string& s) {
    return (int)s.length() - lpsLength(s);
}

int main() {
    cout << "Minimum number of insertions required ---> " << minInsertions("abdbca") << endl;
    cout << "Minimum number of insertions required ---> " << minInsertions("cddpd") << endl;
    cout << "Minimum number of insertions required ---> " << minInsertions("pqr") << endl;
    cout << "Minimum number of insertions required ---> " << minInsertions("zzazz") << endl;
    cout << "Minimum number of insertions required ---> " << minInsertions("mbadm") << endl;
    cout << "Minimum number of insertions required ---> " << minInsertions("leetcode") << endl;
    return 0;
}
```

## 2. Find if a string is K-Palindromic

https://leetcode.com/problems/valid-palindrome-iii/

Any string will be called `K`<b>-palindromic</b> if it can be transformed into a <i>palindrome</i> by removing at most `K` characters from it.

This problem can easily be converted to our base problem of <i>finding the minimum deletions in a string to make it a palindrome</i>. If the <i>“minimum deletion count”</i> is not more than `K`, the string will be `K`<b>-palindromic</b>.

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int lpsLen(const string& s) {
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; ++i) dp[i][i] = 1;
    for (int start = n - 1; start >= 0; --start) {
        for (int end = start + 1; end < n; ++end) {
            if (s[start] == s[end]) dp[start][end] = 2 + dp[start + 1][end - 1];
            else dp[start][end] = max(dp[start + 1][end], dp[start][end - 1]);
        }
    }
    return dp[0][n - 1];
}

bool isValidPalindrome(const string& s, int K) {
    return (int)s.length() - lpsLen(s) <= K;
}

int main() {
    cout << "Is abcdeca a k-palindrome ---> " << (isValidPalindrome("abcdeca", 2) ? "true" : "false") << endl;
    cout << "Is abbababa a k-palindrome ---> " << (isValidPalindrome("abbababa", 1) ? "true" : "false") << endl;
    return 0;
}
```

## Palindromic Partitioning

https://leetcode.com/problems/palindrome-partitioning-ii/

> Given a string, we want to cut it into pieces such that each piece is a <b>palindrome</b>. Write a function to return the minimum number of cuts needed.

#### Example 1:

```text
Input: "abdbca"
Output: 3
Explanation: Palindrome pieces are "a", "bdb", "c", "a".
```

#### Example 2:

```text
Input: = "cddpd"
Output: 2
Explanation: Palindrome pieces are "c", "d", "dpd".
```

#### Example 3:

```text
Input: = "pqr"
Output: 2
Explanation: Palindrome pieces are "p", "q", "r".
```

#### Example 4:

```text
Input: = "pp"
Output: 0
Explanation: We do not need to cut, as "pp" is a palindrome.
```

### Brute-Force Recursive Solution

This problem follows the <b>[Longest Palindromic Subsequence pattern](#pattern-4-palindromic-subsequence)</b> and shares a similar approach as that of the [Longest Palindromic Substring](#longest-palindromic-subsequence).

The <b>brute-force solution</b> will be to try all the <i>substring combinations</i> of the given string. We can start processing from the beginning of the string and keep adding one character at a time. At any step, if we get a <i>palindrome</i>, we take it as one piece and <i>recursively</i> process the remaining length of the string to find the minimum cuts needed.

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

bool isPalindrome(const string& s, int start, int end) {
    while (start <= end) {
        if (s[start++] != s[end--]) return false;
    }
    return true;
}

int findMPPCutsRecursive(const string& s, int startIndex, int endIndex) {
    if (startIndex >= endIndex || isPalindrome(s, startIndex, endIndex)) return 0;
    int minimumCuts = endIndex - startIndex;
    for (int i = startIndex; i <= endIndex; ++i) {
        if (isPalindrome(s, startIndex, i)) {
            minimumCuts = min(minimumCuts, 1 + findMPPCutsRecursive(s, i + 1, endIndex));
        }
    }
    return minimumCuts;
}

int findMPPCuts(const string& s) {
    return findMPPCutsRecursive(s, 0, (int)s.size() - 1);
}

int main() {
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("abdbca") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("cdpdd") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("pqr") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("pp") << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ⁿ)`, where `n` represents the total number.
- The <b>space complexity</b> is `O(n)`, which will be used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

We can <i>memoize</i> both functions `findMPPCutsRecursive()` and `isPalindrome()`. The two changing values in both these functions are the two indices; therefore, we can store the results of all the <i>subproblems</i> in a two-dimensional array. (alternatively, we can use a <i>hash-table</i>).

Here is the code:

````cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

bool isPalindromeMemo(const string& s, int start, int end, vector<vector<int>>& pal) {
    if (start >= end) return true;
    int& cached = pal[start][end];
    if (cached != -1) return cached == 1;
    if (s[start] == s[end]) {
        cached = isPalindromeMemo(s, start + 1, end - 1, pal) ? 1 : 0;
    } else {
        cached = 0;
    }
    return cached == 1;
}

int minCutsMemo(const string& s, int start, int end, vector<vector<int>>& dp, vector<vector<int>>& pal) {
    if (start >= end || isPalindromeMemo(s, start, end, pal)) return 0;
    int& res = dp[start][end];
    if (res != -1) return res;
    int minimumCuts = end - start;
    for (int i = start; i <= end; ++i) {
        if (isPalindromeMemo(s, start, i, pal)) {
            minimumCuts = min(minimumCuts, 1 + minCutsMemo(s, i + 1, end, dp, pal));
        }
    }
    return res = minimumCuts;
}

int findMPPCuts(const string& s) {
    int n = (int)s.size();
    vector<vector<int>> dp(n, vector<int>(n, -1));
    vector<vector<int>> pal(n, vector<int>(n, -1));
    return minCutsMemo(s, 0, n - 1, dp, pal);
}

int main() {
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("abdbca") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("cdpdd") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("pqr") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("pp") << endl;
    return 0;
}
````

### Bottom-up Dynamic Programming

The above solution tells us that we need to build two tables, one for the `isPalindrome()` and one for `findMPPCuts()`.

If you remember, we built a table in the <b>[Longest Palindromic Substring (LPS)](#longest-palindromic-subsequence)</b> chapter that can tell us what <i>substrings</i> (of the input <i>string</i>) are <i>palindrome</i>. We will use the same approach here to build the table required for `isPalindrome()`.

To build the second table for finding the minimum cuts, we can iterate through the first table built for `isPalindrome()`. At any step, if we get a <i>palindrome</i>, we can cut the <i>string</i> there. Which means minimum cuts will be one plus the cuts needed for the <i>remaining string</i>.

Here is the code for the <b>bottom-up approach</b>:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int findMPPCuts(const string& s) {
    int n = (int)s.size();
    vector<vector<bool>> isPalindrome(n, vector<bool>(n, false));
    for (int i = 0; i < n; ++i) isPalindrome[i][i] = true;

    for (int start = n - 1; start >= 0; --start) {
        for (int end = start + 1; end < n; ++end) {
            if (s[start] == s[end]) {
                if (end - start == 1 || isPalindrome[start + 1][end - 1]) {
                    isPalindrome[start][end] = true;
                }
            }
        }
    }

    vector<int> cuts(n, 0);
    for (int start = n - 1; start >= 0; --start) {
        int minCuts = n;
        for (int end = n - 1; end >= start; --end) {
            if (isPalindrome[start][end]) {
                minCuts = (end == n - 1) ? 0 : min(minCuts, 1 + cuts[end + 1]);
            }
        }
        cuts[start] = minCuts;
    }

    return cuts[0];
}

int main() {
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("abdbca") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("cdpdd") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("pqr") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("pp") << endl;
    cout << "Minimum palindrome partitions ---> " << findMPPCuts("madam") << endl;
    return 0;
}
```

- The <b>time and space complexity</b> of the above algorithm is `O(n²)`, where `n` is the length of the input string.

# Pattern 5: Longest Common Substring

## Problem Set

1. [Longest Common Substring](#longest-common-substring)
2. [🔎 Longest Common Subsequence](#🔎-longest-common-subsequence)
3. [Minimum Deletions & Insertions to Transform a String into another](#minimum-deletions--insertions-to-transform-a-string-into-another)
4. [👩🏽‍🦯 🔎 Longest Increasing Subsequence](#👩🏽‍🦯-🔎-longest-increasing-subsequence)
5. [Maximum Sum Increasing Subsequence](#maximum-sum-increasing-subsequence)
6. [Shortest Common Super-sequence](#shortest-common-super-sequence)
7. [Minimum Deletions to Make a Sequence Sorted](#minimum-deletions-to-make-a-sequence-sorted)
8. [Longest Repeating Subsequence](#longest-repeating-subsequence)
9. [Subsequence Pattern Matching](#subsequence-pattern-matching)
10. [Longest Bitonic Subsequence](#longest-bitonic-subsequence)
11. [Longest Alternating Subsequence](#longest-alternating-subsequence)
12. [🔎 Edit Distance](#🔎-edit-distance)
13. [🔎 Strings Interleaving](#🔎-strings-interleaving)

## Longest Common Substring

https://www.geeksforgeeks.org/longest-common-substring-dp-29/

> Given two strings `str1` and `str2`, find the length of the longest <b>substring</b> which is common in both the strings.

#### Example 1:

```text
Input: str1 = "abdca"
       str2 = "cbda"
Output: 2
Explanation: The longest common substring is "bd".
```

#### Example 2:

```text
Input: str1 = "passport"
       str2 = "ppsspt"
Output: 3
Explanation: The longest common substring is "ssp".
```

### Brute-Force Solution

A basic <b>brute-force solution</b> could be to try all substrings of `str1` and `str2` to find the longest common one. We can start matching both the strings one character at a time, so we have two options at any step:

1. If the strings have a matching character, we can <i>recursively</i> match for the remaining lengths and keep a track of the current matching length.
2. If the strings don’t match, we start two new <i>recursive calls</i> by skipping one character separately from each string and reset the matching length.

The length of the <b>Longest Common Substring (LCS)</b> will be the maximum number returned by the three <i>recurse calls</i> in the above two options.

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

int findLCSLengthRecursive(const string& s1, const string& s2, int i1, int i2, int count) {
    if (i1 == (int)s1.size() || i2 == (int)s2.size()) return count;
    int newCount = count;
    if (s1[i1] == s2[i2]) {
        newCount = findLCSLengthRecursive(s1, s2, i1 + 1, i2 + 1, count + 1);
    }
    int check1 = findLCSLengthRecursive(s1, s2, i1, i2 + 1, 0);
    int check2 = findLCSLengthRecursive(s1, s2, i1 + 1, i2, 0);
    return max(newCount, max(check1, check2));
}

int findLCSLength(const string& s1, const string& s2) {
    return findLCSLengthRecursive(s1, s2, 0, 0, 0);
}

int main() {
    cout << "Length of Longest Common Substring: ---> " << findLCSLength("abdca", "cbda") << endl;
    cout << "Length of Longest Common Substring: ---> " << findLCSLength("passport", "ppsspt") << endl;
    return 0;
}
```

- Because of the three <i>recursive calls</i>, the <b>time complexity</b> of the above algorithm is exponential `O(3ᵐ⁺ⁿ)`, where `m` and `n` are the lengths of the two input strings. The <b>space complexity</b> is `O(m+n)`, this <b>space</b> will be used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

We can use an array to store the already solved <i>subproblems</i>.

The three changing values to our <i>recursive function</i> are the two indices (`index1` and `index2`) and the `count`. Therefore, we can store the results of all <i>subproblems</i>in a <i>three-dimensional array</i>. (Another alternative could be to use a <i>hash-table</i> whose key would be a string (`index1` + `“|”` `index2` + `“|”` + `count`)).

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

vector<vector<vector<int>>> dp;

int lcsRec(const string& s1, const string& s2, int i1, int i2, int count) {
    if (i1 == (int)s1.size() || i2 == (int)s2.size()) return count;
    if (dp[i1][i2][count] != -1) return dp[i1][i2][count];
    int c1 = count;
    if (s1[i1] == s2[i2]) {
        c1 = lcsRec(s1, s2, i1 + 1, i2 + 1, count + 1);
    }
    int c2 = lcsRec(s1, s2, i1, i2 + 1, 0);
    int c3 = lcsRec(s1, s2, i1 + 1, i2, 0);
    return dp[i1][i2][count] = max(c1, max(c2, c3));
}

int findLCSLength(const string& s1, const string& s2) {
    int maxLen = min(s1.size(), s2.size());
    dp.assign(s1.size(), vector<vector<int>>(s2.size(), vector<int>(maxLen + 1, -1)));
    return lcsRec(s1, s2, 0, 0, 0);
}

int main() {
    cout << "Length of Longest Common Substring: ---> " << findLCSLength("abdca", "cbda") << endl;
    dp.clear();
    cout << "Length of Longest Common Substring: ---> " << findLCSLength("passport", "ppsspt") << endl;
    return 0;
}
```

### Bottom-up Dynamic Programming

Since we want to match all the <i>substrings</i> of the given two <i>strings</i>, we can use a two-dimensional array to store our results. The lengths of the two <i>strings</i> will define the size of the two dimensions of the array. So for every index `index1` in string `str1` and `index2` in string `str2`, we have two options:

1. If the character at `str1[index1]` matches `str2[index2]`, the length of the <i>common substring</i> would be one plus the length of the <i>common substring</i> until `index1-1` and `index2-1` indices in the two <i>strings</i>.
2. If the character at the `str1[index1]` does not match `str2[index2]`, we don’t have any <i>common substring</i>.
   So our <i>recursive formula</i> would be:

```cpp
if (str1[index1] == str2[index2])
  dp[index1][index2] = 1 + dp[index1-1][index2-1];
else
  dp[index1][index2] = 0;
```

we can clearly see that the <b>longest common substring</b> is of length `2`-- as shown by `dp[3][3]`.

Here is the code for our <b>bottom-up dynamic programming approach</b>:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int findLCSLength(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.size() + 1, vector<int>(s2.size() + 1, 0));
    int maxLength = 0;
    for (size_t i = 1; i <= s1.size(); ++i) {
        for (size_t j = 1; j <= s2.size(); ++j) {
            if (s1[i - 1] == s2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
                maxLength = max(maxLength, dp[i][j]);
            }
        }
    }
    return maxLength;
}

int main() {
    cout << "Length of Longest Common Substring: ---> " << findLCSLength("abdca", "cbda") << endl;
    cout << "Length of Longest Common Substring: ---> " << findLCSLength("passport", "ppsspt") << endl;
    return 0;
}
```

- The <b>time and space complexity</b> of the above algorithm is `O(m∗n)`, where `m` and `n` are the lengths of the two input <i>strings</i>.

### Challenge

Can we further improve our <b>bottom-up DP</b> solution? Can you find an algorithm that has `O(n)` <b>space complexity</b>

## 🔎 Longest Common Subsequence

https://leetcode.com/problems/longest-common-subsequence/

> Given two strings `str1` and `str2`, find the length of the <i>longest subsequence</i> which is common in both the strings.

A <b>subsequence</b> is a <i>sequence that can be derived from another sequence by deleting some or no elements without changing the order of the remaining elements</i>.

#### Example :

```text
Input: str1 = "abdca"
       str2 = "cbda"
Output: 3
Explanation: The longest common subsequence is "bda".
```

#### Example 2:

```text
Input: str1 = "passport"
       str2 = "ppsspt"
Output: 5
Explanation: The longest common subsequence is "psspt".
```

### Basic Brute-Force Solution

A <b>basic brute-force solution</b> could be to try all <i>subsequences</i> of `str1` and `str2` to find the longest one. We can match both the strings one character at a time. So for every index `index1` in `str1` and `index2` in `str2` we must choose between:

1. If the character `str1[index1]` matches `str2[index2`, we can <i>recursively</i> match for the remaining lengths.
2. If the character `str1[index1]` does not match `str2[index2]`, we will start two new <i>recursive calls</i> by skipping one character separately from each string.

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

int lcsRec(const string& s1, const string& s2, int i1, int i2) {
    if (i1 == (int)s1.size() || i2 == (int)s2.size()) return 0;
    if (s1[i1] == s2[i2]) return 1 + lcsRec(s1, s2, i1 + 1, i2 + 1);
    int c1 = lcsRec(s1, s2, i1, i2 + 1);
    int c2 = lcsRec(s1, s2, i1 + 1, i2);
    return max(c1, c2);
}

int findLCSLength(const string& s1, const string& s2) {
    return lcsRec(s1, s2, 0, 0);
}

int main() {
    cout << "Length of Longest Common Subsequence: ---> " << findLCSLength("abdca", "cbda") << endl;
    cout << "Length of Longest Common Subsequence: ---> " << findLCSLength("passport", "ppsspt") << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ᵐ⁺ⁿ)`, where `m` and `n` are the lengths of the two input strings.
- The <b>space complexity</b> is `O(m+n)`, this <b>space</b> will be used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

We can use an array to store the already solved <i>subproblems</i>.

The two changing values to our <i>recursive function</i> are the two indices, `index1` and `index2`. Therefore, we can store the results of all the <i>subproblems</i> in a two-dimensional array. (Another alternative could be to use a <i>hash-table</i> whose key would be a <i>string</i> (`index1` + `“|”` + `index2`)).

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int lcsRec(const string& s1, const string& s2, int i1, int i2, vector<vector<int>>& dp) {
    if (i1 == (int)s1.size() || i2 == (int)s2.size()) return 0;
    if (dp[i1][i2] != -1) return dp[i1][i2];
    if (s1[i1] == s2[i2]) {
        return dp[i1][i2] = 1 + lcsRec(s1, s2, i1 + 1, i2 + 1, dp);
    }
    int c1 = lcsRec(s1, s2, i1, i2 + 1, dp);
    int c2 = lcsRec(s1, s2, i1 + 1, i2, dp);
    return dp[i1][i2] = max(c1, c2);
}

int findLCSLength(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.size(), vector<int>(s2.size(), -1));
    return lcsRec(s1, s2, 0, 0, dp);
}

int main() {
    cout << "Length of Longest Common Subsequence: ---> " << findLCSLength("abdca", "cbda") << endl;
    cout << "Length of Longest Common Subsequence: ---> " << findLCSLength("passport", "ppsspt") << endl;
    return 0;
}
```

### Bottom-up Dynamic Programming

Since we want to match all the <b>subsequences</b> of the given two strings, we can use a two-dimensional array to store our results. The lengths of the two strings will define the size of the array’s two dimensions. So for every index `index1` in string `str1` and `index2` in string `str2`, we will choose one of the following two options:

1. If the character `str1[index1]` matches `str2[index2]`, the length of the <b>common subsequence</b> would be one plus the length of the <b>common subsequence</b> until the `index1-1` and `index2-1` indices in the two respective strings.
2. If the character `str1[index1]`does not match `str2[index2]`, we will take the <i>longest subsequence</i> by either skipping `[index1]th` or `[index2]th` character from the respective strings.

So our <b>recursive formula</b> would be:

```cpp
if (str1[index1] == str2[index2])
  dp[index1][index2] = 1 + dp[index1-1][index2-1];
else
  dp[index1][index2] = max(dp[index1-1][index2], dp[index1][index2-1]);
```

From the above visualization, we can clearly see that the <b>longest common subsequence</b> is of length `3` – as shown by `dp[4][5]`.

Here is the code for our <b>bottom-up dynamic programming approach</b>:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int findLCSLength(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.size() + 1, vector<int>(s2.size() + 1, 0));
    int maxLength = 0;
    for (size_t i = 1; i <= s1.size(); ++i) {
        for (size_t j = 1; j <= s2.size(); ++j) {
            if (s1[i - 1] == s2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
            maxLength = max(maxLength, dp[i][j]);
        }
    }
    return maxLength;
}

int main() {
    cout << "Length of Longest Common Subsequence: ---> " << findLCSLength("abdca", "cbda") << endl;
    cout << "Length of Longest Common Subsequence: ---> " << findLCSLength("passport", "ppsspt") << endl;
    return 0;
}
```

- The <b>time and space complexity</b> of the above algorithm is `O(m*n)`, where `m` and `n` are the lengths of the two input strings.

### Challenge

Can we further improve our <b>bottom-up DP solution</b>? Can you find an algorithm that has `O(n)` <b>space complexity</b>?

## Minimum Deletions & Insertions to Transform a String into another

https://practice.geeksforgeeks.org/problems/minimum-number-of-deletions-and-insertions0209/1/

> Given strings `str1` and `str2`, we need to transform `str1` into `str2` by deleting and inserting characters. Write a function to calculate the count of the minimum number of deletion and insertion operations.

### Example 1:

```text
Input: str1 = "abc"
       str2 = "fbc"
Output: 1 deletion and 1 insertion.
Explanation: We need to delete {'a'} and insert {'f'} to str1 to transform it into str2.
```

### Example 2:

```text
Input: str1 = "abdca"
       str2 = "cbda"
Output: 2 deletions and 1 insertion.
Explanation: We need to delete {'a', 'c'} and insert {'c'} to str1 to transform it into str2.
```

### Example 3:

```text
Input: str1 = "passport"
       str2 = "ppsspt"
Output: 3 deletions and 1 insertion
Explanation: We need to delete {'a', 'o', 'r'} and insert {'p'} to str1 to transform it into str2.
```

This problem can easily be converted to the <b>[Longest Common Subsequence (LCS)](#🔎-longest-common-subsequence)</b>. If we can find the <b>LCS</b> of the two input strings, we can easily find how many characters we need to insert and delete from `str1`. Here is how we can do this:

1. Let’s assume `length1` is the length of `str1` and `length2` is the length of `str2`.
2. Now let’s assume `c1` is the length of <b>LCS</b> of the two strings `str1` and `str2`.
3. To transform `str1` into `str2`, we need to delete everything from `str1` which is not part of <b>LCS</b>, so minimum deletions we need to perform from `str1` => `length1 - c1`
4. Similarly, we need to insert everything in `str1` which is present in `str2` but not part of <b>LCS</b>, so minimum insertions we need to perform in `str1` => `length2 - c1`

### Bottom-up Dynamic Programming Solution

Let’s jump directly to the <b>bottom-up dynamic programming solution</b>:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int findLCSLength(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.size() + 1, vector<int>(s2.size() + 1, 0));
    int maxLength = 0;
    for (size_t i = 1; i <= s1.size(); ++i) {
        for (size_t j = 1; j <= s2.size(); ++j) {
            if (s1[i - 1] == s2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
            maxLength = max(maxLength, dp[i][j]);
        }
    }
    return maxLength;
}

void findMDI(const string& s1, const string& s2) {
    int c1 = findLCSLength(s1, s2);
    cout << "We need " << (s1.size() - c1) << " deletions and " 
         << (s2.size() - c1) << " insertions to transform \"" 
         << s1 << "\" into \"" << s2 << "\"" << endl;
}

int main() {
    findMDI("abc", "fbc");
    findMDI("abdca", "cbda");
    findMDI("passport", "ppsspt");
    return 0;
}
```

- The <b>time and space complexity</b> of the above algorithm is `O(m*n)`, where `m` and `n` are the lengths of the two input strings.

## 👩🏽‍🦯 🔎 Longest Increasing Subsequence

https://leetcode.com/problems/longest-increasing-subsequence/

> Given a number `sequence`, find the length of its <b>Longest Increasing Subsequence (LIS)</b>. In an <b>increasing subsequence</b>, <i>all the elements are in increasing order (from lowest to highest)</i>.

### Example 1:

```text
Input: {4,2,3,6,10,1,12}
Output: 5
Explanation: The LIS is {2,3,6,10,12}.
```

### Example 2:

```text
Input: {-4,10,3,7,15}
Output: 4
Explanation: The LIS is {-4,3,7,15}.
```

### Basic Brute-Force Solution

A <b>basic brute-force solution</b> could be to try all the <i>subsequences</i> of the given number sequence. We can process one number at a time, so we have two options at any step:

1. If the current number is greater than the previous number that we included, we can <i>increment our count</i> and make a <i>recursive call</i> for the remaining array.
2. We can skip the current number to make a <i>recursive call</i> for the remaining array.

The length of the <b>longest increasing subsequence</b> will be the maximum number returned by the two recurse calls from the above two options.

Here is the code:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int lisRec(const vector<int>& nums, int curr, int prev) {
    if (curr == (int)nums.size()) return 0;
    int c1 = 0;
    if (prev == -1 || nums[curr] > nums[prev]) {
        c1 = 1 + lisRec(nums, curr + 1, curr);
    }
    int c2 = lisRec(nums, curr + 1, prev);
    return max(c1, c2);
}

int findLISLength(const vector<int>& nums) {
    return lisRec(nums, 0, -1);
}

int main() {
    cout << "Length of Longest Increasing Subsequence: ---> " << findLISLength({4, 2, 3, 6, 10, 1, 12}) << endl;
    cout << "Length of Longest Increasing Subsequence: ---> " << findLISLength({-4, 10, 3, 7, 15}) << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ⁿ)`, where `n` is the lengths of the input array.
- The <b>space complexity</b> is `O(n)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

To overcome the <i>overlapping subproblems</i>, we can use an array to store the already solved <i>subproblems</i>.

The two changing values for our <i>recursive function</i> are the `currIndex` and the `prevIndex`. Therefore, we can store the results of all <i>subproblems</i> in a two-dimensional array. (Another alternative could be to use a <i>hash-table</i> whose key would be a string (`currIndex` + `“|”` + `prevIndex`)).

Here is the code:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int lisRec(const vector<int>& nums, int curr, int prev, vector<vector<int>>& dp) {
    if (curr == (int)nums.size()) return 0;
    if (dp[curr][prev + 1] != -1) return dp[curr][prev + 1];
    int c1 = 0;
    if (prev == -1 || nums[curr] > nums[prev]) {
        c1 = 1 + lisRec(nums, curr + 1, curr, dp);
    }
    int c2 = lisRec(nums, curr + 1, prev, dp);
    return dp[curr][prev + 1] = max(c1, c2);
}

int findLISLength(const vector<int>& nums) {
    vector<vector<int>> dp(nums.size(), vector<int>(nums.size() + 1, -1));
    return lisRec(nums, 0, -1, dp);
}

int main() {
    cout << "Length of Longest Increasing Subsequence: ---> " << findLISLength({4, 2, 3, 6, 10, 1, 12}) << endl;
    cout << "Length of Longest Increasing Subsequence: ---> " << findLISLength({-4, 10, 3, 7, 15}) << endl;
    return 0;
}
```

- Since our <i>memoization</i> array `dp[nums.length()][nums.length()]` stores the results for all the <i>subproblems</i>, we can conclude that we will not have more than `N*N` <i>subproblems</i> (where `N` is the length of the input sequence). This means that our <b>time complexity</b> will be `O(N²)`.
- The above algorithm will be using `O(N²)` <b>space</b> for the <i>memoization array</i>. Other than that we will use `O(N)` <b>space</b> for the <i>recursion call-stack</i>. So the total <b>space complexity</b> will be `O(N² + N)`, which is <i>asymptotically</i> equivalent to `O(N²)`.

### Bottom-up Dynamic Programming

The above algorithm tells us two things:

1. If the number at the `currIndex` is bigger than the number at the `prevIndex`, we increment the count for <b>LIS</b> up to the `currIndex`.
2. But if there is a bigger <b>LIS</b> without including the number at the `currIndex`, we take that.
   So we need to find all the <i>increasing subsequences</i> for the number at index `i`, from all the previous numbers (i.e. number until index `i-1`), to eventually find the <i>longest increasing subsequence.</i>

If `i` represents the `currIndex` and `j` represents the `prevIndex`, our <i>recursive formula</i> would look like:

```cpp
if (num[i] > num[j])
  dp[i] = dp[j] + 1; // if there is no bigger LIS for 'i'
```

Here is the code for our <b>bottom-up dynamic programming approach</b>:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findLISLength(const vector<int>& nums) {
    vector<int> dp(nums.size(), 1);
    int maxLength = 1;
    for (size_t i = 0; i < nums.size(); ++i) {
        for (size_t j = 0; j < i; ++j) {
            if (nums[i] > nums[j] && dp[i] <= dp[j]) {
                dp[i] = dp[j] + 1;
                maxLength = max(maxLength, dp[i]);
            }
        }
    }
    return maxLength;
}

int main() {
    cout << "Length of Longest Increasing Subsequence: ---> " << findLISLength({4, 2, 3, 6, 10, 1, 12}) << endl;
    cout << "Length of Longest Increasing Subsequence: ---> " << findLISLength({-4, 10, 3, 7, 15}) << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is `O(N²)` and the <b>space complexity</b> is `O(n)`.

## Maximum Sum Increasing Subsequence

https://www.geeksforgeeks.org/maximum-sum-increasing-subsequence-dp-14/

> Given a number sequence, find the increasing subsequence with the highest `sum`. Write a method that returns the highest `sum`.

#### Example 1:

```text
Input: {4,1,2,6,10,1,12}
Output: 32
Explanation: The increaseing sequence is {4,6,10,12}.
Please note the difference, as the LIS is {1,2,6,10,12} which has a sum of '31'.
```

#### Example 2:

```text
Input: {-4,10,3,7,15}
Output: 25
Explanation: The increaseing sequences are {10, 15} and {3,7,15}.
```

### Basic Brute Force Solution

The problem is quite similar to the <b>[Longest Increasing Subsequence](#👩🏽‍🦯-🔎-longest-increasing-subsequence)</b>. The only difference is that, instead of finding the increasing subsequence with the maximum length, we need to find an increasing sequence with the maximum `sum`.

A <b>basic brute-force solution</b> could be to try all the <i>subsequences</i> of the given array. We can process one number at a time, so we have two options at any step:

1. If the current number is greater than the previous number that we included, we include that number in a running `sum` and make a <i>recursive call</i> for the remaining array.
2. We can skip the current number to make a <i>recursive call</i> for the remaining array.

The highest `sum` of any <i>increasing subsequence</i> would be the max value returned by the two <i>recurse calls</i> from the above two options.

Here is the code:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int msisRec(const vector<int>& nums, int curr, int prev, int sum) {
    if (curr == (int)nums.size()) return sum;
    int s1 = sum;
    if (prev == -1 || nums[curr] > nums[prev]) {
        s1 = msisRec(nums, curr + 1, curr, sum + nums[curr]);
    }
    int s2 = msisRec(nums, curr + 1, prev, sum);
    return max(s1, s2);
}

int findMSIS(const vector<int>& nums) {
    return msisRec(nums, 0, -1, 0);
}

int main() {
    cout << "Maximum Sum Increasing Subsequence is: ---> " << findMSIS({4, 1, 2, 6, 10, 1, 12}) << endl;
    cout << "Maximum Sum Increasing Subsequence is: ---> " << findMSIS({-4, 10, 3, 7, 15}) << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ⁿ)`, where `n` is the lengths of the input array.
- The <b>space complexity</b> is `O(n)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

We can use <b>memoization</b> to overcome the <i>overlapping subproblems</i>.

The three changing values for our <i>recursive function</i> are the `currIndex`, the `prevIndex`, and the `sum`. An efficient way of storing the results of the <i>subproblems</i> could be a <i>hash-table</i> whose <i>key</i> would be a string (`currIndex` + `“|”` + `prevIndex` + `“|”` + `sum`).

Here is the code:

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>
using namespace std;

unordered_map<string, int> dp;

int msisRec(const vector<int>& nums, int curr, int prev, int sum) {
    if (curr == (int)nums.size()) return sum;
    string key = to_string(curr) + "-" + to_string(prev) + "-" + to_string(sum);
    if (dp.count(key)) return dp[key];
    int s1 = sum;
    if (prev == -1 || nums[curr] > nums[prev]) {
        s1 = msisRec(nums, curr + 1, curr, sum + nums[curr]);
    }
    int s2 = msisRec(nums, curr + 1, prev, sum);
    return dp[key] = max(s1, s2);
}

int findMSIS(const vector<int>& nums) {
    dp.clear();
    return msisRec(nums, 0, -1, 0);
}

int main() {
    cout << "Maximum Sum Increasing Subsequence is: ---> " << findMSIS({4, 1, 2, 6, 10, 1, 12}) << endl;
    cout << "Maximum Sum Increasing Subsequence is: ---> " << findMSIS({-4, 10, 3, 7, 15}) << endl;
    return 0;
}
```

### Bottom-up Dynamic Programming

The above algorithm tells us two things:

1. If the number at the `currIndex` is bigger than the number at the `prevIndex`, we include that number in the `sum` for an increasing sequence up to the `currIndex`.
2. But if there is a <b>maximum sum increasing subsequence (MSIS)</b>, without including the number at the `currIndex`, we take that.

So we need to find all the <i>increasing subsequences</i> for a number at index `i`, from all the previous numbers (i.e. numbers until index `i-1`), to find <b>MSIS</b>.

If `i` represents the `currIndex` and `j` represents the `prevIndex`, our <i>recursive formula</i> would look like:

```cpp
if (num[i] > num[j])
  dp[i] = dp[j] + num[i]; // if there is no bigger MSIS for 'i'
```

Here is the code for our <b>bottom-up dynamic programming approach</b>:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findMSIS(const vector<int>& nums) {
    vector<int> dp(nums.size());
    dp[0] = nums[0];
    int maxSum = nums[0];
    for (size_t i = 1; i < nums.size(); ++i) {
        dp[i] = nums[i];
        for (size_t j = 0; j < i; ++j) {
            if (nums[i] > nums[j] && dp[i] < dp[j] + nums[i]) {
                dp[i] = dp[j] + nums[i];
            }
        }
        maxSum = max(maxSum, dp[i]);
    }
    return maxSum;
}

int main() {
    cout << "Maximum Sum Increasing Subsequence is: ---> " << findMSIS({4, 1, 2, 6, 10, 1, 12}) << endl;
    cout << "Maximum Sum Increasing Subsequence is: ---> " << findMSIS({-4, 10, 3, 7, 15}) << endl;
    cout << "Maximum Sum Increasing Subsequence is: ---> " << findMSIS({1, 3, 8, 4, 14, 6, 14, 1, 9, 4, 13, 3, 11, 17, 29}) << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is is `O(n²)` and the <b>space complexity</b> is `O(n)`.

## Shortest Common Super-sequence

https://leetcode.com/problems/shortest-common-supersequence/

> Given two <b>sequences</b> `s1` and `s2`, write a method to find the length of the shortest sequence which has `s1` and `s2` as <b>subsequences</b>.

#### Example 1:

```text
Input: s1: "abcf" s2:"bdcf"
Output: 5
Explanation: The shortest common super-sequence (SCS) is "abdcf".
```

#### Example 2:

```text
Input: s1: "dynamic" s2:"programming"
Output: 15
Explanation: The SCS is "dynprogrammicng".
```

### Basic Brute-Force Solution

The problem is quite similar to the <b>[Longest Common Subsequence](#🔎-longest-common-subsequence)</b>.

A <b>basic brute-force solution</b> could be to try all the <b>super-sequences</b> of the given <b>sequences</b>. We can process both of the <b>sequences</b> one character at a time, so at any step, we must choose between:

1. If the <b>sequences</b> have a matching character, we can skip one character from both the <b>sequences</b> and make a <i>recursive call</i> for the remaining lengths to get <b>SCS</b>.
2. If the strings don’t match, we start two new <i>recursive calls</i> by skipping one character separately from each string. The minimum of these two <i>recursive calls</i> will have our answer.

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

int scsRec(const string& s1, const string& s2, int i1, int i2) {
    if (i1 == (int)s1.size()) return (int)s2.size() - i2;
    if (i2 == (int)s2.size()) return (int)s1.size() - i1;
    if (s1[i1] == s2[i2]) return 1 + scsRec(s1, s2, i1 + 1, i2 + 1);
    int len1 = 1 + scsRec(s1, s2, i1, i2 + 1);
    int len2 = 1 + scsRec(s1, s2, i1 + 1, i2);
    return min(len1, len2);
}

int findSCSLength(const string& s1, const string& s2) {
    return scsRec(s1, s2, 0, 0);
}

int main() {
    cout << "Length of Shortest Common Subsequence: Substring: ---> " << findSCSLength("dynamic", "programming") << endl;
    cout << "Length of Shortest Common Subsequence: Substring: ---> " << findSCSLength("abcf", "bdcf") << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ᵐ⁺ⁿ)`, where `m` and `n` are the lengths of the input <b>sequences</b>.
- The <b>space complexity</b> is `O(m+n)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

Let's use <b>memoization</b> to overcome the <i>overlapping subproblems</i>.

The two changing values to our <b>recursive function</b> are the two indices, `index1` and `index2`. Therefore, we can store the results of all the <i>subproblems</i> in a two-dimensional array. (Another alternative could be to use a <i>hash-table</i> whose key would be a string (`index1` + `|` + `index2`)).

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int scsRec(const string& s1, const string& s2, int i1, int i2, vector<vector<int>>& dp) {
    if (i1 == (int)s1.size()) return (int)s2.size() - i2;
    if (i2 == (int)s2.size()) return (int)s1.size() - i1;
    if (dp[i1][i2] != -1) return dp[i1][i2];
    if (s1[i1] == s2[i2]) {
        return dp[i1][i2] = 1 + scsRec(s1, s2, i1 + 1, i2 + 1, dp);
    }
    int len1 = 1 + scsRec(s1, s2, i1, i2 + 1, dp);
    int len2 = 1 + scsRec(s1, s2, i1 + 1, i2, dp);
    return dp[i1][i2] = min(len1, len2);
}

int findSCSLength(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.size(), vector<int>(s2.size(), -1));
    return scsRec(s1, s2, 0, 0, dp);
}

int main() {
    cout << "Length of Shortest Common Subsequence: Substring: ---> " << findSCSLength("dynamic", "programming") << endl;
    cout << "Length of Shortest Common Subsequence: Substring: ---> " << findSCSLength("abcf", "bdcf") << endl;
    return 0;
}
```

### Bottom-up Dynamic Programming

Since we want to match all the <b>subsequences</b> of the given <b>sequences</b>, we can use a two-dimensional array to store our results. The lengths of the two strings will define the size of the array’s dimensions. So for every index `i` in sequence `s1` and `j` in sequence `s2`, we will choose one of the following two options:

1. If the character `s1[i]` matches `s2[j]`, the length of the <b>SCS</b> would be the one plus the length of the <b>SCS</b> until `i-1` and `j-1` indices in the two strings.
2. If the character `s1[i]` does not match `s2[j]`, we will consider two <b>SCS</b>:

- one without `s1[i]` and one without `s2[j]`.
- Our required <b>SCS</b> length will be the shortest of these two super-sequences plus one.

So our <b>recursive formula</b> would be:

```cpp
if (s1[i] == s2[j])
  dp[i][j] = 1 + dp[i-1][j-1];
else
  dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1]);
```

Here is the code for our <b>bottom-up dynamic programming</b> approach:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int findSCSLength(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.size() + 1, vector<int>(s2.size() + 1, 0));
    for (size_t i = 0; i <= s1.size(); ++i) dp[i][0] = i;
    for (size_t j = 0; j <= s2.size(); ++j) dp[0][j] = j;

    for (size_t i = 1; i <= s1.size(); ++i) {
        for (size_t j = 1; j <= s2.size(); ++j) {
            if (s1[i - 1] == s2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + min(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[s1.size()][s2.size()];
}

int main() {
    cout << "Length of Shortest Common Subsequence: Substring: ---> " << findSCSLength("dynamic", "programming") << endl;
    cout << "Length of Shortest Common Subsequence: Substring: ---> " << findSCSLength("abcf", "bdcf") << endl;
    return 0;
}
```

- The <b>time and space complexity</b> of the above algorithm is `O(n*m)`.

## Minimum Deletions to Make a Sequence Sorted

https://www.geeksforgeeks.org/minimum-number-deletions-make-sorted-sequence/

> Given a number <b>sequence</b>, find the minimum number of elements that should be deleted to make the remaining <b>sequence</b> sorted.

#### Example 1:

```text
Input: {4,2,3,6,10,1,12}
Output: 2
Explanation: We need to delete {4,1} to make the remaing sequence sorted {2,3,6,10,12}.
```

#### Example 2:

```text
Input: {-4,10,3,7,15}
Output: 1
Explanation: We need to delete {10} to make the remaing sequence sorted {-4,3,7,15}.
```

#### Example 3:

```text
Input: {3,2,1,0}
Output: 3
Explanation: Since the elements are in reverse order, we have to delete all except one to get a
sorted sequence. Sorted sequences are {3}, {2}, {1}, and {0}
```

### Basic Brute-force Solution

A <b>basic brute-force solution</b> could be to try deleting all combinations of elements, one by one, and checking if that makes the <b>subsequence</b> sorted.

Alternately, we can convert this problem into a <b>[Longest Increasing Subsequence (LIS)](#👩🏽‍🦯-🔎-longest-increasing-subsequence)</b> problem. As we know that <b>LIS</b> will give us the length of the <b>longest increasing subsequence</b> (in the sorted order!), which means that the elements which are not part of the <b>LIS</b> should be removed to make the <b>sequence</b> sorted. This is exactly what we need. So we’ll get our solution by subtracting the length of <b>LIS</b> from the length of the input array: `Length-of-input-array - LIS()`

Let’s jump directly to the <b>bottom-up dynamic programming</b> solution.

### Bottom-up Dynamic Programming

Here is the code for our <b>bottom-up dynamic programming</b> approach:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findLISLength(const vector<int>& nums) {
    vector<int> dp(nums.size(), 1);
    int maxLength = 1;
    for (size_t i = 1; i < nums.size(); ++i) {
        for (size_t j = 0; j < i; ++j) {
            if (nums[i] > nums[j] && dp[i] <= dp[j]) {
                dp[i] = dp[j] + 1;
                maxLength = max(maxLength, dp[i]);
            }
        }
    }
    return maxLength;
}

int findMinimumDeletions(const vector<int>& nums) {
    return (int)nums.size() - findLISLength(nums);
}

int main() {
    cout << "Minimum deletion needed: ---> " << findMinimumDeletions({4, 2, 3, 6, 10, 1, 12}) << endl;
    cout << "Minimum deletion needed: ---> " << findMinimumDeletions({-4, 10, 3, 7, 15}) << endl;
    cout << "Minimum deletion needed: ---> " << findMinimumDeletions({3, 2, 1, 0}) << endl;
    return 0;
}
```
// Output: 3
// Explanation: Since the elements are in reverse order, we have to delete all except one to get a sorted sequence. Sorted sequences are {3}, {2}, {1}, and {0}
```

- The <b>time complexity</b> of the above algorithm is is `O(n²)` and the <b>space complexity</b> is `O(n)`.

## Longest Repeating Subsequence

https://www.geeksforgeeks.org/longest-repeating-subsequence/

> Given a <b>sequence</b>, find the length of its <b>longest repeating subsequence (LRS)</b>. A <b>repeating subsequence</b> will be <i>the one that appears at least twice in the original <b>sequence</b> and is not overlapping (i.e. none of the corresponding characters in the repeating subsequences have the same index)</i>.

### Example 1:

```text
Input: “t o m o r r o w”
Output: 2
Explanation: The longest repeating subsequence is “or” {tomorrow}.
```

### Example 2:

```text
Input: “a a b d b c e c”
Output: 3
Explanation: The longest repeating subsequence is “a b c” {a a b d b c e c}.
```

### Example 3:

```text
Input: “f m f f”
Output: 2
Explanation: The longest repeating subsequence is “f f” {f m f f, f m f f}.
Please note the second last character is shared in LRS.
```

### Basic Brute-Force Solution

The problem is quite similar to the <b>[Longest Common Subsequence (LCS)](#🔎-longest-common-subsequence)</b>, with two differences:

1. In <b>LCS</b>, we were trying to find the <b>longest common subsequence</b> between the <i>two strings</i>, whereas in <b>LRS</b> we are trying to find the <b>two longest common subsequences</b> within <i>one string</i>.
2. In <b>LRS</b>, every corresponding character in the <b>subsequences</b> should not have the same <i>index</i>.

A <b>basic brute-force solution</b> could be to try all <b>subsequences</b> of the given <b>sequence</b> to find <i>the longest repeating one</i>, but the problem is how to ensure that the <b>LRS</b>’s characters do not have the <i>same index</i>. For this, we can start with two <i>indices</i> in the given <b>sequence</b>, so at any step we have two choices:

1. If the <i>two indices</i> are not the same and the characters at both the <i>indices</i> are same, we can <b>recursively</b> match for the remaining length (i.e. by incrementing both the <i>indices</i>).
2. If the characters at both the <i>indices</i> don’t match, we start two new <b>recursive calls</b> by incrementing each <i>index</i> separately. The <b>LRS</b> would be the one with the highest length from the two <b>recursive calls</b>.

Here is the code:

```cpp
#include <iostream>\n#include <string>\n#include <algorithm>\nusing namespace std;\n\nint findLRSLengthRecursive(const string& str, int index1, int index2), find the length of its longest repeating subsequence (LRS). A repeating subsequence will be the one that appears at least twice in the original sequence and is not overlapping (i.e. none of the corresponding characters in the repeating subsequences have the same index). */

  /*MY THOUGHT PROCESS*/
  // 1 => start with initial index, go through string and check if that char repeats
  // 2 => if it does repeat it could potentially be part of the LRS
  // 3 => repeat step one for each char
  /**/

  int findLRSLengthRecursive(const string& str, int index1, int index2) {
    if (index1 == str.length() || index2 == str.length()) return 0;
    if (index1 != index2 && str[index1] == str[index2])
      return 1 + findLRSLengthRecursive(str, index1 + 1, index2 + 1);
    int call1 = findLRSLengthRecursive(str, index1 + 1, index2);
    int call2 = findLRSLengthRecursive(str, index1, index2 + 1);
    return max(call1, call2);
  }

  return findLRSLengthRecursive(str, 0, 0);
}

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("tomorrow") << ""
);
// Output: 2
// Explanation: The longest repeating subsequence is “or” {tomorrow}.
// tOmORROw

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("aabdbcec") << ""
);
// Output: 3
// Explanation: The longest repeating subsequence is “a b c” {a a b d b c e c}.
// A A B d B C e C

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("fmff") << ""
);
// Output: 2
// Explanation: The longest repeating subsequence is “f f” {f m f f, f m f f}. Please note the second last character is shared in LRS.
// FmFF
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ⁿ)`, where `n` is the lengths of the input <b>sequence</b>.
- The <b>space complexity</b> is `O(n)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

We can use an array to store the already solved <b>subproblems</b>.

The two changing values to our <b>recursive</b> function are the two <i>indices</i>, `index1` and `index2`. Therefore, we can store the results of all the <b>subproblems</b> in a two-dimensional array. (Another alternative could be to use a <b>hash-table</b> whose key would be a string (`index1` + `“|”` + `index2`)).

Here is the code:

```cpp
function findLRSLength(str) {
  dp = [];

  function findLRSLengthRecursive(str, index1, index2) {
    //base case => 0 length or end of str
    if (index1 == str.length() || index2 == str.length()) return 0;

    dp[index1] = dp[index1] || [];

    if (index1 != index2 && str[index1] == str[index2])
      dp[index1][index2] =
        1 + findLRSLengthRecursive(str, index1 + 1, index2 + 1);
    else {
      call1 = findLRSLengthRecursive(str, index1 + 1, index2);
      call2 = findLRSLengthRecursive(str, index1, index2 + 1);
      dp[index1][index2] = max(call1, call2);
    }

    return dp[index1][index2];
  }
  return findLRSLengthRecursive(str, 0, 0);
}

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("tomorrow") << ""
);
// Output: 2
// Explanation: The longest repeating subsequence is “or” {tomorrow}.

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("aabdbcec") << ""
);
// Output: 3
// Explanation: The longest repeating subsequence is “a b c” {a a b d b c e c}.

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("fmff") << ""
);
// Output: 2
// Explanation: The longest repeating subsequence is “f f” {f m f f, f m f f}. Please note the second last character is shared in LRS.
```

### Bottom-up Dynamic Programming

Since we want to match all the <b>subsequences</b> of the given string, we can use a two-dimensional array to store our results. As mentioned above, we will be tracking two <i>indices</i> to overcome the overlapping problem. So for each of the two <i>indices</i>, `index1` and `index2`, we will choose one of the following options:

1. If `index1` and `index2` are different and the character `str[index1]` matches the character `str[index2]`, then the length of the <b>LRS</b> would be one plus the length of <b>LRS</b> up to `index1-1` and `index2-1` <i>indices</i>.
2. If the character at `str[index1]` does not match `str[index2]`, we will take the <b>LRS</b> by either skipping `index1`th or `index2`th character.

So our <b>recursive formula</b> would be:

```cpp
if (index1 != index2 && str[index1] == str[index2])
  dp[index1][index2] = 1 + dp[index1-1][index2-1];
else
  dp[index1][index2] = max(dp[index1-1][index2], dp[index1][index2-1]);
```

Here is the code for our <b>bottom-up dynamic programming</b> approach:

```cpp
function findLRSLength(str) {
  dp = Array(str.length() + 1)
    .fill(0)
    .map(() => Array(str.length() + 1).fill(0));

  maxLength = 0;

  // dp[index1][index2] will be storing the LRS up to str[0... index1-1][0... index2-1]
  // this also means that subsequences of length 0(first row and column of dp[][])
  // will always have a LRS of size 0

  for (index1 = 1; index1 <= str.length(); index1++) {
    for (index2 = 1; index2 <= str.length(); index2++) {
      if (index1 != index2 && str[index1 - 1] == str[index2 - 1]) {
        dp[index1][index2] = 1 + dp[index1 - 1][index2 - 1];
      } else {
        dp[index1][index2] = max(
          dp[index1 - 1][index2],
          dp[index1][index2 - 1]
        );
      }
      maxLength = max(maxLength, dp[index1][index2]);
    }
  }

  return maxLength;
}

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("tomorrow") << ""
);
// Output: 2
// Explanation: The longest repeating subsequence is “or” {tomorrow}.

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("aabdbcec") << ""
);
// Output: 3
// Explanation: The longest repeating subsequence is “a b c” {a a b d b c e c}.

console.log(
  "Length of Longest Repeating Subsequence: ---> " << findLRSLength("fmff") << ""
);
// Output: 2
// Explanation: The longest repeating subsequence is “f f” {f m f f, f m f f}. Please note the second last character is shared in LRS.
```

- The <b>time complexity and space complexity</b> of the above algorithm is is `O(n²)`.

## Subsequence Pattern Matching

https://www.geeksforgeeks.org/find-number-times-string-occurs-given-string/

> Given a `string` and a `pattern`, write a method to count the number of times the `pattern` appears in the `string` as a <b>subsequence</b>.

#### Example 1:

```text
Input: string: “baxmx”, pattern: “ax”
Output: 2
Explanation: {baxmx, baxmx}.
```

#### Example 2:

```text
Input: string: “tomorrow”, pattern: “tor”
Output: 4
Explanation: Following are the four occurences: {tomorrow, tomorrow, tomorrow, tomorrow}.
```

### Basic Brute-force Solution

This problem follows the <b>[Longest Common Subsequence (LCS) pattern](#pattern-5-longest-common-substring)</b> and is quite similar to the <b>[Longest Repeating Subsequence](#longest-repeating-subsequence)</b>; the difference is that we need to count the total occurrences of the <b>subsequence</b>.

A <b>basic brute-force solution</b> could be to try all the <b>subsequences</b> of the given `string` to count all that match the given `pattern`. We can match the pattern with the given `string` one character at a time, so we can do two things at any step:

1. If the `pattern` has a matching character with the `string`, we can <b>recursively</b> match for the remaining lengths of the `pattern` and the `string`.
2. At every step, we can always skip a character from the `string` to try to match the remaining `string` with the `pattern`. So we can start a <b>recursive call</b> by skipping one character from the `string`.

Our total `count` will be the sum of the `count`s returned by the above two options.

Here is the code:

```cpp
#include <iostream>
#include <string>
using namespace std;

int spmRec(const string& s, const string& p, int si, int pi) {
    if (pi == (int)p.size()) return 1;
    if (si == (int)s.size()) return 0;
    int c1 = 0;
    if (s[si] == p[pi]) {
        c1 = spmRec(s, p, si + 1, pi + 1);
    }
    int c2 = spmRec(s, p, si + 1, pi);
    return c1 + c2;
}

int findSPMCount(const string& s, const string& p) {
    return spmRec(s, p, 0, 0);
}

int main() {
    cout << "Count of pattern in the string: ---> " << findSPMCount("baxmx", "ax") << endl;
    cout << "Count of pattern in the string: ---> " << findSPMCount("tomorrow", "tor") << endl;
    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ᵐ)`, where `m` is the length of the `string`, as our <i>recursion stack</i> will not be deeper than `m`. The <b>space complexity</b> is `O(m)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

We can use an array to store the already solved <i>subproblems</i>.

The two changing values to our <b>recursive function</b> are the two <i>indices</i> `strIndex` and `patternIndex`. Therefore, we can store the results of all the <i>subproblems</i> in a two-dimensional array. (Another alternative could be to use a <b>hash-table</b> whose key would be a `string` (`strIndex` + `“|”` + `patternIndex`)).

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

int spmRec(const string& s, const string& p, int si, int pi, vector<vector<int>>& dp) {
    if (pi == (int)p.size()) return 1;
    if (si == (int)s.size()) return 0;
    if (dp[si][pi] != -1) return dp[si][pi];
    int c1 = 0;
    if (s[si] == p[pi]) {
        c1 = spmRec(s, p, si + 1, pi + 1, dp);
    }
    int c2 = spmRec(s, p, si + 1, pi, dp);
    return dp[si][pi] = c1 + c2;
}

int findSPMCount(const string& s, const string& p) {
    vector<vector<int>> dp(s.size(), vector<int>(p.size(), -1));
    return spmRec(s, p, 0, 0, dp);
}

int main() {
    cout << "Count of pattern in the string: ---> " << findSPMCount("baxmx", "ax") << endl;
    cout << "Count of pattern in the string: ---> " << findSPMCount("tomorrow", "tor") << endl;
    return 0;
}
```

### Bottom-up Dynamic Programming

Since we want to match all the <b>subsequences</b> of the given `string`, we can use a two-dimensional array to store our results. As mentioned above, we will be tracking separate <i>indices</i> for the `string` and the `pattern`, so we will be doing two things for every value of `strIndex` and `patternIndex`:

1. If the character at the `strIndex` (in the `string`) matches the character at `patIndex` (in the `pattern`), the count of the <b>SPM</b> would be equal to the count of <b>SPM</b> up to `strIndex-1` and `patternIndex-1`.
2. At every step, we can always skip a character from the `string` to try matching the remaining `string` with the `pattern`; therefore, we can add the <b>SPM</b> count from the <i>indices</i> `strIndex-1` and `patternIndex`.

So our <b>recursive formula</b> would be:

```cpp
if (str[strIndex] == pat[patIndex]) {
  dp[strIndex][patIndex] = dp[strIndex-1][patIndex-1];
}
dp[strIndex][patIndex] += dp[strIndex-1][patIndex];
```

Here is the code for our <b>bottom-up dynamic programming</b> approach:

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

int findSPMCount(const string& s, const string& p) {
    if (p.empty()) return 1;
    if (s.empty() || p.size() > s.size()) return 0;
    vector<vector<int>> dp(s.size() + 1, vector<int>(p.size() + 1, 0));
    for (size_t i = 0; i <= s.size(); ++i) dp[i][0] = 1;
    for (size_t si = 1; si <= s.size(); ++si) {
        for (size_t pi = 1; pi <= p.size(); ++pi) {
            if (s[si - 1] == p[pi - 1]) {
                dp[si][pi] = dp[si - 1][pi - 1];
            }
            dp[si][pi] += dp[si - 1][pi];
        }
    }
    return dp[s.size()][p.size()];
}

int main() {
    cout << "Count of pattern in the string: ---> " << findSPMCount("baxmx", "ax") << endl;
    cout << "Count of pattern in the string: ---> " << findSPMCount("tomorrow", "tor") << endl;
    return 0;
}
```

- The <b>time and space complexity</b> of the above algorithm is `O(m*n)`, where `m` and `n` are the lengths of the `string` and the `pattern` respectively.

## Longest Bitonic Subsequence

https://www.geeksforgeeks.org/longest-bitonic-subsequence-dp-15/

> Given a number sequence, find the length of its <b>Longest Bitonic Subsequence (LBS)</b>. <i>A <b>subsequence</b> is considered <b>bitonic</b> if it is monotonically increasing and then monotonically decreasing</i>.

#### Example 1:

```text
Input: {4,2,3,6,10,1,12}
Output: 5
Explanation: The LBS is {2,3,6,10,1}.
```

#### Example 2:

```text
Input: {4,2,5,9,7,6,10,3,1}
Output: 7
Explanation: The LBS is {4,5,9,7,6,3,1}.
```

### Basic Solution

A <b>basic brute-force solution</b> could be to try finding the <b>Longest Decreasing Subsequences (LDS)</b>, starting from every number in both directions. So for every index `i` in the given array, we will do two things:

1. Find <b>LDS</b> starting from `i` to the end of the array.
2. Find <b>LDS</b> starting from `i` to the beginning of the array.

<b>Longest Bitonic Subsequence (LBS)</b> would be the maximum sum of the above two <i>subsequences</i>.

Here is the code:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

int findLDSLength(const vector<int>& nums, int currIndex, int prevIndex) {
  //find the longest decreasing subsequence from
  //currIndex to the end of the array
  if (currIndex == nums.size()) return 0;

  //include nums[currIndex] if it is smaller than previous number
  int checkWithCurrIndex = 0;

  if (prevIndex == -1 || nums[currIndex] < nums[prevIndex]) {
    checkWithCurrIndex = 1 + findLDSLength(nums, currIndex + 1, currIndex);
  }

  //excluding the number at currIndex
  int checkExcludingCurrIndex = findLDSLength(nums, currIndex + 1, prevIndex);

  return max(checkWithCurrIndex, checkExcludingCurrIndex);
}

int findLDSLengthReverse(const vector<int>& nums, int currIndex, int prevIndex) {
  //find the longest decreasing subsequence from
  //currIndex to the beginning of the array
  if (currIndex < 0) return 0;

  // include nums[currIndex] if it is smaller than the prev number
  int checkWithCurrIndex = 0;

  if (prevIndex == -1 || nums[currIndex] < nums[prevIndex]) {
    checkWithCurrIndex =
        1 + findLDSLengthReverse(nums, currIndex - 1, currIndex);
  }

  //excluding the number at currIndex
  int checkExcludingCurrIndex =
      findLDSLengthReverse(nums, currIndex - 1, prevIndex);

  return max(checkWithCurrIndex, checkExcludingCurrIndex);
}

int findLBSLength(const vector<int>& nums) {
  int maxLength = 0;

  for (int i = 0; i < nums.size(); i++) {
    int checkIndexToEnd = findLDSLength(nums, i, -1);
    int checkIndexToStart = findLDSLengthReverse(nums, i, -1);
    // cout << checkIndexToEnd << " " << checkIndexToStart << " " << checkIndexToEnd + checkIndexToStart - 1 << endl;
    maxLength = max(maxLength, checkIndexToEnd + checkIndexToStart - 1);
  }
  return maxLength;
}

int main() {
  cout << "Length of Longest Bitonic Subsequence: ---> "
       << findLBSLength({4, 2, 3, 6, 10, 1, 12}) << endl;
  // Output: 5
  // Explanation: The LBS is {2,3,6,10,1}.

  cout << "Length of Longest Bitonic Subsequence: ---> "
       << findLBSLength({4, 2, 5, 9, 7, 6, 10, 3, 1}) << endl;
  // Output: 7
  // Explanation: The LBS is {4,5,9,7,6,3,1}.
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ⁿ)`, where `n` is the lengths of the input array.
- The <b>space complexity</b> is `O(n)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

To overcome the <b>overlapping subproblems</b>, we can use an array to store the already solved <b>subproblems</b>.

We need to <b>memoize</b> the <i>recursive functions</i> that calculate the <b>longest decreasing subsequence</b>. The two changing values for our <i>recursive function</i> are the `current` and the `previous` index. Therefore, we can store the results of all <b>subproblems</b> in a two-dimensional array. (Another alternative could be to use a <i>hash-table</i> whose key would be a string (`currIndex` + `“|”` + `prevIndex`)).

Here is the code:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findLBSLength(const vector<int>& nums) {
    int n = (int)nums.size();
    vector<vector<int>> lds(n, vector<int>(n + 1, -1));
    vector<vector<int>> ldsReversed(n, vector<int>(n + 1, -1));

    function<int(int, int, vector<vector<int>>&)> findLDSLength = 
        [&](int currIndex, int prevIndex, vector<vector<int>>& memo) {
        if (currIndex == n) return 0;
        if (memo[currIndex][prevIndex + 1] != -1) return memo[currIndex][prevIndex + 1];

        int checkWithCurrIndex = 0;
        if (prevIndex == -1 || nums[currIndex] < nums[prevIndex]) {
            checkWithCurrIndex = 1 + findLDSLength(currIndex + 1, currIndex, memo);
        }
        int checkExcludingCurrIndex = findLDSLength(currIndex + 1, prevIndex, memo);
        return memo[currIndex][prevIndex + 1] = max(checkWithCurrIndex, checkExcludingCurrIndex);
    };

    function<int(int, int, vector<vector<int>>&)> findLDSLengthReverse = 
        [&](int currIndex, int prevIndex, vector<vector<int>>& memo) {
        if (currIndex < 0) return 0;
        if (memo[currIndex][prevIndex + 1] != -1) return memo[currIndex][prevIndex + 1];

        int checkWithCurrIndex = 0;
        if (prevIndex == -1 || nums[currIndex] < nums[prevIndex]) {
            checkWithCurrIndex = 1 + findLDSLengthReverse(currIndex - 1, currIndex, memo);
        }
        int checkExcludingCurrIndex = findLDSLengthReverse(currIndex - 1, prevIndex, memo);
        return memo[currIndex][prevIndex + 1] = max(checkWithCurrIndex, checkExcludingCurrIndex);
    };

    int maxLength = 0;
    for (int i = 0; i < n; i++) {
        int checkIndexToEnd = findLDSLength(i, -1, lds);
        int checkIndexToStart = findLDSLengthReverse(i, -1, ldsReversed);
        maxLength = max(maxLength, checkIndexToEnd + checkIndexToStart - 1);
    }

    return maxLength;
}

int main() {
    cout << "Length of Longest Bitonic Subsequence: ---> " 
         << findLBSLength({4, 2, 3, 6, 10, 1, 12}) << endl;
    // Output: 5
    // Explanation: The LBS is {2,3,6,10,1}.

    cout << "Length of Longest Bitonic Subsequence: ---> " 
         << findLBSLength({4, 2, 5, 9, 7, 6, 10, 3, 1}) << endl;
    // Output: 7
    // Explanation: The LBS is {4,5,9,7,6,3,1}.

    return 0;
}
```

### Bottom-up Dynamic Programming

The above algorithm shows us a clear <b>bottom-up approach</b>. We can separately calculate <b>LDS</b> for every index i.e., from the beginning to the end of the array and vice versa. The required length of <b>LBS</b> would be the one that has the maximum sum of <b>LDS</b> for a given index (from both ends).

Here is the code for our <b>bottom-up dynamic programming approach</b>:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findLBSLength(const vector<int>& nums) {
    int n = (int)nums.size();
    vector<int> lds(n, 0);
    vector<int> ldsReversed(n, 0);

    // Find the longest decreasing subsequence from currIndex to the end of the array
    for (int i = 0; i < n; i++) {
        lds[i] = 1;
        for (int j = i - 1; j >= 0; j--) {
            if (nums[j] < nums[i]) {
                lds[i] = max(lds[i], lds[j] + 1);
            }
        }
    }

    // Find the longest decreasing subsequence from currIndex to the beginning of the array
    for (int i = n - 1; i >= 0; i--) {
        ldsReversed[i] = 1;
        for (int j = i + 1; j < n; j++) {
            if (nums[j] < nums[i]) {
                ldsReversed[i] = max(ldsReversed[i], ldsReversed[j] + 1);
            }
        }
    }

    int maxLength = 0;
    for (int i = 0; i < n; i++) {
        maxLength = max(maxLength, lds[i] + ldsReversed[i] - 1);
    }

    return maxLength;
}

int main() {
    cout << "Length of Longest Bitonic Subsequence: ---> " 
         << findLBSLength({4, 2, 3, 6, 10, 1, 12}) << endl;
    // Output: 5
    // Explanation: The LBS is {2,3,6,10,1}.

    cout << "Length of Longest Bitonic Subsequence: ---> " 
         << findLBSLength({4, 2, 5, 9, 7, 6, 10, 3, 1}) << endl;
    // Output: 7
    // Explanation: The LBS is {4,5,9,7,6,3,1}.

    return 0;
}
```

- The <b>time omplexity</b> of the above algorithm is `O(n²)` and the <b>space complexity</b> is `O(n)`.

## Longest Alternating Subsequence

https://www.geeksforgeeks.org/longest-alternating-subsequence/

> Given a number sequence, find the length of its <b>Longest Alternating Subsequence (LAS)</b>. A <b>subsequence</b> is considered <b>alternating</b> if its elements are in <b>alternating</b> order.

A three element sequence (`a1, a2, a3`) will be an <b>alternating sequence</b> if its elements hold one of the following conditions:

`{a1 > a2 < a3 }` or `{ a1 < a2 > a3}`.

#### Example 1:

```text
Input: {1,2,3,4}
Output: 2
Explanation: There are many LAS: {1,2}, {3,4}, {1,3}, {1,4}
```

#### Example 2:

```text
Input: {3,2,1,4}
Output: 3
Explanation: The LAS are {3,2,4} and {2,1,4}.
```

#### Example 3:

```text
Input: {1,3,2,4}
Output: 4
Explanation: The LAS is {1,3,2,4}.
```

### Basic Solution

A <b>basic brute-force solution</b> could be to try finding the <b>LAS</b> starting from every number in both <i>ascending</i> and <i>descending</i> order. So for every index `i` in the given array, we will have three options:

1. If the element at `i` is bigger than the last element we considered, we include the element at `i` and recursively process the remaining array to find the next element in <i>descending</i> order.
2. If the element at `i` is smaller than the last element we considered, we include the element at `i` and recursively process the remaining array to find the next element in <i>ascending</i> order.
3. In addition to the above two cases, we can always skip the element `i` to recurse for the remaining array. This will ensure that we try all subsequences.

<b>LAS</b> would be the maximum of the above three subsequences.

Here is the code:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findLASLength(const vector<int>& nums) {
    int n = (int)nums.size();

    function<int(int, int, bool)> findLASLengthRecursive = 
        [&](int prevIndex, int currIndex, bool isAscending) {
        if (currIndex == n) return 0;

        int condition1 = 0;
        // If ascending, the next element should be bigger
        if (isAscending) {
            if (prevIndex == -1 || nums[prevIndex] < nums[currIndex]) {
                condition1 = 1 + findLASLengthRecursive(currIndex, currIndex + 1, !isAscending);
            }
        } else {
            // If descending, the next element should be smaller
            if (prevIndex == -1 || nums[prevIndex] > nums[currIndex]) {
                condition1 = 1 + findLASLengthRecursive(currIndex, currIndex + 1, !isAscending);
            }
        }

        // OR skip the current element
        int condition2 = findLASLengthRecursive(prevIndex, currIndex + 1, isAscending);
        return max(condition1, condition2);
    };

    // Start with two recursive calls
    return max(findLASLengthRecursive(-1, 0, true), findLASLengthRecursive(-1, 0, false));
}

int main() {
    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({1, 2, 3, 4}) << endl;
    // Output: 2
    // Explanation: There are many LAS: {1,2}, {3,4}, {1,3}, {1,4}

    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({3, 2, 1, 4}) << endl;
    // Output: 3
    // Explanation: The LAS are {3,2,4} and {2,1,4}.

    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({1, 3, 2, 4}) << endl;
    // Output: 4
    // Explanation: The LAS is {1,3,2,4}.

    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is exponential `O(2ᴺ)`, where `n` is the lengths' of the input array.
- The <b>space complexity</b> is `O(n)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

To overcome the <b>overlapping subproblems</b>, we can use an array to store the already solved <b>subproblems</b>.

The three changing values for our <i>recursive function</i> are the `currIndex` and `prevIndex` and the `isAscending` flag. Therefore, we can store the results of all <b>subproblems</b> in a three-dimensional array, where the third dimension will be of size two, to store the <i>boolean</i> flag `isAscending`. (Another alternative could be to use a <i>hash-table</i> whose key would be a string (`currIndex` + `“|”` + `prevIndex`+ `“|”` + `isAscending`)).

Here is the code:

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>
#include <algorithm>
using namespace std;

int findLASLength(const vector<int>& nums) {
    int n = (int)nums.size();
    map<string, int> dp;

    function<int(int, int, bool)> findLASLengthRecursive = 
        [&](int prevIndex, int currIndex, bool isAscending) {
        if (currIndex == n) return 0;

        string key = to_string(prevIndex) + "-" + to_string(currIndex) + "-" + (isAscending ? "1" : "0");
        if (dp.find(key) != dp.end()) return dp[key];

        int condition1 = 0;
        // If ascending, the next element should be bigger
        if (isAscending) {
            if (prevIndex == -1 || nums[prevIndex] < nums[currIndex]) {
                condition1 = 1 + findLASLengthRecursive(currIndex, currIndex + 1, !isAscending);
            }
        } else {
            // If descending, the next element should be smaller
            if (prevIndex == -1 || nums[prevIndex] > nums[currIndex]) {
                condition1 = 1 + findLASLengthRecursive(currIndex, currIndex + 1, !isAscending);
            }
        }

        // OR skip the current element
        int condition2 = findLASLengthRecursive(prevIndex, currIndex + 1, isAscending);
        return dp[key] = max(condition1, condition2);
    };

    // Start with two recursive calls
    return max(findLASLengthRecursive(-1, 0, true), findLASLengthRecursive(-1, 0, false));
}

int main() {
    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({1, 2, 3, 4}) << endl;
    // Output: 2
    // Explanation: There are many LAS: {1,2}, {3,4}, {1,3}, {1,4}

    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({3, 2, 1, 4}) << endl;
    // Output: 3
    // Explanation: The LAS are {3,2,4} and {2,1,4}.

    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({1, 3, 2, 4}) << endl;
    // Output: 4
    // Explanation: The LAS is {1,3,2,4}.

    return 0;
}
```

### Bottom-up Dynamic Programming

The above algorithm tells us three things:

1. We need to find an <i>ascending</i> and <i>descending</i> <i>subsequence</i> at every index.
2. While finding the next element in the <i>ascending</i> order, if the number at the `currIndex` is bigger than the number at the `prevIndex`, we increment the count for a <b>LAS</b> up to the `currIndex`. But if there is a bigger <b>LAS</b> without including the number at the `currIndex`, we take that.
3. Similarly for the <i>descending</i> order, if the number at the `currIndex` is smaller than the number at the `prevndex`, we increment the count for a <b>LAS</b> up to the `currIndex`. But if there is a bigger <b>LAS</b> without including the number at the `currIndex`, we take that.

To find the largest <b>LAS</b>, we need to find all of the <b>LAS</b> for a number at index `i` from all the previous numbers (i.e. number until index `i-1`).

We can use two arrays to store the length of <b>LAS</b>, one for <i>ascending</i> order and one for <i>descending</i> order. (Actually, we will use a two-dimensional array, where the second dimension will be of size two).

If `i` represents the `currIndex` and `j` represents the `prevIndex`, our recursive formula would look like:

- If `nums[i]` is bigger than `nums[j]` then we will consider the <b>LAS</b> ending at `j` where the last two elements were in <i>descending</i> order =>

```cpp
if num[i] > num[j] => dp[i][0] = 1 + dp[j][1], if there is no bigger LAS for 'i'
```

- If `nums[i]` is smaller than `nums[j]` then we will consider the <b>LAS</b> ending at `j` where the last two elements were in <i>ascending</i> order =>

```cpp
if num[i] < num[j] => dp[i][1] = 1 + dp[j][0], if there is no bigger LAS for 'i'
```

Here is the code for our <b>bottom-up dynamic programming</b> approach:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int findLASLength(const vector<int>& nums) {
    if (nums.empty()) return 0;

    int n = (int)nums.size();
    // dp[i][0] = stores the LAS ending at 'i' such that the last two elements are in ascending order
    // dp[i][1] = stores the LAS ending at 'i' such that the last two elements are in descending order
    vector<vector<int>> dp(n, vector<int>(2, 0));

    int maxLength = 1;
    for (int i = 0; i < n; i++) {
        // every single element can be considered as a LAS of length 1
        dp[i][0] = dp[i][1] = 1;
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                // if nums[i] is BIGGER than nums[j] then we will consider the LAS ending at 'j' where the
                // last two elements were in DESCENDING order
                dp[i][0] = max(dp[i][0], 1 + dp[j][1]);
                maxLength = max(maxLength, dp[i][0]);
            } else if (nums[i] != nums[j]) {
                // if the numbers are equal, don't do anything
                // if nums[i] is SMALLER than nums[j] then we will consider the LAS ending at 'j' where the
                // last two elements were in ASCENDING order
                dp[i][1] = max(dp[i][1], 1 + dp[j][0]);
                maxLength = max(maxLength, dp[i][1]);
            }
        }
    }
    return maxLength;
}

int main() {
    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({1, 2, 3, 4}) << endl;
    // Output: 2
    // Explanation: There are many LAS: {1,2}, {3,4}, {1,3}, {1,4}

    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({3, 2, 1, 4}) << endl;
    // Output: 3
    // Explanation: The LAS are {3,2,4} and {2,1,4}.

    cout << "Length of Longest Alternating Subsequence: ---> " 
         << findLASLength({1, 3, 2, 4}) << endl;
    // Output: 4
    // Explanation: The LAS is {1,3,2,4}.

    return 0;
}
```

- The <b>time complexity</b> of the above algorithm is `O(n²)` and the <b>space complexity</b> is `O(n)`.

## 🔎 Edit Distance

https://leetcode.com/problems/edit-distance/

> Given strings `s1` and `s2`, we need to transform `s1` into `s2` by <i>deleting</i>, <i>inserting</i>, or <i>replacing</i> characters. Write a function to calculate the count of the minimum number of edit operations.

#### Example 1:

```text
Input: s1 = "bat"
       s2 = "but"
Output: 1
Explanation: We just need to replace 'a' with 'u' to transform s1 to s2.
```

#### Example 2:

```text
Input: s1 = "abdca"
       s2 = "cbda"
Output: 2
Explanation: We can replace first 'a' with 'c' and delete second 'c'.
```

#### Example 3:

```text
Input: s1 = "passpot"
       s2 = "ppsspqrt"
Output: 3
Explanation: Replace 'a' with 'p', 'o' with 'q', and insert 'r'.
```

### Basic Solution

A <b>basic brute-force solution</b> could be to try all operations (one by one) on each character of `s1`. We can iterate through `s1` and `s2` together. Let’s assume `index1` and `index2` point to the current indexes of `s1` and `s2` respectively, so we have two options at every step:

1. If the strings have a matching character, we can recursively match for the remaining lengths.
2. If the strings don’t match, we start three new recursive calls representing the three edit operations. Whichever recursive call returns the minimum count of operations will be our answer.

Here is the recursive implementation:

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

int findMinOperationsRecursive(const string& s1, const string& s2, int index1, int index2) {
    // if we have reached the end of s1, then we have to insert all the remaining characters of s2
    if (index1 == (int)s1.length()) return (int)s2.length() - index2;

    // if we have reached the end of s2, then we have to delete all the remaining characters of s1
    if (index2 == (int)s2.length()) return (int)s1.length() - index1;

    // If the strings have a matching character, we can recursively match for the remaining lengths.
    if (s1[index1] == s2[index2]) {
        return findMinOperationsRecursive(s1, s2, index1 + 1, index2 + 1);
    }

    // perform deletion
    int onDeletion = 1 + findMinOperationsRecursive(s1, s2, index1 + 1, index2);
    // perform insertion
    int onInsertion = 1 + findMinOperationsRecursive(s1, s2, index1, index2 + 1);
    // perform replacement
    int onReplacement = 1 + findMinOperationsRecursive(s1, s2, index1 + 1, index2 + 1);

    return min(onDeletion, min(onInsertion, onReplacement));
}

int findMinOperations(const string& s1, const string& s2) {
    return findMinOperationsRecursive(s1, s2, 0, 0);
}

int main() {
    cout << "Minimum Edit Distance: ---> " << findMinOperations("bat", "but") << endl;
    // Output: 1
    // Explanation: We just need to replace 'a' with 'u' to transform s1 to s2.

    cout << "Minimum Edit Distance: ---> " << findMinOperations("abdca", "cbda") << endl;
    // Output: 2
    // Explanation: We can replace first 'a' with 'c' and delete second 'c'.

    cout << "Minimum Edit Distance: ---> " << findMinOperations("passpot", "ppsspqrt") << endl;
    // Output: 3
    // Explanation: Replace 'a' with 'p', 'o' with 'q', and insert 'r'.

    return 0;
}
```

- Because of the three recursive calls, the <b>time complexity</b> of the above algorithm is exponential `O(3ᵐ⁺ⁿ)`, where `m` and `n` are the lengths of the two input strings.
- The <b>space complexity</b> is `O(m+n)` which is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

We can use an array to store the already solved <b>subproblems</b>.

The two changing values in our recursive function are the two indexes, `index1` and `index2`. Therefore, we can store the results of all the subproblems in a two-dimensional array. (Another alternative could be to use a hash-table whose key would be a string (`index1` + `“|”` + `index2`)).

Here is the code for the <b>Top-down Dynamic Programming</b>approach:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int findMinOperations(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.length(), vector<int>(s2.length(), -1));

    function<int(int, int)> findMinOperationsRecursive = [&](int index1, int index2) {
        // if we have reached the end of s1, then we have to insert all the remaining characters of s2
        if (index1 == (int)s1.length()) return (int)s2.length() - index2;
        // if we have reached the end of s2, then we have to delete all the remaining characters of s1
        if (index2 == (int)s2.length()) return (int)s1.length() - index1;

        if (dp[index1][index2] != -1) return dp[index1][index2];

        // If the strings have a matching character, we can recursively match for the remaining lengths.
        if (s1[index1] == s2[index2])
            return dp[index1][index2] = findMinOperationsRecursive(index1 + 1, index2 + 1);

        // perform deletion
        int onDeletion = findMinOperationsRecursive(index1 + 1, index2);
        // perform insertion
        int onInsertion = findMinOperationsRecursive(index1, index2 + 1);
        // perform replacement
        int onReplacement = findMinOperationsRecursive(index1 + 1, index2 + 1);

        return dp[index1][index2] = 1 + min(onDeletion, min(onInsertion, onReplacement));
    };

    return findMinOperationsRecursive(0, 0);
}

int main() {
    cout << "Minimum Edit Distance: ---> " << findMinOperations("bat", "but") << endl;
    // Output: 1
    // Explanation: We just need to replace 'a' with 'u' to transform s1 to s2.

    cout << "Minimum Edit Distance: ---> " << findMinOperations("abdca", "cbda") << endl;
    // Output: 2
    // Explanation: We can replace first 'a' with 'c' and delete second 'c'.

    cout << "Minimum Edit Distance: ---> " << findMinOperations("passpot", "ppsspqrt") << endl;
    // Output: 3
    // Explanation: Replace 'a' with 'p', 'o' with 'q', and insert 'r'.

    return 0;
}
```

- Since our <i>memoization</i> array `dp[s1.length()][s2.length()]` stores the results for all the <b>subproblems</b>, we can conclude that we will not have more than `m*n` <b>subproblems</b> (where `m` and `n` are the lengths of the two input strings.). This means that our <b>time complexity</b> will be `O(m*n)`.
- The above algorithm will be using `O(m*n)`space for the<i> memoization</i> array. Other than that we will use `O(m+n)`space for the <b>recursion call-stack</b>. So the total <b>space complexity</b> will be `O(m*n + (m+n))`, which is asymptotically equivalent to `O(m*n)`.

### Bottom-up Dynamic Programming

Since we want to match all the characters of the given two strings, we can use a two-dimensional array to store our results. The lengths of the two strings will define the size of the two dimensions of the array. So for every index `index1` in string `s1` and `index2` in string `s2`, we will choose one of the following options:

1. If the character `s1[index1]` matches `s2[index2]`, the count of the edit operations will be equal to the count of the edit operations for the remaining strings.
2. If the character `s1[index1]` does not match `s2[index2]`, we will take the minimum count from the remaining strings after performing any of the three edit operations.

So our <b>recursive formula</b> would be:

```cpp
if s1[index1] == s2[index2]
  dp[index1][index2] = dp[index1-1][index2-1]
else
  dp[index1][index2] = 1 + min(dp[index1-1][index2], // delete
                       dp[index1][index2-1], // insert
                       dp[index1-1][index2-1]) // replace
```

Here is the code for our <b>bottom-up dynamic programming</b> approach:

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int findMinOperations(const string& s1, const string& s2) {
    vector<vector<int>> dp(s1.length() + 1, vector<int>(s2.length() + 1, 0));

    // if s2 is empty, we can remove all the characters of s1 to make it empty too
    for (size_t index1 = 0; index1 <= s1.length(); index1++) dp[index1][0] = index1;

    // if s1 is empty, we have to insert all the characters of s2
    for (size_t index2 = 0; index2 <= s2.length(); index2++) dp[0][index2] = index2;

    for (size_t index1 = 1; index1 <= s1.length(); index1++) {
        for (size_t index2 = 1; index2 <= s2.length(); index2++) {
            // If the strings have a matching character, we can recursively match for the remaining lengths
            if (s1[index1 - 1] == s2[index2 - 1]) {
                dp[index1][index2] = dp[index1 - 1][index2 - 1];
            } else {
                dp[index1][index2] = 1 + min({dp[index1 - 1][index2], dp[index1][index2 - 1], dp[index1 - 1][index2 - 1]});
            }
        }
    }

    return dp[s1.length()][s2.length()];
}

int main() {
    cout << "Minimum Edit Distance: ---> " << findMinOperations("bat", "but") << endl;
    // Output: 1
    // Explanation: We just need to replace 'a' with 'u' to transform s1 to s2.

    cout << "Minimum Edit Distance: ---> " << findMinOperations("abdca", "cbda") << endl;
    // Output: 2
    // Explanation: We can replace first 'a' with 'c' and delete second 'c'.

    cout << "Minimum Edit Distance: ---> " << findMinOperations("passpot", "ppsspqrt") << endl;
    // Output: 3
    // Explanation: Replace 'a' with 'p', 'o' with 'q', and insert 'r'.

    return 0;
}
```

- The <b>time complexity and space complexity</b> of the above algorithm is `O(n*m)`, where `m` and `n` are the lengths of the two input strings.

## 🔎 Strings Interleaving

https://leetcode.com/problems/interleaving-string/

> Given three strings `m`, `n`, and `p`, write a method to find out if `p` has been formed by interleaving `m` and `n`. `p` would be considered interleaving `m` and `n` if it contains all the letters from `m` and `n` and the order of letters is preserved too.

#### Example 1:

```text
Input: m="abd", n="cef", p="abcdef"
Output: true
Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.
```

#### Example 2:

```text
Input: m="abd", n="cef", p="adcbef"
Output: false
Explanation: 'p' contains all the letters from 'm' and 'n' but does not preserve the order.
```

#### Example 3:

```text
Input: m="abc", n="def", p="abdccf"
Output: false
Explanation: 'p' does not contain all the letters from 'm' and 'n'.
```

#### Example 3:

```text
Input: m="abcdef", n="mnop", p="mnaobcdepf"
Output: true
Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.
```

### Basic Solution

The problem follows the <b>[Longest Common Subsequence (LCS) pattern](#pattern-5-longest-common-substring)</b> and has some similarities with <b>[Subsequence Pattern Matching](#subsequence-pattern-matching)</b>.

A <b>basic brute-force solution</b>could be to try matching `m` and `n` with `p` one letter at a time. Let’s assume `mIndex`, `nIndex`, and `pIndex` represent the current indexes of `m`, `n`, and `p` strings respectively. Therefore, we have two options at any step:

1. If the letter at `mIndex` matches with the letter at `pIndex`, we can recursively match for the remaining lengths of `m` and `p`.
2. If the letter at `nIndex` matches with the letter at `pIndex`, we can recursively match for the remaining lengths of `n` and `p`.

<b>LAS</b> would be the maximum of the above three subsequences.

Here is the code:

```cpp
#include <iostream>
#include <string>
using namespace std;

bool findSIRecursive(const string& m, const string& n, const string& p, int mIndex, int nIndex, int pIndex) {
    // if we have reached the end of the all the strings
    if (mIndex == (int)m.length() && nIndex == (int)n.length() && pIndex == (int)p.length())
        return true;

    // if we have reached the end of 'p' but 'm' or 'n' still have some characters left
    if (pIndex == (int)p.length()) return false;

    bool mMatchesP = false;
    bool nMatchesP = false;

    if (mIndex < (int)m.length() && m[mIndex] == p[pIndex])
        mMatchesP = findSIRecursive(m, n, p, mIndex + 1, nIndex, pIndex + 1);

    if (nIndex < (int)n.length() && n[nIndex] == p[pIndex])
        nMatchesP = findSIRecursive(m, n, p, mIndex, nIndex + 1, pIndex + 1);

    return mMatchesP || nMatchesP;
}

bool findSI(const string& m, const string& n, const string& p) {
    return findSIRecursive(m, n, p, 0, 0, 0);
}

int main() {
    cout << "String interleaving: ---> " << (findSI("abd", "cef", "abcdef") ? "true" : "false") << endl;
    // Output: true
    // Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.

    cout << "String interleaving: ---> " << (findSI("abd", "cef", "adcbef") ? "true" : "false") << endl;
    // Output: false
    // Explanation: 'p' contains all the letters from 'm' and 'n' but does not preserve the order.

    cout << "String interleaving: ---> " << (findSI("abc", "def", "abdccf") ? "true" : "false") << endl;
    // Output: false
    // Explanation: 'p' does not contain all the letters from 'm' and 'n'.

    cout << "String interleaving: ---> " << (findSI("abcdef", "mnop", "mnaobcdepf") ? "true" : "false") << endl;
    // Output: true
    // Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.

    return 0;
}
```

- The <b>time complexity</b>of the above algorithm is exponential `O(2ᵐ⁺ⁿ)`
  , where `m` and `n` are the lengths of the two interleaving strings.
- The <b>space complexity</b> is `O(m+n)`, the value that is used to store the <i>recursion stack</i>.

### Top-down Dynamic Programming with Memoization

This problem can have <b>overlapping subproblems</b> only when there are some common letters between `m` and `n` at the same index. Because whenever we hit such a scenario, we get an option to match with any one of them.

The three changing values in our recursive function are the three indexes `mIndex`, `nIndex`, and `pIndex`. Therefore, we can store the results of all the <b>subproblems</b> in a three-dimensional array. Alternately, we can use a hash-table whose key would be a string (`mIndex` + `“|”` +`nIndex` + `“|”` + `pIndex`).

Here is the code:

```cpp
#include <iostream>
#include <string>
#include <map>
using namespace std;

bool findSI(const string& m, const string& n, const string& p) {
    map<string, bool> dp;

    function<bool(int, int, int)> findSIRecursive = [&](int mIndex, int nIndex, int pIndex) {
        // if we have reached the end of the all the strings
        if (mIndex == (int)m.length() && nIndex == (int)n.length() && pIndex == (int)p.length())
            return true;

        // if we have reached the end of 'p' but 'm' or 'n' still have some characters left
        if (pIndex == (int)p.length()) return false;

        string key = to_string(mIndex) + "-" + to_string(nIndex) + "-" + to_string(pIndex);
        if (dp.find(key) != dp.end()) return dp[key];

        bool mMatchesP = false;
        bool nMatchesP = false;

        if (mIndex < (int)m.length() && m[mIndex] == p[pIndex])
            mMatchesP = findSIRecursive(mIndex + 1, nIndex, pIndex + 1);

        if (nIndex < (int)n.length() && n[nIndex] == p[pIndex])
            nMatchesP = findSIRecursive(mIndex, nIndex + 1, pIndex + 1);

        return dp[key] = mMatchesP || nMatchesP;
    };

    return findSIRecursive(0, 0, 0);
}

int main() {
    cout << "String interleaving: ---> " << (findSI("abd", "cef", "abcdef") ? "true" : "false") << endl;
    // Output: true
    // Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.

    cout << "String interleaving: ---> " << (findSI("abd", "cef", "adcbef") ? "true" : "false") << endl;
    // Output: false
    // Explanation: 'p' contains all the letters from 'm' and 'n' but does not preserve the order.

    cout << "String interleaving: ---> " << (findSI("abc", "def", "abdccf") ? "true" : "false") << endl;
    // Output: false
    // Explanation: 'p' does not contain all the letters from 'm' and 'n'.

    cout << "String interleaving: ---> " << (findSI("abcdef", "mnop", "mnaobcdepf") ? "true" : "false") << endl;
    // Output: true
    // Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.

    return 0;
}
```

### Bottom-up Dynamic Programming

Since we want to completely match `m` and `n` (the two interleaving strings) with `p`, we can use a two-dimensional array to store our results. The lengths of `m` and `n` will define the dimensions of the result array.

As mentioned above, we will be tracking separate indexes for `m`, `n` and `p`, so we will have the following options for every value of `mIndex`, `nIndex`, and `pIndex`:

1. If the character `m[mIndex]` matches the character `p[pIndex]`, we will take the matching result up to `mIndex-1` and `nIndex`.
2. If the character `n[nIndex]` matches the character `p[pIndex]`, we will take the matching result up to `mIndex` and `nIndex-1`.

String `p` will be interleaving strings `m` and `n` if any of the above two options is `true`. This is also required as there could be some common letters between `m` and `n`.

So our recursive formula would look like:

```cpp
dp[mIndex][nIndex] = false
if m[mIndex] == p[pIndex]
  dp[mIndex][nIndex] = dp[mIndex-1][nIndex]
if n[nIndex] == p[pIndex]
 dp[mIndex][nIndex] |= dp[mIndex][nIndex-1]
```

Here is the code for our <b>bottom-up dynamic programming</b> approach:

```cpp
function findSI(m, n, p) {
  // dp[mIndex][nIndex] will be storing the result of string leterleaving
  // up to p[0..mIndex+nIndex-1]
  dp = Array(m.length() + 1)
  // dp[mIndex][nIndex] will be storing the result of string interleaving
  // up to p[0..mIndex+nIndex-1]
  vector<vector<bool>> dp(m.length() + 1, vector<bool>(n.length() + 1, false));

  // make sure if lengths of the strings add up
  if (m.length() + n.length() != p.length()) return false;

  for (size_t mIndex = 0; mIndex <= m.length(); mIndex++) {
    for (size_t nIndex = 0; nIndex <= n.length(); nIndex++) {
      // if 'm' and 'n' are empty, then 'p' must have been empty too.
      if (mIndex == 0 && nIndex == 0) {
        dp[mIndex][nIndex] = true;
      }
      // if 'm' is empty, we need to check the interleaving with 'n' only
      else if (mIndex == 0 && n[nIndex - 1] == p[mIndex + nIndex - 1]) {
        dp[mIndex][nIndex] = dp[mIndex][nIndex - 1];
      }
      // if 'n' is empty, we need to check the interleaving with 'm' only
      else if (nIndex == 0 && m[mIndex - 1] == p[mIndex + nIndex - 1]) {
        dp[mIndex][nIndex] = dp[mIndex - 1][nIndex];
      } else {
        // if the letter of 'm' and 'p' match, we take whatever is matched till mIndex-1
        if (mIndex > 0 && m[mIndex - 1] == p[mIndex + nIndex - 1]) {
          dp[mIndex][nIndex] = dp[mIndex - 1][nIndex];
        }
        // if the letter of 'n' and 'p' match, we take whatever is matched till nIndex-1 too
        // note the '||', this is required when we have common letters
        if (nIndex > 0 && n[nIndex - 1] == p[mIndex + nIndex - 1]) {
          dp[mIndex][nIndex] = dp[mIndex][nIndex] || dp[mIndex][nIndex - 1];
        }
      }
    }
  }
  return dp[m.length()][n.length()];
}

int main() {
    cout << "String interleaving: ---> " << (findSI("abd", "cef", "abcdef") ? "true" : "false") << endl;
    // Output: true
    // Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.

    cout << "String interleaving: ---> " << (findSI("abd", "cef", "adcbef") ? "true" : "false") << endl;
    // Output: false
    // Explanation: 'p' contains all the letters from 'm' and 'n' but does not preserve the order.

    cout << "String interleaving: ---> " << (findSI("abc", "def", "abdccf") ? "true" : "false") << endl;
    // Output: false
    // Explanation: 'p' does not contain all the letters from 'm' and 'n'.

    cout << "String interleaving: ---> " << (findSI("abcdef", "mnop", "mnaobcdepf") ? "true" : "false") << endl;
    // Output: true
    // Explanation: 'p' contains all the letters from 'm' and 'n' and preserves their order too.

    return 0;
}
```

- The <b>time and space complexity</b> of the above algorithm is `O(m*n)`, where `m` and `n` are the lengths of the two interleaving strings.


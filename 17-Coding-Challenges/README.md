# General Coding Challenges

## 🎯 Challenge Categories

### String Manipulation
- [ ] **Two Pointers Technique**
- [ ] **Sliding Window Pattern**
- [ ] **String Building and Parsing**
- [ ] **Pattern Matching**

### Array and Matrix
- [ ] **Sorting and Searching**
- [ ] **Two Pointers and Three Pointers**
- [ ] **Matrix Traversal**
- [ ] **Prefix Sum Techniques**

### Mathematical Problems
- [ ] **Number Theory**
- [ ] **Bit Manipulation**
- [ ] **Mathematical Formulas**
- [ ] **Probability and Statistics**

## 🧩 String Manipulation Challenges

### 1. Valid Palindrome
**Difficulty**: Easy | **Time**: 15 minutes
```javascript
function isPalindrome(s) {
    const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, '');
    let left = 0, right = cleaned.length - 1;

    while (left < right) {
        if (cleaned[left] !== cleaned[right]) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

### 2. Longest Substring Without Repeating Characters
**Difficulty**: Medium | **Time**: 30 minutes
```javascript
function lengthOfLongestSubstring(s) {
    const seen = new Set();
    let left = 0, maxLength = 0;

    for (let right = 0; right < s.length; right++) {
        while (seen.has(s[right])) {
            seen.delete(s[left]);
            left++;
        }
        seen.add(s[right]);
        maxLength = Math.max(maxLength, right - left + 1);
    }

    return maxLength;
}
```

### 3. Group Anagrams
**Difficulty**: Medium | **Time**: 25 minutes
```javascript
function groupAnagrams(strs) {
    const map = new Map();

    for (const str of strs) {
        const sorted = str.split('').sort().join('');
        if (!map.has(sorted)) {
            map.set(sorted, []);
        }
        map.get(sorted).push(str);
    }

    return Array.from(map.values());
}
```

### 4. Minimum Window Substring
**Difficulty**: Hard | **Time**: 45 minutes
```javascript
function minWindow(s, t) {
    const need = new Map();
    const window = new Map();

    for (const char of t) {
        need.set(char, (need.get(char) || 0) + 1);
    }

    let left = 0, right = 0;
    let valid = 0;
    let start = 0, len = Infinity;

    while (right < s.length) {
        const c = s[right];
        right++;

        if (need.has(c)) {
            window.set(c, (window.get(c) || 0) + 1);
            if (window.get(c) === need.get(c)) {
                valid++;
            }
        }

        while (valid === need.size) {
            if (right - left < len) {
                start = left;
                len = right - left;
            }

            const d = s[left];
            left++;

            if (need.has(d)) {
                if (window.get(d) === need.get(d)) {
                    valid--;
                }
                window.set(d, window.get(d) - 1);
            }
        }
    }

    return len === Infinity ? "" : s.substring(start, start + len);
}
```

## 📊 Array and Matrix Challenges

### 5. Two Sum
**Difficulty**: Easy | **Time**: 15 minutes
```javascript
function twoSum(nums, target) {
    const map = new Map();

    for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];
        if (map.has(complement)) {
            return [map.get(complement), i];
        }
        map.set(nums[i], i);
    }

    return [];
}
```

### 6. Three Sum
**Difficulty**: Medium | **Time**: 30 minutes
```javascript
function threeSum(nums) {
    nums.sort((a, b) => a - b);
    const result = [];

    for (let i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] === nums[i - 1]) continue;

        let left = i + 1, right = nums.length - 1;

        while (left < right) {
            const sum = nums[i] + nums[left] + nums[right];

            if (sum === 0) {
                result.push([nums[i], nums[left], nums[right]]);

                while (left < right && nums[left] === nums[left + 1]) left++;
                while (left < right && nums[right] === nums[right - 1]) right--;

                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }

    return result;
}
```

### 7. Container With Most Water
**Difficulty**: Medium | **Time**: 20 minutes
```javascript
function maxArea(height) {
    let left = 0, right = height.length - 1;
    let maxWater = 0;

    while (left < right) {
        const water = Math.min(height[left], height[right]) * (right - left);
        maxWater = Math.max(maxWater, water);

        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }

    return maxWater;
}
```

### 8. Spiral Matrix
**Difficulty**: Medium | **Time**: 35 minutes
```javascript
function spiralOrder(matrix) {
    if (!matrix.length) return [];

    const result = [];
    let top = 0, bottom = matrix.length - 1;
    let left = 0, right = matrix[0].length - 1;

    while (top <= bottom && left <= right) {
        // Traverse right
        for (let col = left; col <= right; col++) {
            result.push(matrix[top][col]);
        }
        top++;

        // Traverse down
        for (let row = top; row <= bottom; row++) {
            result.push(matrix[row][right]);
        }
        right--;

        // Traverse left
        if (top <= bottom) {
            for (let col = right; col >= left; col--) {
                result.push(matrix[bottom][col]);
            }
            bottom--;
        }

        // Traverse up
        if (left <= right) {
            for (let row = bottom; row >= top; row--) {
                result.push(matrix[row][left]);
            }
            left++;
        }
    }

    return result;
}
```

## 🔢 Mathematical Problems

### 9. Reverse Integer
**Difficulty**: Easy | **Time**: 20 minutes
```javascript
function reverse(x) {
    const INT_MAX = 2**31 - 1;
    const INT_MIN = -(2**31);

    let result = 0;

    while (x !== 0) {
        const digit = x % 10;
        x = Math.trunc(x / 10);

        // Check for overflow before multiplication
        if (result > Math.trunc(INT_MAX / 10) ||
            (result === Math.trunc(INT_MAX / 10) && digit > 7)) {
            return 0;
        }
        if (result < Math.trunc(INT_MIN / 10) ||
            (result === Math.trunc(INT_MIN / 10) && digit < -8)) {
            return 0;
        }

        result = result * 10 + digit;
    }

    return result;
}
```

### 10. Power of Two
**Difficulty**: Easy | **Time**: 10 minutes
```javascript
function isPowerOfTwo(n) {
    if (n <= 0) return false;
    return (n & (n - 1)) === 0;
}

// Alternative mathematical approach
function isPowerOfTwoMath(n) {
    if (n <= 0) return false;
    return Math.log2(n) % 1 === 0;
}
```

### 11. Happy Number
**Difficulty**: Easy | **Time**: 20 minutes
```javascript
function isHappy(n) {
    const seen = new Set();

    while (n !== 1 && !seen.has(n)) {
        seen.add(n);
        n = getNext(n);
    }

    return n === 1;
}

function getNext(n) {
    let sum = 0;
    while (n > 0) {
        const digit = n % 10;
        sum += digit * digit;
        n = Math.floor(n / 10);
    }
    return sum;
}
```

### 12. Excel Sheet Column Number
**Difficulty**: Easy | **Time**: 15 minutes
```javascript
function titleToNumber(columnTitle) {
    let result = 0;

    for (let i = 0; i < columnTitle.length; i++) {
        const char = columnTitle[i];
        const value = char.charCodeAt(0) - 'A'.charCodeAt(0) + 1;
        result = result * 26 + value;
    }

    return result;
}
```

## 🎲 Bit Manipulation Challenges

### 13. Single Number
**Difficulty**: Easy | **Time**: 10 minutes
```javascript
function singleNumber(nums) {
    let result = 0;
    for (const num of nums) {
        result ^= num; // XOR operation
    }
    return result;
}
```

### 14. Number of 1 Bits
**Difficulty**: Easy | **Time**: 15 minutes
```javascript
function hammingWeight(n) {
    let count = 0;
    while (n !== 0) {
        count++;
        n &= (n - 1); // Remove rightmost 1 bit
    }
    return count;
}

// Alternative approach
function hammingWeightSimple(n) {
    return n.toString(2).split('1').length - 1;
}
```

### 15. Counting Bits
**Difficulty**: Easy | **Time**: 25 minutes
```javascript
function countBits(n) {
    const dp = new Array(n + 1).fill(0);

    for (let i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
    }

    return dp;
}
```

## 🧮 Dynamic Programming Foundations

### 16. Climbing Stairs
**Difficulty**: Easy | **Time**: 15 minutes
```javascript
function climbStairs(n) {
    if (n <= 2) return n;

    let prev2 = 1, prev1 = 2;

    for (let i = 3; i <= n; i++) {
        const current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }

    return prev1;
}
```

### 17. House Robber
**Difficulty**: Medium | **Time**: 20 minutes
```javascript
function rob(nums) {
    if (nums.length === 0) return 0;
    if (nums.length === 1) return nums[0];

    let prev2 = 0, prev1 = 0;

    for (const num of nums) {
        const temp = prev1;
        prev1 = Math.max(prev1, prev2 + num);
        prev2 = temp;
    }

    return prev1;
}
```

### 18. Maximum Subarray
**Difficulty**: Easy | **Time**: 20 minutes
```javascript
function maxSubArray(nums) {
    let maxSoFar = nums[0];
    let maxEndingHere = nums[0];

    for (let i = 1; i < nums.length; i++) {
        maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }

    return maxSoFar;
}
```

## 🔄 Recursion and Backtracking

### 19. Generate Parentheses
**Difficulty**: Medium | **Time**: 30 minutes
```javascript
function generateParenthesis(n) {
    const result = [];

    function backtrack(current, open, close) {
        if (current.length === 2 * n) {
            result.push(current);
            return;
        }

        if (open < n) {
            backtrack(current + '(', open + 1, close);
        }

        if (close < open) {
            backtrack(current + ')', open, close + 1);
        }
    }

    backtrack('', 0, 0);
    return result;
}
```

### 20. Letter Combinations of Phone Number
**Difficulty**: Medium | **Time**: 25 minutes
```javascript
function letterCombinations(digits) {
    if (!digits) return [];

    const phone = {
        '2': 'abc', '3': 'def', '4': 'ghi', '5': 'jkl',
        '6': 'mno', '7': 'pqrs', '8': 'tuv', '9': 'wxyz'
    };

    const result = [];

    function backtrack(index, path) {
        if (index === digits.length) {
            result.push(path);
            return;
        }

        const letters = phone[digits[index]];
        for (const letter of letters) {
            backtrack(index + 1, path + letter);
        }
    }

    backtrack(0, '');
    return result;
}
```

## 📈 Sorting and Searching

### 21. Merge Intervals
**Difficulty**: Medium | **Time**: 25 minutes
```javascript
function merge(intervals) {
    if (intervals.length <= 1) return intervals;

    intervals.sort((a, b) => a[0] - b[0]);
    const merged = [intervals[0]];

    for (let i = 1; i < intervals.length; i++) {
        const current = intervals[i];
        const lastMerged = merged[merged.length - 1];

        if (current[0] <= lastMerged[1]) {
            lastMerged[1] = Math.max(lastMerged[1], current[1]);
        } else {
            merged.push(current);
        }
    }

    return merged;
}
```

### 22. Search in Rotated Sorted Array
**Difficulty**: Medium | **Time**: 30 minutes
```javascript
function search(nums, target) {
    let left = 0, right = nums.length - 1;

    while (left <= right) {
        const mid = Math.floor((left + right) / 2);

        if (nums[mid] === target) {
            return mid;
        }

        if (nums[left] <= nums[mid]) {
            // Left half is sorted
            if (nums[left] <= target && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        } else {
            // Right half is sorted
            if (nums[mid] < target && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }

    return -1;
}
```

## 🎯 Problem-Solving Strategy

### Before Coding (5 minutes)
1. **Understand the problem completely**
2. **Identify input/output constraints**
3. **Consider edge cases**
4. **Choose appropriate data structures**
5. **Plan the algorithm approach**

### During Implementation
1. **Start with brute force if needed**
2. **Write clean, readable code**
3. **Handle edge cases explicitly**
4. **Test with examples as you go**
5. **Optimize time/space complexity**

### After Implementation
1. **Test with provided examples**
2. **Consider additional edge cases**
3. **Analyze time and space complexity**
4. **Look for optimization opportunities**
5. **Clean up and document code**

## 📚 Pattern Recognition

### Common Patterns
- **Two Pointers**: Palindromes, sorted arrays
- **Sliding Window**: Substring problems
- **Fast & Slow Pointers**: Cycle detection
- **Merge Intervals**: Overlapping intervals
- **Cyclic Sort**: Missing numbers
- **In-place Reversal**: Linked list reversal
- **Tree DFS**: Tree traversal problems
- **Tree BFS**: Level-order traversal
- **Two Heaps**: Median finding
- **Subsets**: Permutations and combinations
- **Modified Binary Search**: Sorted arrays with twists
- **Top K Elements**: Heap-based problems
- **K-way Merge**: Merge sorted arrays
- **Dynamic Programming**: Optimization problems

## ✅ Progress Tracker

### String Challenges
- [ ] Valid Palindrome (Easy)
- [ ] Longest Substring Without Repeating (Medium)
- [ ] Group Anagrams (Medium)
- [ ] Minimum Window Substring (Hard)

### Array Challenges
- [ ] Two Sum (Easy)
- [ ] Three Sum (Medium)
- [ ] Container With Most Water (Medium)
- [ ] Spiral Matrix (Medium)

### Mathematical Problems
- [ ] Reverse Integer (Easy)
- [ ] Power of Two (Easy)
- [ ] Happy Number (Easy)
- [ ] Excel Sheet Column Number (Easy)

### Bit Manipulation
- [ ] Single Number (Easy)
- [ ] Number of 1 Bits (Easy)
- [ ] Counting Bits (Easy)

### Dynamic Programming
- [ ] Climbing Stairs (Easy)
- [ ] House Robber (Medium)
- [ ] Maximum Subarray (Easy)

### Recursion & Backtracking
- [ ] Generate Parentheses (Medium)
- [ ] Letter Combinations (Medium)

### Sorting & Searching
- [ ] Merge Intervals (Medium)
- [ ] Search in Rotated Array (Medium)

**Target**: Solve 50+ coding challenges across various categories with pattern recognition and optimization skills
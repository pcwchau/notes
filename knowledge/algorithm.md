# Index
- [Index](#index)
- [Notes](#notes)
- [Questions](#questions)
  - [Divide and Conquer](#divide-and-conquer)
  - [Binary Search](#binary-search)
    - [Search Insert Position](#search-insert-position)
    - [Search a 2D Matrix](#search-a-2d-matrix)
  - [Dynamic Programming](#dynamic-programming)
    - [Jump Game](#jump-game)
  - [Greedy Algorithm](#greedy-algorithm)
    - [Gas Station](#gas-station)
    - [Jump Game](#jump-game-1)
  - [Arrays](#arrays)
    - [Best Time to Buy and Sell Stock](#best-time-to-buy-and-sell-stock)
  - [Matrix](#matrix)
    - [Sprial Matrix](#sprial-matrix)
    - [Printing number in spiral pattern](#printing-number-in-spiral-pattern)
  - [Two Pointers (Fast and Slow)](#two-pointers-fast-and-slow)
    - [Remove Nth Node From End of List](#remove-nth-node-from-end-of-list)
  - [Two Pointers (Left and Right)](#two-pointers-left-and-right)
    - [Two Sum II - Input Array Is Sorted](#two-sum-ii---input-array-is-sorted)
    - [Three Sum](#three-sum)
    - [Container With Most Water](#container-with-most-water)
  - [Sliding Window](#sliding-window)
    - [Longest Substring Without Repeating Characters](#longest-substring-without-repeating-characters)
    - [Maximum consecutive occurrences of a string in another given string](#maximum-consecutive-occurrences-of-a-string-in-another-given-string)

# Notes

- Remember to update variable inside loop.

# Questions

## Divide and Conquer

Solve a problem of size `n` by breaking it into one or more smaller problems of size less than `n`.

## Binary Search

Recursion version:

```java
public static int binarySearch(int[] nums, int target, int from, int to) {
    if (from > to) return -1;
    int middle = (from + to) / 2;

    if (nums[middle] == target)
        return middle;
    else if (nums[middle] < target)
        return binarySearch(nums, target, middle + 1, to);
    else
        return binarySearch(nums, target, from, middle - 1);
}
```

While-loop version:
```java
public static int binarySearch(int[] nums, int target) {
    int from = 0;
    int to = nums.length - 1;

    while (from <= to) {
        int middle = (from + to) / 2;
        if (nums[middle] == target)
            return middle;
        else if (nums[middle] < target)
            from = middle + 1;
        else
            to = middle - 1;
    }
    return -1;
}
```

**Analysis:**

```
Target = 4, First call = binarySearch(arr, 4, 0, 9)

 0  1  2   3   4   5   6   7   8   9
[4, 7, 10, 15, 19, 20, 42, 54, 87, 90]  Mid = (0 + 9) / 2 = 5
                   M                    4 < 20, so call binarySearch(arr, 4, 0, 4)
 0  1  2   3   4
[4, 7, 10, 15, 19]                      Mid = (0 + 4) / 2 = 2
       M                                4 < 10, so call binarySearch(arr, 4, 0, 1)
 0  1
[4, 7]                                  Mid = (0 + 1) / 2 = 0
 M                                      4 == 4, found


Target = 7, First call = binarySearch(arr, 7, 0, 9)

 0  1  2   3   4   5   6   7   8   9
[4, 7, 10, 15, 19, 20, 42, 54, 87, 90]  Mid = (0 + 9) / 2 = 5
                   M                    7 < 20, so call binarySearch(arr, 7, 0, 4)
 0  1  2   3   4
[4, 7, 10, 15, 19]                      Mid = (0 + 4) / 2 = 2
       M                                7 < 10, so call binarySearch(arr, 7, 0, 1)
 0  1
[4, 7]                                  Mid = (0 + 1) / 2 = 0
 M                                      7 > 4, so call binarySearch(arr, 7, 1, 1)
    1
   [7]                                  Mid = (1 + 1) / 2 = 1
    M                                   7 == 4, found


Target = 8, First call = binarySearch(arr, 8, 0, 9)

 0  1  2   3   4   5   6   7   8   9
[4, 7, 10, 15, 19, 20, 42, 54, 87, 90]  Mid = (0 + 9) / 2 = 5
                   M                    8 < 20, so call binarySearch(arr, 8, 0, 4)
 0  1  2   3   4
[4, 7, 10, 15, 19]                      Mid = (0 + 4) / 2 = 2
       M                                8 < 10, so call binarySearch(arr, 8, 0, 1)
 0  1
[4, 7]                                  Mid = (0 + 1) / 2 = 0
 M                                      8 > 4, so call binarySearch(arr, 8, 1, 1)
    1
   [7]                                  Mid = (1 + 1) / 2 = 1
    M                                   8 > 7, so call binarySearch(arr, 8, 2, 1)

                                        Since 2 > 1, so 8 cannot be found

Target = 6, First call = binarySearch(arr, 6, 0, 9)

    1
   [7]                                  Mid = (1 + 1) / 2 = 1
    M                                   6 < 7, so call binarySearch(arr, 6, 1, 0)

                                        Since 1 > 0, so 6 cannot be found
```

### Search Insert Position

- https://leetcode.com/problems/search-insert-position/description/

If a number is found, `from` is its index. If a number is not found, `from` is its index if it were inserted in order.

### Search a 2D Matrix

- https://leetcode.com/problems/search-a-2d-matrix/description/

In most cases of the row binary search, the target number is not the first number of any rows.

- If the target is larger, then it is the same row.
- If the target is smaller, then it may be the upper row, which can be an edge case when there is no more upper row.

## Dynamic Programming

Solve all smaller size problems => build larger problem solutions from them.

DP often used for optimization problems

### Jump Game

- https://leetcode.com/problems/jump-game/description/

**Analysis:**

```
Example 1:
Input: nums = [2, 2, 0, 1, 4]; so last index is 4.

Calculate maximum index that you can jump to from each element
    canJumpTo = [2, 3, 2, 4, 8]

max index from range 0 to 0 = 2
max index from range 1 to 2 = 3
max index from range 3 to 3 = 4 (return true, already >= last index)

Example 2:
Input: nums = [3, 2, 1, 0, 4]; so last index is 4.

Calculate maximum index that you can jump to from each element
    canJumpTo = [3, 3, 3, 3, 4]

max index from range 0 to 0 = 3
max index from range 1 to 3 = 3
max index from range 4 to 3 (return false, it is impossible to go to index 4)
```

## Greedy Algorithm

A greedy algorithm always makes the choice that looks best at the moment and adds it to the current partial solution.

### Gas Station

- https://leetcode.com/problems/gas-station/description/

**Analysis:**

Given `gas[]` and `cost[]`, calculate `realCost[i] = gas[i] - cost[i]`.

Initialize: `start = 0`, `end = 0`, `currentGas = 0`

We have `n` stations in the circular route. We start at station 0.

Run a loop for `n - 1` times:

- If there is not enough gas to travel to the next station, i.e. `currentGas <= 0`, move the starting point `start` to the previous station.
- If there is enough gas to travel to the next station, i.e. `currentGas > 0`, move the ending point `end` to the next station.
- Calculate current gas after moving either the starting point or the ending point: `currentGas += realCost[i]`.

Why it is correct?

If you start from station `A` and stuck at `B`, then you can't get to `B` from any station between `A` and `B`. So the only way is to check the station before `A`.

### Jump Game

- https://leetcode.com/problems/jump-game/description/

**Analysis:**

```
Example 1:
Input: nums = [2, 2, 0, 1, 4]; so last index is 4.

Calculate maximum index that you can jump to from each element
    canJumpTo = [2, 3, 2, 4, 8]

max index from index 0 = 2, current max = 2
max index from index 1 = 3, current max = 3
max index from index 2 = 2, current max = 3
max index from index 3 = 4, current max = 4 (return true, already >= last index)

Example 2:
Input: nums = [3, 2, 1, 0, 4]; so last index is 4.

Calculate maximum index that you can jump to from each element
    canJumpTo = [3, 3, 3, 3, 4]

max index from index 0 = 3, current max = 3
max index from index 1 = 3, current max = 3
max index from index 2 = 3, current max = 3
max index from index 3 = 3, current max = 3
max index from index 4 ... (return false, it is impossible to go to index 4)
```

## Arrays

### Best Time to Buy and Sell Stock

- https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/
- O(n), avoid using O(n^2) algorithm.

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i th` day.

You want to maximize your profit by choosing a single day to buy one stock and choosing a different day in the future to sell that stock.

*Return the maximum profit you can achieve from this transaction. If you cannot achieve any profit, return 0.*

```
Input: prices = [7,2,5,3,6,4,1,7]
Output: 6
Explanation: Buy on day 7 (price = 1) and sell on day 8 (price = 7), profit = 7-1 = 6.
```

**Analysis:**

When iterating the price array, you have to keep track of the minimum price seen so far to calculate the maximum profit. For example, `[2,3,1,6]`, you will take `1` instead of `2`.

However, the minimum price does not necessarily contribute to the maximum profit. For example, `[2,6,1,3]`, you will still take "`2` buy `6` sell" even `1` is the minumum price. Therefore, you have to store the largest profit that can be achieved already.

**Solution:**

```java
public int maxProfit(int[] prices) {
    int maxProfit = 0;
    int minPrice = prices[0];

    for (int i = 1; i < prices.length; i++)
    {
        int todayPrice = prices[i];
        if (todayPrice < minPrice)
        {
            minPrice = todayPrice;
        }
        else
        {
            int profit = todayPrice - minPrice;
            if (profit > maxProfit)
            {
                maxProfit = profit;
            }
        }
    }
    return maxProfit;
}
```

## Matrix

To simulate a spiral order, we need variables to save:

- Current row and column
- Current direction
- The four boundaries (since we are moving inwards and inwards)

### Sprial Matrix

Given an `m x n` `matrix`, return all elements of the `matrix` in spiral order.

```
1 2 3
4 5 6
7 8 9

return:
1 2 3 6 9 8 7 4 5
```

**Solution:**

```java
public static List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> result = new ArrayList<>();

    int rowNum = matrix.length;
    int colNum = matrix[0].length;

    int currentRow = 0;
    int currentCol = -1; // after first location update, will become 0

    // next location cannot exceed the boundary
    int upBoundary = 1; // starting from row 0, so row 0 cannot be inputted again
    int downBoundary = rowNum - 1;
    int leftBoundary = 0;
    int rightBoundary = colNum - 1;

    int direction = 0; // 0 right, 1 down, 2 left, 3 up

    for (int i = 0; i < rowNum * colNum; i++) {
        // update the location of next input number based on the direction
        if (direction == 0)
            currentCol++; // right
        else if (direction == 1)
            currentRow++; // down
        else if (direction == 2)
            currentCol--; // left
        else if (direction == 3)
            currentRow--; // up

        // input the number
        result.add(matrix[currentRow][currentCol]);

        // test next location valid or not, change direction if not
        if (direction == 0 && currentCol + 1 > rightBoundary) {
            direction = 1;
            rightBoundary--;
        } else if (direction == 1 && currentRow + 1 > downBoundary) {
            direction = 2;
            downBoundary--;
        } else if (direction == 2 && currentCol - 1 < leftBoundary) {
            direction = 3;
            leftBoundary++;
        } else if (direction == 3 && currentRow - 1 < upBoundary) {
            direction = 0;
            upBoundary++;
        }
    }
    return result;
}
```

### Printing number in spiral pattern

Examples:
```
Input: 3
Output:
2 1 8
3 0 7
4 5 6

Input: 5
Output:
12 11 10 9 24 
13 2 1 8 23 
14 3 0 7 22 
15 4 5 6 21 
16 17 18 19 20 
```

Approach:
- Initialize a 2D int array, put 0 in the centre.
- Note that the numbers are printed as the sequence: up, left, down, right
- Given input number `N`, there will be `N - 1` loop
  - 1st loop, print 1 number up, then 1 number left
  - 2nd loop, print 2 numbers down, then 2 numbers right
  - 3rd loop, print 3 numbers up, then 3 numbers left
  - ...
  - `N - 1` loop, print `N - 1` numbers down, then `N - 1` numbers right, then `N - 1` numbers up

## Two Pointers (Fast and Slow)

This technique uses two pointers that traverse the linked list at different speeds or in different directions.

**Finding the Middle of a Linked List:**

- Initialize both pointers to the head of the linked list.
- Move the fast pointer two nodes at a time, while moving the slow pointer one node at a time.
- When the fast pointer reaches the end of the linked list, the slow pointer will be at the middle node.

**Finding a Loop in a Linked List:**

- Initialize both pointers to the head of the linked list.
- Move the fast pointer two nodes at a time, while moving the slow pointer one node at a time.
- If the fast pointer ever catches up to the slow pointer, there is a loop in the linked list.

**Finding the nth Node from the End of a Linked List:**

- Initialize two pointers, `p` and `q`, to the head of the linked list.
- Advance the pointer `q`, n nodes ahead of the pointer `p`.
- Now, advance both pointers `p` and `q` one node at a time.
- When the pointer `q` reaches the end of the linked list, the pointer `p` will be at the nth node from the end.

### Remove Nth Node From End of List

- https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/

**Analysis:**

Basically it is finding the N-1 th node from the end of the linked list.

Be aware of the following edge cases

- N = 1 (removing the last node)
- N = number of nodes (removing the first node), and if there is only one node, you need to return `null`.

## Two Pointers (Left and Right)

This technique uses two pointers - **left** and **right** - to traverse an array from two corners.

### Two Sum II - Input Array Is Sorted

- https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/

Given a 1-indexed array of integers numbers that is already sorted in non-decreasing order, find two numbers such that they add up to a specific target number.

```
Input: numbers = [2,7,11,15], target = 13
Output: [0,2]
Explanation: The sum of 2 and 11 is 13. They are in index 0 and 2. We return [0, 2].
```

**Analysis:**

Initialize: `left = 0`, `right = n – 1`.

Run a loop while `left` < `right`, do the following inside the loop:

- Compute current sum = `numbers[left]` + `numbers[right]`
- If the sum equals the target, we’ve found the pair.
- If the sum is less than the target, move the **left** pointer to the right to increase the sum.
- If the sum is greater than the target, move the **right** pointer to the left to decrease the sum.

Why can we move one of the pointers by judging the sum?

Because the given array is sorted in *increasing order*. If we move a pointer to the left, the new number must be smaller. If we move a pointer to the right, the new number must be larger.

**Solution:**

```java
public int[] twoSum(int[] numbers, int target) {
    int left = 0;
    int right = numbers.length - 1;
    while (left < right) {
        int sum = numbers[left] + numbers[right];
        if (sum == target) break;
        else if (sum > target) right--;
        else left++;
    }
    int[] result = {left, right};
    return result;
}
```

### Three Sum

- https://leetcode.com/problems/3sum/description/

Given an integer array `nums`, return all the triplets `[nums[i], nums[j], nums[k]]` such that `i != j`, `i != k`, and `j != k`, and `nums[i] + nums[j] + nums[k] == 0`.

Notice that the solution set must not contain duplicate triplets.

**Analysis:**

Run a loop for `i` from `0` to `n - 1`, do the following inside the loop:

- Find two numbers from `nums[i+1]` to `nums[n - 1]` such that their sum equals to `-nums[i]`. So basically it is a two sum problem.

### Container With Most Water

- https://leetcode.com/problems/container-with-most-water/description/

**Analysis:**

Initialize: `left = 0`, `right = n – 1`.

Run a loop while `left` < `right`, do the following inside the loop:

- Compute current area of water = width * maxWaterLevel = (`right` - `left`) * min(`height[left]`, `height[right]`)
- If the height of the left pointer is lower, move the left pointer to the right.
- If the height of the right pointer is lower, move the right pointer to the left.
- If the height of both pointers are the same, move either one.

Why can we move one of the pointers by judging which one is lower in height?

After moving the lower pointer closer to another, the width decreases. Therefore, there cannot be a bigger area in between the two points that contains the lower pointer's point. This fact allows us to ignore any other combination with the lower point.

## Sliding Window

Sliding Window problems are problems in which a fixed or variable-size window is moved through a data structure, typically an array or string, to solve problems efficiently based on continuous subsets of elements.

**This technique is used when we need to find subarrays or substrings according to a given set of conditions.**

In a sliding window, we need to maintain **one left pointer** and **one right pointer** to control the window size and ensure the window is under the condition.

### Longest Substring Without Repeating Characters

- https://leetcode.com/problems/longest-substring-without-repeating-characters/description/

Given a string `s`, find the length of the longest substring  without repeating characters.

```
Input: s = "abcaabcd"
Output: 4
Explanation: The answer is "abcd", with the length of 4.
```

**Analysis:**

We need to find the length of the longest substring while looping the input string `s`, so we need a variable to store the longest length that is achieved already.

We need to use the sliding window technique, so we need variables to store the pointers.

The condition of the sliding window is that there is no repeated character within the sliding window. When we see a repeated character, we need to shrink the window.

```
                                           Shrink
abcaabcd    abcaabcd    abcaabcd    abcaabcd    abcaabcd
|        -> ||       -> |-|      -> |--|     ->  |-|    

                   Shrink      Shrink      Shrink
abcaabcd    abcaabcd    abcaabcd    abcaabcd    abcaabcd
 |-|     ->  |--|    ->   |-|    ->    ||    ->     |


abcaabcd    abcaabcd    abcaabcd    abcaabcd
    |    ->     ||   ->     |-|  ->     |--| -> Finish
```

To check if there is a repeated character, we can use a hash table to store the characters in the current substring. Here, we will use `HashMap` instead of `HashSet`. In the `HashMap`, the key is the character and the value is the position. When we see a repeated character, we can immediately move the left pointer to shrink the window.

**Solution:**

```java
public int lengthOfLongestSubstring(String s) {
    if (s.length() == 0) return 0;
    else if (s.length() == 1) return 1;
    else {
        Map<Character, Integer> map = new HashMap<>();
        int leftPointer = 0;
        int rightPointer = 0;
        int longest = 0;

        for (int i = 0; i < s.length(); i++) {
            rightPointer = i;
            char currentChar = s.charAt(i);

            if (!map.containsKey(currentChar) || 
                map.get(currentChar) < leftPointer) {

                map.put(currentChar, i);

                int length = rightPointer - leftPointer + 1;
                if (length > longest) longest = length;
            } else {
                leftPointer = map.get(currentChar) + 1;
                map.put(currentChar, i);
            }
        }
        return longest;
    }
}
```

### Maximum consecutive occurrences of a string in another given string

Given two strings str1 and str2, the task is to count the maximum consecutive occurrences of the string str2 in the string str1.

Examples:
```
Input: str1 = “abababcba”, str2 = “ba” 
Output: 2 
Explanation: String str2 occurs consecutively in the substring { str[1], …, str[4] }. Therefore, the maximum count obtained is 2

Input: str1 = “ababc”, str2 = “ac” 
Output: 0 
Explanation: 
Since str2 is not present as a substring in str1, the required output is 0.
```
Approach:
- Initialize a variable, `CntOcc`, to store the count of the occurrences of `str2` in `str1`.
  - 3 occurrences of `ba` in `abababcba`
- Iterate over the range `[CntOcc, 1]`. For every i th value in the iteration, concatenate the string `str2` i times and check if the concatenated string is a substring of the string `str1` or not. The first ith value for which it is found to be true, print it as the required answer.
  - Does `abababcba` contain 3 consecutive `ba`? -> No
  - Does `abababcba` contain 2 consecutive `ba`? -> Yes, return 2

Time complexity: O(MN), M and N are length of `str1` and `str2`
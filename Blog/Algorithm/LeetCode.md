## sieve of Eratosthenes 素数筛选法
 

[素数与合数](https://zhuanlan.zhihu.com/p/143521872)

## sieve of Eratosthenes 素数筛选法是什么？

sieve of Eratosthenes 素数筛选法是一种找出指定范围内所有素数的方法。

现在给出一个问题：如何找到 [0, n] 之间的所有素数？

当然是可以一个个去试的，但是这样是很低效的，因为每个都需要去挨个挨个的整除。

基本的思想是这样的，如果我知道一个数 x 是素数，那么它的倍数一定是合数，也就是 2x, 3x, 4x, ... 都是合数。

举例说明：2 是素数，它的倍数们：4、6、8、10 等等，都是合数，这是显而易见的。

现在可以用伪代码来描述一下这个过程。

```
input: n; // 找出 [0, n] 之间所有的素数

  

bool[] mem = new bool[n + 1]; // 初始化一个长度为 n + 1 的数组

fillAll(mem, true); // 将所有的值填充为 true，认为所有的数都是素数

count = 0;

for (i = 2 to sqrt(n)) { // 从 2 开始迭代到 sqrt(n)

if (mem[i]) {

count++; // 发现素数，count 自增

  

// 并将后面的倍数都标记为 false

for (j = i * i; j < n + 1; j += i) {

mem[i] = false;

}

}

}

output: count; // count 就是结果
```

## TOP-K问题几种解法


## Boyer-Moore Voting Algorithm

Intuition

If we had some way of counting instances of the majority element as +1+1 and instances of any other element as -1−1, summing them would make it obvious that the majority element is indeed the majority element.

Algorithm

Essentially, what Boyer-Moore does is look for a suffix sufsuf of nums where suf[0]suf[0] is the majority element in that suffix. To do this, we maintain a count, which is incremented whenever we see an instance of our current candidate for majority element and decremented whenever we see anything else. Whenever count equals 0, we effectively forget about everything in nums up to the current index and consider the current number as the candidate for majority element. It is not immediately obvious why we can get away with forgetting prefixes of nums - consider the following examples (pipes are inserted to separate runs of nonzero count).

[7, 7, 5, 7, 5, 1 | 5, 7 | 5, 5, 7, 7 | 7, 7, 7, 7]

Here, the 7 at index 0 is selected to be the first candidate for majority element. count will eventually reach 0 after index 5 is processed, so the 5 at index 6 will be the next candidate. In this case, 7 is the true majority element, so by disregarding this prefix, we are ignoring an equal number of majority and minority elements - therefore, 7 will still be the majority element in the suffix formed by throwing away the first prefix.

[7, 7, 5, 7, 5, 1 | 5, 7 | 5, 5, 7, 7 | 5, 5, 5, 5]

Now, the majority element is 5 (we changed the last run of the array from 7s to 5s), but our first candidate is still 7. In this case, our candidate is not the true majority element, but we still cannot discard more majority elements than minority elements (this would imply that count could reach -1 before we reassign candidate, which is obviously false).

Therefore, given that it is impossible (in both cases) to discard more majority elements than minority elements, we are safe in discarding the prefix and attempting to recursively solve the majority element problem for the suffix. Eventually, a suffix will be found for which count does not hit 0, and the majority element of that suffix will necessarily be the same as the majority element of the overall array.

  

Complexity Analysis

-   Time complexity : O(n)O(n)
    
    Boyer-Moore performs constant work exactly nn times, so the algorithm runs in linear time.
    
-   Space complexity : O(1)O(1)
    
    Boyer-Moore allocates only constant additional memory.
    

示例：

169. Majority Element

Easy

Given an array nums of size n, return the majority element.

The majority element is the element that appears more than ⌊n / 2⌋ times. You may assume that the majority element always exists in the array.

Example 1:

Input: nums = [3,2,3]

Output: 3

Example 2:

Input: nums = [2,2,1,1,1,2,2]
Output: 2
Constraints:
-   n == nums.length 
-   1 <= n <= 5 * 104
-   -231 <= nums[i] <= 231 - 1
    
Follow-up: Could you solve the problem in linear time and in O(1) space?

```
class Solution {
    public int majorityElement(int[] nums) {
        int major =0;
        int count = 0;
        for (int num : nums) {
            if (count==0){
                major=num;
            }
            count+=major==num?1:-1;
        }
        return major;
    }
}
```
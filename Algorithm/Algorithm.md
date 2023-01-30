## C1 搜索算法
### 第一部分 搜索算法

搜索算法是利用计算机的高性能来有目的的穷举一个问题的部分或所有的可能情况，从而求出问题的解的一种方法。搜索过程实际上是根据初始条件和扩展规则构造一棵解答树并寻找符合目标状态的节点的过程。

所有的搜索算法从其最终的算法实现上来看，都可以划分成两个部分──控制结构和产生系统，而所有的算法的优化和改进主要都是通过修改其控制结构来完成的。第一部分　基本搜索算法

#### 一、回溯算法（backtracking）

回溯算法是所有搜索算法中最为基本的一种算法，其采用了一种“走不通就掉头”思想作为其控制结构，其相当于采用了先根遍历的方法来构造解答树，可用于找解或所有解以及最优解。

评价：回溯算法对空间的消耗较少，当其与分枝定界法一起使用时，对于所求解在解答树中层次较深的问题有较好的效果。但应避免在后继节点可能与前继节点相同的问题中使用，以免产生循环。

二、深度搜索（dfs）与广度搜索（bfs）

深度搜索与广度搜索的控制结构和产生系统很相似，唯一的区别在于对扩展节点选取上。由于其保留了所有的前继节点，所以在产生后继节点时可以去掉一部分重复的节点，从而提高了搜索效率。
这两种算法每次都扩展一个节点的所有子节点，而不同的是，深度搜索下一次扩展的是本次扩展出来的子节点中的一个，而广度搜索扩展的则是本次扩展的节点的兄弟节点。
评价：广度搜索是求解最优解的一种较好的方法，在后面将会对其进行进一步的优化。而深度搜索多用于只要求解，并且解答树中的重复节点较多并且重复较难判断时使用，但往往可以用A*或回溯算法代替。

三、 示例

  

78. Subsets（backtracking）

[https://leetcode.com/problems/subsets/](https://leetcode.com/problems/subsets/)

  

Given an integer array nums of unique elements, return all possible subsets (the power set).

The solution set must not contain duplicate subsets. Return the solution in any order.

Example 1:

Input: nums = [1,2,3]

Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

Constraints:

-   1 <= nums.length <= 10
    
-   -10 <= nums[i] <= 10
    
-   All the numbers of nums are unique.
    
```
class Solution {

    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> lists = new ArrayList<>();
        dfs(nums,new ArrayList<Integer>(),0,0,lists);
        return lists;
    }
    private void dfs(int[] nums ,List<Integer> list,int start, int size,List<List<Integer>> lists){

        lists.add(new ArrayList<>(list));
        for (int i = start; i < nums.length; i++) {
            list.add(nums[i]);
            dfs(nums,list,i+1,size+1,lists);
            list.remove(list.size()-1);
        }
    }
}
```


  

79. Word Search（backtracking）

[https://leetcode.com/problems/word-search/](https://leetcode.com/problems/word-search/)

Medium

Given an m x n grid of characters board and a string word, return true if word exists in the grid.
The word can be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighboring. The same letter cell may not be used more than once.

Example 1:
Output: true
Constraints:
-   m == board.length
-   n = board[i].length
-   1 <= m, n <= 6
-   1 <= word.length <= 15
-   board and word consists of only lowercase and uppercase English letters.

Follow up: Could you use search pruning to make your solution faster with a larger board?

class Solution {

    public boolean exist(char[][] board, String word) {
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                boolean ret = dfs(board, word, 0, i, j);
                if (ret) return true;
            }
        }
        return false;
    }

    private boolean dfs(char[][] board, String word, int wIndex, int i, int j) {

        if (wIndex == word.length()) return true;
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length) return false;
        if (board[i][j] == word.charAt(wIndex)) {
            char tmp = board[i][j];
            board[i][j] = '#';
            boolean ret = dfs(board, word, wIndex + 1, i + 1, j)
                    || dfs(board, word, wIndex + 1, i, j + 1)
                    || dfs(board, word, wIndex + 1, i - 1, j)
                    || dfs(board, word, wIndex + 1, i, j - 1);
            board[i][j] = tmp;
            return ret;
        } else return false;
    }
}


### 第二部分　搜索算法的优化

#### 一、双向广度搜索

广度搜索虽然可以得到最优解，但是其空间消耗增长太快。但如果从正反两个方向进行广度搜索，理想情况下可以减少二分之一的搜索量，从而提高搜索速度。

范例：有N个黑白棋子排成一派，中间任意两个位置有两个连续的空格。每次空格可以与序列中的某两个棋子交换位置，且两子的次序不变。要求出入长度为length的一个初始状态和一个目标状态，求出最少的转化步数。
问题分析：该题要求求出最少的转化步数，但如果直接使用广度搜索，很容易产生数据溢出。但如果从初始状态和目标状态两个方向同时进行扩展，如果两棵解答树在某个节点第一次发生重合，则该节点所连接的两条路径所拼成的路径就是最优解。

对广度搜索算法的改进：
1、添加一张节点表，作为反向扩展表。
2、在while循环体中在正向扩展代码后加入反向扩展代码，其扩展过程不能与正向过程共享一个for循环。
3、在正向扩展出一个节点后，需在反向表中查找是否有重合节点。反向扩展时与之相同。

对双向广度搜索算法的改进：
略微修改一下控制结构，每次while循环时只扩展正反两个方向中节点数目较少的一个，可以使两边的发展速度保持一定的平衡，从而减少总扩展节点的个数，加快搜索速度。

#### 二、分支定界

分支定界实际上是A*算法的一种雏形，其对于每个扩展出来的节点给出一个预期值，如果这个预期值不如当前已经搜索出来的结果好的话，则将这个节点(包括其子节点)从解答树中删去，从而达到加快搜索速度的目的。
范例：在一个商店中购物，设第I种商品的价格为Ci。但商店提供一种折扣，即给出一组商品的组合，如果一次性购买了这一组商品，则可以享受较优惠的价格。现在给出一张购买清单和商店所提供的折扣清单，要求利用这些折扣，使所付款最少。

问题分析：显然，折扣使用的顺序与最终结果无关，所以可以先将所有的折扣按折扣率从大到小排序，然后采用回溯法的控制结构，对每个折扣从其最大可能使用次数向零递减搜索，设A为以打完折扣后优惠的价格，C为当前未打折扣的商品零售价之和，则其预期值为A+a*C，其中a为下一个折扣的折扣率。如当前已是最后一个折扣，则a=1。

对回溯算法的改进：
1、添加一个全局变量BestAnswer，记录当前最优解。
2、在每次生成一个节点时，计算其预期值，并与BestAnswer比较。如果不好，则调用回溯过程。

#### 三、A*算法

 A*算法中更一般的引入了一个估价函数f,其定义为f=g+h。其中g为到达当前节点的耗费，而h表示对从当前节点到达目标节点的耗费的估计。其必须满足两个条件：
1、h必须小于等于实际的从当前节点到达目标节点的最小耗费h*。
2、f必须保持单调递增。
A*算法的控制结构与广度搜索的十分类似，只是每次扩展的都是当前待扩展节点中f值最小的一个，如果扩展出来的节点与已扩展的节点重复，则删去这个节点。如果与待扩展节点重复，如果这个节点的估价函数值较小，则用其代替原待扩展节点，具体算法描述如下：

范例：一个3*3的棋盘中有1-8八个数字和一个空格，现给出一个初始态和一个目标态，要求利用这个空格，用最少的步数，使其到达目标态。
问题分析：预期值定义为h=|x-dx|+|y-dy|。
              　估价函数定义为f=g+h。
对A*算法的改进--分阶段A*：
当A*算法出现数据溢出时，从待扩展节点中取出若干个估价函数值较小的节点，然后放弃其余的待扩展节点，从而可以使搜索进一步的进行下去。

### 第三部分　搜索与动态规划的结合


## C2 动态规划Dynamic Programming
动态规划算法 Dynamic Programming

### 第1部分 动态规划的基本概念

多阶段决策问题

    多阶段决策过程，是指这样的一类特殊的活动过程，问题可以按时间顺序分解成若干相互联系的阶段，在每一个阶段都要做出决策，全部过程的决策是一个决策序列。要使整个过程达到最优的问题，称为多阶段决策问题。

动态规划的基本要素

一个多阶段决策过程最优化问题的动态规划通常包含以下要素：

1.阶段：阶段(step)是对整个过程的自然划分。通常根据时间顺序或空间特征来划分阶段，以便按阶段的次序解优化问题。

2.状态：状态(state)表示每个阶段开始时过程所处的自然状况。它应该能够描述过程的特征并且具有无后向性，即当某阶段的状态给定时，这个阶段以后过程的演变与该阶段以前各阶段的状态无关，即每个状态都是过去历史的一个完整总结。通常还要求状态是直接或间接可以观测的。

3.决策/策略：当一个阶段的状态确定后，可以作出各种选择从而演变到下一阶段的某个状态，这种选择手段称为决策(decision)。

4.状态转移方程：在确定性过程中，一旦某阶段的状态和决策为已知，下阶段的状态便完全确定。用状态转移方程(equation of state)表示这种演变规律

     dp[i] = f(dp[i-1])+f(dp[i-2])

     dp[i] = f(dp[i-1])

5.指标函数和最优值函数：指标函数(objective function)是衡量过程优劣的数量指标，它是关于策略的数量函数。能够用动态规划解决的问题的指标函数应具有可分离性，其中函数是一个关于变量单调递增的函数。这一性质保证了最优化原理(principle of optimality)的成立，是动态规划的适用前提。

### 第2部分 动态规划的适用条件

任何思想方法都有一定的局限性，超出了特定条件，它就失去了作用。同样，动态规划也并不是万能的。适用动态规划的问题必须满足最优化原理和无后效性。

1.最优化原理（最优子结构性质）

    最优化原理可这样阐述：一个最优化策略具有这样的性质，不论过去状态和决策如何，对前面的决策所形成的状态而言，余下的诸决策必须构成最优策略。简而言之，一个最优化策略的子策略总是最优的。一个问题满足最优化原理又称其具有最优子结构性质。最优化原理是动态规划的适用前提，动态规划的最优化理在其指标函数的可分离性和单调性中得到体现。

2.无后向性

    将各阶段按照一定的次序排列好之后，对于某个给定的阶段状态，它以前各阶段的状态无法直接影响它未来的决策，而只能通过当前的这个状态。换句话说，每个状态都是过去历史的一个完整总结。这就是无后向性。

3.子问题的重叠性

    动态规划将原来具有指数级复杂度的搜索算法改进成了具有多项式时间的算法。其中的关键在于解决冗余，这是动态规划算法的根本目的。设原问题的规模为n，当子问题树中的子问题总数是n的超多项式函数，而不同的子问题数只是n的多项式函数时，动态规划法具有线性时间复杂性。所以，能够用动态规划解决的问题还有一个显著特征：子问题的重叠性。这个性质并不是动态规划适用的必要条件，但是如果该性质无法满足，动态规划算法同其他算法相比就不具备优势。

### 第3部分 动态规划的基本思想

    动态规划的实质是分治思想和解决冗余，因此，动态规划是一种自下而上将问题实例分解为更小的、相似的子问题，并存储子问题的解而避免计算重复的子问题，以解决最优化问题的算法策略。由此可知，动态规划法与分治法和贪心法类似，它们都是将问题实例归纳为更小的、相似的子问题，并通过求解子问题产生一个全局最优解。其中贪心法的当前选择可能要依赖已经作出的所有选择，但不依赖于有待于做出的选择和子问题。因此贪心法自顶向下，一步一步地作出贪心选择；而分治法中的各个子问题是独立的(即不包含公共的子子问题)，因此一旦递归地求出各子问题的解后，便可自下而上地将子问题的解合并成问题的解。但不足的是，如果当前选择可能要依赖子问题的解时，则难以通过局部的贪心策略达到全局最优解；如果各子问题是不独立的，则分治法要做许多不必要的工作，重复地解公共的子问题。解决上述问题的办法是利用动态规划。

    动态规划主要应用于最优化问题，这类问题会有多种可能的解，每个解都有一个值，而动态规划找出其中最优(最大或最小)值的解。若存在若干个取最优值的解的话，它只取其中的一个。在求解过程中，该方法也是通过求解局部子问题的解达到全局最优解，但与分治法和贪心法不同的是，动态规划允许这些子问题不独立，(亦即各子问题可包含公共的子子问题)也允许其通过自身子问题的解作出选择。


### 第4部分 动态规划实现中的问题

    应用动态规划解决问题，在有了基本的思路之后，一般来说，算法实现是比较好考虑的。但有时也会遇到一些问题，而使算法难以实现。动态规划思想设计的算法从整体上来看基本都是按照得出的递推关系式进行递推，这种递推相对于计算机来说，只要设计得当，效率往往是比较高的，这样在时间上溢出的可能性不大，而相反地，动态规划需要很大的空间以存储中间产生的结果，这样可以使包含同一个子问题的所有问题共用一个子问题解，从而体现动态规划的优越性，但这是以牺牲空间为代价的，为了有效地访问已有结果，数据也不易压缩存储，因而空间矛盾是比较突出的。另一方面，动态规划的高时效性往往要通过大的测试数据体现出来（以与搜索作比较），因而，对于大规模的问题如何在基本不影响运行速度的条件下，解决空间溢出的问题，是动态规划解决问题时一个普遍会遇到的问题。

  

动态规划的主要缺点是:

     没有统一的标准模型，也没有构造模型的通用方法，甚至还没有判断一个问题能否构造动态规划模型的具体准则（大部分情况只能够凭经验判断是否适用动态规划）。这样就只能对每类问题进行具体分析，构造具体的模型。对于较复杂的问题在选择状态、决策、确定状态转移规律等方面需要丰富的想象力和灵活的技巧性，这就带来了应用上的局限性。

    用数值方法求解时存在维数灾(curse of dimensionality)。若一维状态变量有m个取值，那么对于n维问题，状态xk就有mn个值，对于每个状态值都要计算、存储函数fk(xk)，对于n稍大(即使n=3)的实际问题的计算往往是不现实的。目前还没有克服维数灾的有效的一般方法。              

  

### 第5部分 动态规划与其他算法的比较

  

动态规划与递推——动态规划是最优化算法

    动态规划的实质是分治和解决冗余，因此动态规划也是递归思想的应用之一。但是，动态规划和递归法还是有区别的。一般我们在实际应用中遇到的问题主要分为四类：判定性问题、构造性问题、计数问题和最优化问题。动态规划是解决最优化问题的有效途径，而递推法在处理判定性问题和计数问题方面是一把利器。

    这个递推法的递推公式和动态规划的规划方程非常相似，我们在这里借用了动态规划的符号也就是为了更清楚地显示这一点。其实它们的思想也是非常相像的，可以说是递推法借用了动态规划的思想解决了动态规划不能解决的问题。

动态规划与搜索——动态规划是高效率、高消费算法

    动态规划可以解决的问题，搜索也可以解决。一般说来，动态规划算法在时间效率上的优势是搜索无法比拟的。从理论上讲，任何拓扑有序（现实中这个条件常常可以满足）的隐式图中的搜索算法都可以改写成动态规划。但事实上，在很多情况下我们仍然不得不采用搜索算法。一般说来，动态规划总要遍历所有的状态，而搜索可以排除一些无效状态，更重要的是搜索还可以剪枝，可能剪去大量不必要的状态，因此在空间开销上往往比动态规划要低很多。如何协调好动态规划的高效率与高消费之间的矛盾呢？有一种折衷的办法就是记忆化算法(备忘录法)。记忆化算法在求解的时候还是按着自顶向下的顺序，但是每求解一个状态，就将它的解保存下来，以后再次遇到这个状态的时候，就不必重新求解了。这种方法综合了搜索和动态规划两方面的优点，因而还是很有实用价值的。
 

第6部分 应用





## C3 贪心算法

一、算法思想

从问题的某一个初始解出发逐步逼近给定的目标，以尽可能快的地求得更好的解。当达到某算法中的某一步不能再继续前进时，算法停止。

在贪心算法（greedy method）中采用逐步构造最优解的方法。在每个阶段，都作出一个看上去最优的决策（在一定的标准下）。决策一旦作出，就不可再更改。作出贪心决策的依据称为贪心准则（greedy criterion）。

该算法存在问题：

1. 不能保证求得的最后解是最佳的；
2. 不能用来求最大或最小解问题；
3. 只能求满足某些约束条件的可行解的范围。

二、应用

45. Jump Game II
Medium
Given an array of non-negative integers nums, you are initially positioned at the first index of the array.
Each element in the array represents your maximum jump length at that position.
Your goal is to reach the last index in the minimum number of jumps.
You can assume that you can always reach the last index.

Example 1:
Input: nums = [2,3,1,1,4]
Output: 2
Explanation: The minimum number of jumps to reach the last index is 2. Jump 1 step from index 0 to 1, then 3 steps to the last index.

Example 2:
Input: nums = [2,3,0,1,4]
Output: 2
Constraints:
-   1 <= nums.length <= 1000
-   0 <= nums[i] <= 105

```
class Solution {
    public int jump(int[] nums) {
        int curStepEnd = 0, step = 0, farthest = 0;
        for (int i = 0; i < nums.length-1; i++) {
            farthest = Math.max(farthest, nums[i] + i);
            if (i == curStepEnd) {
                step++;
                curStepEnd = farthest;
            }
        }
        return step;
    }
}
```


## C4 分治算法
  
对于一个规模为n的问题，若该问题可以容易地解决（比如说规模n较小）则直接解决，否则将其分解为k个规模较小的子问题，这些子问题互相独立且与原问题形式相同，递归地解这些子问题，然后将各子问题的解合并得到原问题的解。这种算法设计策略叫做分治法。

### 一、分治法的基本思想

    分治法的设计思想是，将一个难以直接解决的大问题，分割成一些规模较小的相同问题，以便各个击破，分而治之。如果原问题可分割成k个子问题，1<k≤n ，且这些子问题都可解，并可利用这些子问题的解求出原问题的解，那么这种分治法就是可行的。由分治法产生的子问题往往是原问题的较小模式，这就为使用递归技术提供了方便。在这种情况下，反复应用分治手段，可以使子问题与原问题类型一致而其规模却不断缩小，最终使子问题缩小到很容易直接求出其解。这自然导致递归过程的产生。分治与递归像一对孪生兄弟，经常同时应用在算法设计之中，并由此产生许多高效算法。

### 二、分治法的适用条件

    分治法所能解决的问题一般具有以下几个特征：

（1）该问题的规模缩小到一定的程度就可以容易地解决；
（2）该问题可以分解为若干个规模较小的相同问题，即该问题具有最优子结构性质。
（3）利用该问题分解出的子问题的解可以合并为该问题的解；
（4）该问题所分解出的各个子问题是相互独立的，即子问题之间不包含公共的子子问题。

    上述的第一条特征是绝大多数问题都可以满足的，因为问题的计算复杂性一般是随着问题规模的增加而增加；第二条特征是应用分治法的前提，它也是大多数问题可以满足的，此特征反映了递归思想的应用；第三条特征是关键，能否利用分治法完全取决于问题是否具有第三条特征，如果具备了第一条和第二条特征，而不具备第三条特征，则可以考虑贪心法或动态规划法。第四条特征涉及到分治法的效率，如果各子问题是不独立的，则分治法要做许多不必要的工作，重复地解公共的子问题，此时虽然可用分治法，但一般用动态规划法较好。

### 三、应用

  
34. Find First and Last Position of Element in Sorted Array

Medium

Given an array of integers nums sorted in ascending order, find the starting and ending position of a given target value.
If target is not found in the array, return [-1, -1].
Follow up: Could you write an algorithm with O(log n) runtime complexity?
Example 1:
Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]
Example 2:
Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]
Example 3:
Input: nums = [], target = 0
Output: [-1,-1]
Constraints:

-   0 <= nums.length <= 105
-   -109 <= nums[i] <= 109
-   nums is a non-decreasing array.
-   -109 <= target <= 109
    
```
/**

* 34. Find First and Last Position of Element in Sorted Array
*
* @author CodeX
* @version 1.0
* @date 2021/4/2 16:08
*/

public class LC0034 {

    public int[] searchRange(int[] nums, int target) {
        int p;
        p = searchTarget(nums, target);
        int s = p, e = p;
        while (s - 1 >= 0 && nums[s - 1] == target) s--;
        while (e >= 0 && e + 1 < nums.length && nums[e + 1] == target) e++;
        return new int[]{s, e};

    }
  
    private int searchTarget(int[] nums, int target) {

        if (nums.length == 0) return -1;
        int start = 0, end = nums.length, mid;
        while (start < nums.length && end >= 0 && start <= end) {
            mid = (start + end) / 2;
            if (target == nums[mid]) return mid;
            if (target > nums[mid]) {
                start = mid + 1;
            } else end = mid - 1;
        }
        return -1;
    }
}
```
  



## C5 Dijkstra算法

Dijkstra算法是典型最短路算法，用于计算一个节点到其他所有节点的最短路径。主要特点是以起始点为中心向外层层扩展，直到扩展到终点为止。Dijkstra算法能得出最短路径的最优解，但由于它遍历计算的节点很多，所以效率低。
Dijkstra一般的表述通常有两种方式，一种用永久和临时标号方式，一种是用OPEN, CLOSE表方式，为了和下面要介绍的 A* 算法和 D* 算法表述一致，这里均采用OPEN,CLOSE表的方式。
其采用的是贪心法的算法策略
dijkstra不适合负数权重。如果边权重为负数，则都加到正数。

### 大概过程：

创建两个表，OPEN, CLOSE。
OPEN表保存所有已生成而未考察的节点，CLOSED表中记录已访问过的节点。
1． 访问路网中距离起始点最近且没有被检查过的点，把这个点放入OPEN组中等待检查。
2． 从OPEN表中找出距起始点最近且没有被检查过的点，找出这个点的所有子节点，把这个点放到CLOSE表中。
3． 遍历考察这个点的子节点。求出这些子节点距起始点的距离值，放子节点到OPEN表中。
4． 重复第2和第3步,直到OPEN表为空

  
```
package search;
import java.util.*;

/**
*
* @author CodeX
* @version 1.0
* @date 2021/4/26 22:36
*/

public class Dijkstra { 

    public static void main(String[] args) {
        List<Integer> node = new ArrayList<>();
        int[][] weights = new int[][]{{0, 0, 10, 0, 30, 100},
                {0, 0, 5, 0, 0, 0},
                {0, 0, 0, 50, 0, 0},
                {0, 0, 0, 0, 0, 10},
                {0, 0, 0, 20, 0, 60},
                {0, 0, 0, 0, 0, 0}};
        int start = 0;
        int[] dist = dijkstra(weights, 0);
        for (int i = 0; i < dist.length; i++) {
            System.out.println(dist[i]);
        }
    }

    private static int[] dijkstra(int[][] weights, int start) {

        HashSet<Integer> close = new HashSet();

        Map<Integer, Integer> open = new HashMap<>();
        int[] dist = new int[weights.length];

        for (int i = 0; i < dist.length; i++) {
            dist[i] = i == start ? 0 : Integer.MAX_VALUE;
        }

        open.put(start, 0);
        int closest = start;
        while (!open.isEmpty()) {
            //find closest
            int min = Integer.MAX_VALUE;
            for (Map.Entry<Integer, Integer> entry : open.entrySet()) {
                int key = entry.getKey();
                int value = entry.getValue();
                if (value < min) {
                    min = entry.getValue();
                    closest = key;
                }
            }
            open.remove(closest);
            close.add(closest);
            for (int i = 0; i < weights[closest].length; i++) {
                int weight = weights[closest][i];
                if (weight != 0) {
                    if (weight + dist[closest] < dist[i]) {
                        dist[i] = weight + dist[closest];
                    }
                    if (!close.contains(i)) open.put(i, dist[i]);
                }
            }
        }
        return dist;
    }
}
```
  




## C6 Floyd算法
Floyd算法又称为弗洛伊德算法，插点法，是一种用于寻找给定的加权图中顶点间最短路径的算法。

### 算法思想

通过一个图的权值矩阵求出它的每两点间的最短路径矩阵。
从图的带权邻接矩阵A=[a(i,j)] n×n开始，递归地进行n次更新，即由矩阵D(0)=A，动态规划得到D(n)。矩阵D(n)的i行j列元素便是i号顶点到j号顶点的最短路径长度，称D(n)为图的距离矩阵，同时还可引入一个后继节点矩阵path来记录两点间的最短路径。


### 算法描述

a)　初始化：
```
D[u,v]=A[u,v]　
```
b)　
```
　For k:=1 to n
　　　　For i:=1 to n
　　　　　For j:=1 to n
　　　　　　If D[i,j]>D[i,k]+D[k,j] Then
　　　　　　　D[i,j]:=D[i,k]+D[k,j];
```

c)　算法结束：D即为所有点对的最短路径矩阵

### 时间复杂度

O(n^3)

### 优缺点分析
Floyd算法适用于APSP(All Pairs Shortest Paths)，稠密图效果最佳，边权可正可负。此算法简单有效，由于三重循环结构紧凑，对于稠密图，效率要高于执行|V|次Dijkstra算法。
优点：容易理解，可以算出任意两个节点之间的最短距离，代码编写简单；
缺点：时间复杂度比较高，不适合计算大量数据。
```
/**
* Floyd
*
* @author CodeX
* @version 1.0
* @date 2021/4/27 12:18
*/

public class Floyd {

    public static void main(String[] args) {
        int max = Integer.MAX_VALUE;
        int[][] weights = new int[][]{
                {max, max, 10, max, 30, 100},
                {max, max, 5, max, max, max},
                {max, max, max, 50, max, max},
                {max, max, max, max, max, 10},
                {max, max, max, 20, max, 60},
                {max, max, max, max, max, max}};

        int[][] dp = floyd(weights);
        for (int i = 0; i < dp.length; i++) {
            StringBuffer sb = new StringBuffer();
            for (int j = 0; j < dp.length; j++) {
                sb.append(dp[i][j] + " ");
            }
            System.out.println(sb.toString().trim());
        }

    }

    private static int[][] floyd(int[][] weights) {
        int[][] dp = weights;
        int len = dp.length;
        for (int i = 0; i < len; i++) {
            for (int j = 0; j < len; j++) {
                for (int k = 0; k < len; k++) {
                    if (dp[j][i] != Integer.MAX_VALUE && dp[i][k] != Integer.MAX_VALUE && dp[j][k] > dp[j][i] + dp[i][k]) {
                        dp[j][k] = dp[j][i] + dp[i][k];
                    }
                }
            }
        }
        return dp;
    }
}
```
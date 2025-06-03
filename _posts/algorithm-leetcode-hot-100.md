---
name: leetcode-hot-100
title: 力扣热题100速通
date: 2025-05-17
categories: 算法
---



转载自 [力扣热题100 速通指南 - 小范同学](https://zhuanlan.zhihu.com/p/458506664) 

思路总结，用于复习。

[1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

解法1：暴力遍历 O(N^2)

解法2：字典。每遍历到一个数，先找目标值在不在字典里。若在，返回；若不在，当前数加入字典。

解法3：排序 + 双指针 O(nlogn)

[2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

循环 while l1 or l2，adder1 = l1.val if l1 else 0，adder2也这样操作。循坏外设carry=0，当前位res = (adder1 + adder2) % 10，carry = (adder1 + adder2) // 10，新建一个节点保存当前res，后移。退出循环后如果有carry位就新建一个节点，如果没有就算。

[3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

同[剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

解法1：dp+字典

dp[i]表示以i结尾的最长不含重复字符的子字符串，字典记录字符s[i]上一次出现的位置下标（如果没有出现过，默认值-1）。特别注意，**是上一次出现**，不是上一次在滑动窗口中出现，虽然我们也可以用i - dp[i-1]得到左端点，但这样比较麻烦。

判断上一次出现的位置是否在滑动窗口中：i-dict[i] > dp[i-1]。前面这个数是[上一个s[i], s[i-1]]的长度，它大于dp[i-1]说明上一个s[i]在dp[i-1]左端点的左边

转移方程：

> 下面这样写是不对的，因为dict应该是上一次出现，而不是上一次在滑动窗口中出现。
> dp[i] = dp[i-1] + 1 if nums[i] not in dict
> dp[i] = i - dict[nums[i]] if nums[i] in dict

正确的转移方程

> dp[i] = dp[i-1] + 1 i-dict[i] > dp[i-1]
> dp[i] = i - dict[nums[i]] else

完事更新hash表。最后输出dp[]最大值

解法2：双指针+字典，可以用双指针是因为我们发现右端点右移左端点也右移，有单调性。遍历right，left = max(left, dict[s.get(right, -1)]+1)，更新哈希表dict[s[right]] = right，更新结果res = max(res, right - left + 1)

ps. k神答案里给的左边界是开区间。

需要注意的是，由于ASCII码是0~127，所以两种解法空间复杂度是O(1)

[4. 寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

解法1：merge 时间空间都O(m+n)

解法2：二分查找 O(log(m+n)) TODO

[5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

解法1：dp（不推荐）
由于回文串必须要左右对称，很明显一个状态变量已经无法描述问题，所以使用二维dp。dp[i][j]表示字符串s[i,j]是否是回文串，转移方程是 P(i,j)=P(i+1,j−1)∧(Si​==Sj​)
边界条件：长度为1的子串是回文串，长度为2的看这两个字符是否相同
递推：从长度为1的开始
记录结果：只要dp[i][j] == True，比较长度，若为最长则记录长度和下标
时间空间都是O(N^2)，因为要给每个状态计算转移方程

解法2：中心扩展法（推荐）

1. 首先写一个中心扩展函数：输入扩展中心左右端点坐标，当左右端点相同时向外扩展，直到无法扩展时，输出最长扩展字串的左右端点
2. 按照“串中每一个点”，“串中每相邻一对点”作为扩展中心，开始扩展，一旦返回的子串长度大于最大长度，将其记录下来。

时间O(N^2)：长度为1的回文中心有n个，长度为2的回文中心有(n-1)个，每个回文中心最多向外扩展n次
空间O(1)

[10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

由于是评估整个串是否匹配，使用动态规划。由于模式串和文本串各自有一个指针，所以应该是二维dp：

dp[i][j]表示文本串前i个位置，模式串前j个位置是否匹配。TODO



[11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

> 注意这道题和[42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/) 不同。本题是“板”，即相邻两板之间可以接水。42题是块，即相邻两块之间不能接雨水。这将引起x坐标计算的不同。

这个问题是个具有单调性的问题，解法一定是首尾双指针。面积是(right - left) * min(height[left], height[right])

> **特别注意这里是right-left不是right-left+1**，前者是right和left的间距，后者是left和right之间有几个数（包括端点）

基本思想就**是若向内移动长板，（由于min这部分要么变小要么不变）则面积一定缩小。若向内移动短板，它有可能变长，所以面积有可能变大。**

那么我们就不断移动短板，一直算面积，直到双指针汇聚，所有算过的这些面积里面最大的就是结果。

[15. 三数之和](https://leetcode-cn.com/problems/3sum/)

TODO



[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

标准试探回溯法

基本流程：找状态变量、明确退出条件、找选择列表、剪枝、做出选择并递归深入、撤销选择

这里首先来个map，记录每个数字对应的字符们

状态变量：电话号码的第几位curr。退出条件是curr=len(nums)，选择列表是迭代dict[curr]，不需要剪枝，做出选择是append当前字符并递归深入，撤销选择是pop当前元素



[19. 删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

很简单，快慢双指针

需要注意的是退出条件。如果fast比slow快n步，fast是倒数第1个时slow是倒数第n个，此时fast.next==None，所以循环条件是while fast.next

另外一种方法比较取巧，链表头部弄个哑节点，slow从哑节点启动，循环条件为while fast，这时候fast比slow快(n+1)步

[20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

括号匹配问题。（由于需要倒序比较）使用栈。具体思路是：先建立一个左右括号匹配字典，遍历序列，如果遇到左括号就入栈，如果遇到右括号，若栈为空或者栈顶括号不与当前右括号匹配，返回false。最后来看栈是否为空。

[21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

mergesort的合并一步。没什么难的，就是while l1 and l2: 拼接最小的到新链表，退出循环后if l1就把l1拼后面，if l2就把l2拼后面，最后return dummy.next

[22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

试探回溯法入门题。先回忆一下步骤：确定状态变量、确定退出条件 、确定选择列表、剪枝、做出选择并深入、撤销选择。

对这道题而言，由于我们要一个一个括号添加，而括号又分为左、右两种，因此以“左括号剩余个数”“右括号剩余个数”同时为状态变量。退出条件是左右括号个数均剩余为0。选择列表就是添加左括号or添加右括号。剪枝条件就是剩余左括号数大于剩余右括号数（说明已经摆放的右括号数大于已经摆放的左括号数），和剩余左右括号数小于0（说明有透支）

时间复杂度：取决于有多少个组合
空间复杂度：递归栈最深为2n，所以是O(n)

[23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

方法1：以一个链表为结果链表，其他链表不断合并上来

方法2：类似数组归并排序一样，对所有链表头节点组成的数组分而治之，链表两两合并，最终得到一个大链表

**方法3（推荐）：**使用堆。首先我们先把所有链表取出一个元素入堆（注意只有在if l的时候才入堆，要把空链表过滤掉），然后从堆里取出最值元素连在结果链表中，并将该元素的next节点入堆

需要注意的一点是，入堆的元素需要是(l.val, l_index, l)这种tuple，因为同值的时候堆会按照第二个元素排序，所以第二个位置要放index。第三个位置放节点本身，这是为了找他在链表中的next节点用。

[31. 下一个排列](https://leetcode-cn.com/problems/next-permutation/)

左边找一个较小数，右边找一个较大数，交换，排序交换后较大数的右侧。

具体而言：从右向左找第一个顺序对，其中较小那个就是我们需要的较小数。我们从这个数以右（从右向左，因为这个区间必然是大->小的）找第一个比他大的数，这个数就是较大数。交换这两个数，之后右边这个区间必然是逆序的，因此我们要翻转这个区间（首尾双指针）。

另外注意，对于已经是最大排列的这种情况，没有第一步和第二步，只有翻转区间。因此我们在做第一步和第二步的时候要先检验下标是否合法。

[33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

这个题不同于剑指里的题，我们是搜索任意值target。但其基本思想一致，就是不断剔除单调区间。

TODO

[42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

> 注意这道题和[11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/) 不同。11题是“板”，即相邻两板之间可以接水。而这道题是块，即相邻两块之间不能接雨水。这将引起x坐标计算的不同。

解法1：最值数组。加和每个位置正上方能接雨水的位置。发现一个规律，每个位置正上方能够接雨水的数量是min(left[i], right[i]) - height[i]。所以就是要先构建左起最大高度数组left、右起最大高度数组right。时间复杂度O(n)，空间复杂度O(n)

解法2：单调栈。逐层横向计算。利用单调栈，

解法3：双指针。

TODO

[48. 旋转图像](https://leetcode-cn.com/problems/rotate-image/)

解法1：推公式（麻烦）

解法2：用翻转代替旋转。上下翻转一下，沿左上到右下的对角线翻转一下。

[53. 最大子数组和](https://leetcode-cn.com/problems/maximum-subarray/)

dp。由于要连续，dp[i]表示以当前字母结尾的连续子数组最大和。那么状态转移方程就是看上一个位置的dp数组是否为正，从而决定子数组要不要之前的部分，还是从当前部分重新开始，即：

dp[i] = dp[i-1] + nums[i] if dp[i-1] > 0
dp[i] = nums[i] else

另外注意一个技巧：我们可以在递推计算dp数组的同时记录数组最大值，这样就避免了重复遍历dp数组。最大值变量初始值可以置为-inf或者num[0]，注意不能置为0因为数组内可能全是负数。

[55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

greedy TODO

[56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

排序+遍历。**注意原始数组可能并不是有序的，因此先sort**(nums, key=lambda elem:elem[0])，然后遍历。

- 其实这里要用栈来处理，因为我们总是在修改最后一个元素。
- 遍历数据，如果栈不是空，pop出来一个元素，比较popped[1]和curr[0]，如果popped[1]<curr[0]说明可以合并，新的左端点还是popped[0]，右端点是max(popped[1], curr[1])。
- 除此之外（即popped[1]>=curr[0]），说明不能合并，直接把curr加到结果里

[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

很明显是dp。由于每次只能爬1或2个台阶，dp[i] = dp[i-1] + dp[i-2]

需要注意的是边界条件。dp[0] = 0（没啥意义，数组里凑数的），dp[1] = 1，dp[2] = 2（因为不等于dp[0] + dp[1]）

[72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

方法1：暴力递归。这个问题很显然可以通过减而治之的方式化成规模更小的问题：

- 递归基：只要有一个串是空，返回另一个串的长度max(len(word1), len(word2))
- 减而治之：
  - 如果两个串末尾字符相同：可以转化为两个串都去掉尾字符得到的子问题
  - 如果两个串末尾字符不同：分为增（word2去掉尾）删（word1去掉尾）改（word1、word2都去尾）三种情况，那就是三个子问题中的最优者（min）得到的答案再+1

**方法2：动态规划（推荐）**

首先由于有两个串，长度还不一定一样长，应该是一个二维dp问题。dp[i][j]表示word1的前i个字符转变为word2的前j个字符需要的最小操作数。
**边界条件：**如果一个字符串是空，答案就应是另一个字符串的长度：

```python3
for i in range(len(word1)+1):
    dp[i][0] = i
for j in range(len(word2)+1):
    dp[0][j] = j
```

转移方程：和上面递归一样分为两种情况：

- 两子串尾字符相同（表达成条件就是i>0 and j > 0 and word1[i-1] == word2[j-1]）：两串各去掉尾字符结果不变，因此dp[i][j] = dp[i-1][j-1]

> 注意这里的条件表达，dp[i][j]表示的是word1[0:i]与word2[0:j]（右边都是开区间），最后一个字符是word1[i-1]和word2[j-1]

- 两子串尾字符不同：增、删、改三种情况

```python3
creat = dp[i][j-1] + 1
delete = dp[i-1][j] + 1
update = dp[i-1][j-1] + 1
dp[i][j] = min(creat, min(delete, update))
```

返回：dp[-1][-1]

时间、空间复杂度都是O(mn)

[78. 子集](https://leetcode-cn.com/problems/subsets/)

典型试探回溯。步骤：确定状态变量、确定结束条件、确定选择列表、剪枝、做出选择并递归深入、撤销选择

状态变量是在数组内的位置，结束不做任何事情（因为每个选择都需要被记录，因此一进递归函数就记录，注意要深拷贝），选择列表是当前字符后的所有字符，不需要剪枝，选择就是append，深入，撤销pop

复杂度？注意**子集的数量是2^n**，这是因为对于每个元素而言都有“在”和“不在”之一。我们每找到一个子集，需要O(n)时间将其拷贝进入结果，因此是O(n*2^n)

[94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

递归法就不说了

迭代法：

- 特殊情况：空树直接返回
- 新建一个栈（用来保存待访问的根节点），新建一个curr节点并初始化为root（用来保存当前发现节点）
- 当栈不空或curr不空时循环
  - 如果curr节点存在，说明发现一个新节点，入栈并转向左子节点
  - 否则，说明已经向左走到了尽头，此时从栈中pop出一个节点访问，并转向右子节点

```python3
def inorderTraversal(self, root: TreeNode) -> List[int]:
        if not root: return []
        res = []
        s = []
        curr = root
        while s or curr:
            if curr:
                s.append(curr)
                curr = curr.left
            else:
                curr = s.pop()
                res.append(curr.val)
                curr = curr.right
        return res
```

[96. 不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

该问题有最有子结构性质，因为以一个点为根节点的二叉搜索树数量，等于其不同的左子树数*不同的右子树数。因此只要让所有的点依次成为根节点，然后求和即可。

记G(i)为长度为i的序列有二叉搜索树的个数，则 G(i)=Σj=0iG(j−1)∗G(i−j)

```python3
def numTrees(self, n: int) -> int:
        G = [0] * (n+1)
        G[0] = 1
        G[1] = 1
        for i in range(2, n+1):
            for j in range(i+1): # 注意这里不能错，上面是加到n这里必须是i+1
                G[i] += G[j-1] * G[i-j]
        return G[n]
```

[98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

**递归解法：**记录中序遍历的前一个节点。全局变量self.pre = float('-inf')，然后中序遍历。递归基返回True，进入左子树，当前节点比较，小于等于上一节点返回false，更新pre，进入右子树。递归回溯阶段返回左子树且右子树。
时间O(n)：因为每个节点最多被访问一次
空间O(n)：递归栈最深为整个树所有节点入栈（二叉树退化成链表）

**迭代解法：**中序遍历迭代，记录上一个节点，在访问当前节点时作验证，且更新上一个节点！
时间O(n)：因为每个节点最多被访问一次
空间O(n)：栈最深为整个树所有节点入栈（二叉树退化成链表）

[101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

**递归法：**

双指针法。新定义一个递归函数，参数是两个指针。
**递归基：**左右均空则返回true
**剪枝：**左右一个空一个不空，或者左右值不相等返回false
**深入与回溯：**深入左右，答案的and返回

**迭代法：**

层次遍历。当我们层次遍历的时候，一层的节点应该互为镜像。当一层只有两个节点的时候，他们应该相等。因此，如果我们层次遍历时对第一个节点的左右子节点正向入队，对第二个节点的左右子节点反向入队，则队列里相邻的两个节点都应该相等。

```cpp
class Solution {
public:
    bool check(TreeNode *u, TreeNode *v) {
        queue <TreeNode*> q;
        q.push(u); q.push(v);
        while (!q.empty()) {
            u = q.front(); q.pop();
            v = q.front(); q.pop();
            if (!u && !v) continue;
            if ((!u || !v) || (u->val != v->val)) return false;

            q.push(u->left); 
            q.push(v->right);

            q.push(u->right); 
            q.push(v->left);
        }
        return true;
    }

    bool isSymmetric(TreeNode* root) {
        return check(root, root);
    }
};
```

[102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

需要确定层，可以通过记录队列元素的做法。具体而言，可以利用双层循环，外层while queue，内层for _ in range(len(queue))，因此每打印完一层就会退出内层，就可以append结果。

记得**from collections import deque**

[104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

法1：自上而下计算

法2：自下而上计算

法3：层次遍历

[114. 二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)

解法1：保存结果至列表，然后对结果列表的元素进行修改

> 需要注意的是解法2和3中，需要开一个变量保存遍历序列中的上一个节点。**我们每遍历到一个节点时，要连接这个节点和上一个节点**（因为下一个节点尚未访问，修改它风险太大）。也就是说我们修改的是curr.left和prev.right。
> **对于curr.left:** 在本题中是先序遍历，肯定是不能修改curr.left的因为visit到当前节点时左节点还没被发现呢，但好在我们不需要做双向链表，只需要将left指针置空，因此我们每访问到一个节点将prev.left置空就可以了。
> **对于prev.right:** 由于是先序遍历，上一节点的right是还没被发现的，不能直接修改，所以需要先备份它

解法2（推荐）--迭代先序遍历：在迭代版本压栈的时候，先压右再压左，prev.right的引用已经在栈里了，可以修改了

解法3（推荐）：递归先序遍历+存右节点引用：预先将prev.right的引用复制，因此可以修改。

[121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

使用股票问题通解

[128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

解法1（暴力法，不合要求，仅提供思路）：
遍历序列，对每个元素num在序列中寻找num+1、num+2等，复杂度O(n^2)

解法2：集合hash（推荐）
上面的解法有两点可以改进：

- 对于查找num+1、num+2 ... 的方式，暴力遍历过于低效，可以使用哈希替代
- 不是每一个元素都需要成为查找的起点，只有每个连续序列的左端点应该成为查找起点。比如1234这个序列，以1开始查找234之后，就不要再以2开始查找34了。具体的办法就是仅有num-1不存在于序列中时（是连续序列左端点），才启动对num+1、num+2的查找

所以具体做法就是：先把所有数加入set，然后开始对序列遍历，如果num-1在set中直接continue，否则开始记录长度并不断寻找num+1、num+2，直至num+n不在set中，此时将长度与最大长度对比

解法3：字典hash

basic idea：新来一个数，假如可以更新某个有序区间的长度，那一定与这个区间的端点相连。我们建立一个字典，字典中的每一个位置表示以该点为端点的区间最大的长度。

具体做法：新建一个**空字典**，遍历序列，如果序列中元素在字典里则跳过，否则计算该点造成的连续区间长度：

```python3
left_len = mp.get(num-1, 0)
right_len = mp.get(num+1, 0)
total_len = left_len + right_len + 1
```

然后更新左右端点的最大长度

```python3
mp[num-left_len] = total_len
mp[num] = total_len
mp[num+right_len] = total_len
```

然后比较该长度与最大长度

[136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

位运算 所有数字异或一遍，留下来的就是只出现一次的数字

[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

本题只需要判断是否有环，不需要找环的位置

快慢指针：慢指针一次一步，快指针一次两步。如果有环，二者终会相遇。

注意循环条件是fast and fast.next因为fast每次走两步。而且如果fast.next不存在，说明也走到尽头了。

[146. LRU 缓存](https://leetcode-cn.com/problems/lru-cache/)

要自己实现一个双向链表。需要功能：添加到首部（添加节点用到）、删除节点（移动时用到）、删除尾部（超出容量、移动节点时用到）、移动到首部（访问时用到）

TODO

[152. 乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)

dp。但需要注意，乘法不同于加法，一个负号就会使得最小值变成最大值。所以我们需要同时维护最小值数组和最大值数组。

fmax[i]表示前i个数的最大乘积，fmin[i]表示前i个数的最小乘积。
**边界条件：**fmax[0]和fmin[0]都是nums[0]

**转移方程：**

- fmax[i]在fmax[i-1]*nums[i]、nums[i]、fmin[i-1]*nums[i]中取max
- fmin[i]在fmax[i-1]*nums[i]、nums[i]、fmin[i-1]*nums[i]中取min

**返回：**max(fmax)

[155. 最小栈](https://leetcode-cn.com/problems/min-stack/)

双栈，一个存数据，一个存最小值，同步压栈弹栈

**简单方法：**压栈时最小栈中压栈栈顶元素和将要入栈元素中的最小值。弹栈时最小栈和数据栈同时弹栈。

**复杂方法：**压栈时只有待入栈元素小于等于栈顶元素时入栈。弹栈时如果待弹栈元素大于最小栈栈顶则最小栈不弹栈。

[160. 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

解法1：栈

开两个栈，两个链表分别入栈。当栈顶元素相同时pop，不同时停止pop，最后一个弹出的元素就是交点。

解法2：双指针

p1和p2共同前进，一旦自己为空就进入另一个链表，直至p1==p2

解释：假设两链表的公共长度为c（可以为0），两链表本身长为a和b，公共节点为node（可以为none），那么p1和p2走到node时，各走了a-c+b和b-c+a，他们是相等的，所以一定是相遇了。此时返回p1或p2即可（如果没有交点，他们也同时为none）

循环条件是p1 != p2，return p1或者return p2都可以

> 另外注意：**要允许p1、p2为None**。None的下一个再接另一个链表的头节点

[169. 多数元素](https://leetcode-cn.com/problems/majority-element/)

摩尔投票。先判断为0，然后做加减。

具体实现：遍历整个数组，每个元素的处理分为两个部分，**第一部分是majority的赋值，第二部分是票数的计算**：

- majority的赋值：当票数为0时将majority的值赋为当前元素。
- 票数的计算：若当前元素==majority，则vote++，否则vote--

[200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

dfs/bfs/并查集

dfs：
**主函数：**遍历整个矩阵，遇到1就count++，并发起一次dfs。
**dfs函数**（只有一个作用就是尽可能多的把1置为0）：递归基：越界、当前是0，做出选择（当前位置置0），上下左右递归深入。

ps. 注意置为0之后就不再置回1了，以避免重复计数

bfs：关于bfs我们可以参考下这道题[542. 01 矩阵](https://leetcode-cn.com/problems/01-matrix/) 。

> 复习一下bfs的基本流程：首先找起点入队并标记发现状态启动bfs，然后队列中拿出一个节点作为当前节点，访问当前节点，**将当前节点所有未发现的邻居入队并标记发现状态，同时标注邻居节点与当前节点的关系。**所有节点有三种状态：未发现、已发现未访问（队列中，已标注发现状态）、已访问（所有邻居节点已入队，已标注当前节点与所有邻居节点的关系）

TODO

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

因为要修改当前节点的next指针，因此需要备份下一个节点的引用。同时因为要把当前节点的next指针指向上一节点，因此需要备份上一节点prev。最后的时候curr为空，返回prev

[215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

虽然“第k”和“k个”看起来不同，但实际上等价于 [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/) ，这是因为我们使用的是快排找轴点，轴点一侧的数必然比他大/小。

partition函数就是快排的partition。主函数可以按快排写，就是先partition再按照pivot递归二分（注意可以剪枝）。递归基是pivot==k（递归二分）。也可以直接迭代二分查找。partition函数在循环里，根据pivot与len-k的关系，做二分。

[221. 最大正方形](https://leetcode-cn.com/problems/maximal-square/)

易知，边长有最优子结构性质但面积没有。那么我们对变长dp，最后平方得面积。

二维dp，dp[i][j]表示位置(i, j)为右下角点的最大正方形边长。易知边界条件为第一排和第一列为1或0（取决于矩阵在该点的值），状态转移方程是当matrix[i][j] == 1时有dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])+1，当matrix[i][j]==0时dp[i][j] = 0

**注意这个转移方程要有三项！**这是因为一个正方形左、上、左上三个位置有一个边长较小，都会影响到该正方形的边长。

相似题[1277. 统计全为 1 的正方形子矩阵](https://leetcode-cn.com/problems/count-square-submatrices-with-all-ones/) 。这个题是求子矩阵的个数。特点在于，**对于以(i, j)为右下角的最大正方形边长就等于以(i, j)为右下角的正方形子矩阵个数**，那么最后我们把所有的dp[i][j]加起来就ok


[226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

选择任何一种遍历即可，visit当前节点时候把左右子树做下交换，然后按照正常遍历的方式进行递归，最后返回一下root

[234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

解法1：复制到数组中，首尾双指针

解法2：递归。但空间复杂度仍然没达标。

递归深入的目的是从后向前遍历，再开一个全局变量p（初始化为head）从前往后遍历。具体实现而言：

- 递归基：当前节点为空，返回true
- 递归深入+剪枝：一旦上一层递归传出的结果是false，不再额外操作，直接返回false
- 剪枝：如果p的值和当前层指针的值不同，直接返回false
- p前进
- 返回true

解法3：**翻转链表（推荐）**





[283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)

类似[剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

其实就是快排的partition函数。partition函数有两种实现：快慢双指针和首尾双指针。由于需要保持相对顺序，那么只能用快慢双指针。



[300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

这个题其实有难度。因为它要的是最长上升子序列的长度，新来一个字符他有可能跟之前的多个地方都构成新的上升子序列。所以直接dp需要两个for

**解法1：dp**。dp[i]表示以nums[i]结尾的最长上升子序列长度。

- **初始化（边界条件）：**每个数自己组成一个上升子序列，所以dp[i]=1
- **状态转移：**对于每一个数nums[i]，遍历j=0->i，如果nums[i]>nums[j]说明能构成上升子序列且这个上升子序列结尾是nums[i]倒数第二个数是nums[j]，因此更新dp[i] = max(dp[i], dp[j]+1)
- **最后返回：**max(dp)

解法2：贪心



[338. 比特位计数](https://leetcode-cn.com/problems/counting-bits/)

**方法1：暴力遍历**

对每一个数都计算一下1的个数。具体计算的方法是，利用n&(n-1)可以消除最低位的1

方法2：利用最高有效位做dp

方法3：利用最低有效位做dp

[448. 找到所有数组中消失的数字](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array/)

哈希计数：开一个长度为n的数组，遇到一个数就把他数组对应位置++，最后再遍历一遍这个数组，把为0的都添加到结果

原地哈希计数：由于最大为n，那么我们遇到一个数就可以把它%n的对应位置的数+n。那么只要出现过，对应位置的数必然>n，那么我们就把<=n的数下标全作为结果。



[461. 汉明距离](https://leetcode-cn.com/problems/hamming-distance/)

异或，比特位计数

[560. 和为 K 的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

首先想到双指针，但其实不对，因为数组不是排序的。因此考虑使用**前缀和数组**。

**注意：快速实现滑动窗口累积值计算，要用前缀和数组！！！**

先弄一个前缀和数组pre。我们发现区间(i, j]等于k只需要pre[j] - pre[i] == k，这就成了两数之和问题。我们弄个字典来放该位置之前pre[]所有的取值，key是取值，value是个数。每次我们先累加结果，再把当前pre更新到dict中

[617. 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

这个题有个地方需要注意。当none了以后还需要访问其左右节点怎么办？答案是递归深入时做下判断。

递归基：两个树都为none。处理当前节点：新建一个node，如果指针1不为空，加指针1.如果指针2不为空，加指针2.递归深入：当前节点的左右引用=递归进入左右子树。注意只有当前节点存在时传入curr.left或curr.right，**如果当前节点不存在，传入None。**返回：返回当前刚新建的节点

[739. 每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

[小范同学：谈单调队列和单调栈](https://zhuanlan.zhihu.com/p/447209490) 

单调栈。**注意单调栈、单调队列里放的都是下标！！**

这道题求的是右边第一个更大值。用单调递增栈。

具体而言，回忆下单调栈流程：遍历数组：当栈不空，或者当前元素比新来元素小，循环弹出当前元素，标记当前元素的结果为新来元素下标-当前元素下标。循环退出后压栈当前元素下标。数组遍历完后，栈中元素依次弹栈，结果标记为0
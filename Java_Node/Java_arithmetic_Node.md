[TOC]



# JAVA算法

## 第一章：初级算法（49题）

### 1.1数组（11题）

#### 1.删除排序数组中的重复项

 给你一个 **升序排列** 的数组 `nums` ，请你 原地删除重复出现的元素，使每个元素 **只出现一次** ，返回删除后数组的新长度。元素的 **相对顺序** 应该保持 **一致** 。 

不要使用额外的空间，你必须在 **原地修改输入数组** 并在使用 O(1) 额外空间的条件下完成。

```
输入：nums = [0,0,1,1,1,2,2,3,3,4]
输出：5, nums = [0,1,2,3,4]
解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。
```

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums== null || nums.length == 0) { //边界条件
            return 0;
        }
        
        int i=0;
        for(int j = 1; j < nums.length; j++){        //右指针始终向右移动，而左指针在特定条件下才会移动
            //如果左指针和右指针指向的值一样，说明有重复的，
            //这个时候，左指针不动，右指针继续往右移。如果他俩
            //指向的值不一样就把右指针指向的值往前挪
            if(nums[i] != nums[j]){
                i = i + 1;
                nums[i] = nums[j];
            }
        }
        return i+1;

    }
}
```

**涉及知识点：使用双指针解决问题**

==使用两个指针，右指针始终往右移动==

- 如果右指针指向的值等于左指针指向的值，左指针不动。
  如果右指针指向的值不等于左指针指向的值，那么左指针往右移一步，然后再把右指针指向的值赋给左指针。

![image-20220916090051872](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220916090051872.png)

![image-20220916090114963](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220916090114963.png)

第二种解题方式：

```java
class Solution {
     public int removeDuplicates(int[] A) {
        int count = 0;//重复的数字个数
        for (int right = 1; right < A.length; right++) {
            if (A[right] == A[right - 1]) {
                //如果有重复的，count要加1
                count++;
            } else {
                //如果没有重复，后面的就往前挪
                A[right - count] = A[right];
            }
        }
        //数组的长度减去重复的个数
        return A.length - count;
    }
}
```

**这里的count用于计数，先确定有多少个值是重复的，再将后面不重复的值赋值给前面应该修改的值**





#### 2.买卖股票的最佳时机 II

给你一个整数数组 prices ，其中 prices[i] 表示某支股票第 i 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 最多 只能持有 一股 股票。你也可以先购买，然后在 同一天 出售。

返回 你能获得的 最大 利润 。

```
输入：prices = [1,2,3,4,5]
输出：4
解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     总利润为 4 。
```

```
class Gupiao {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length < 2)
            return 0;
        int length = prices.length;
        int[][] dp = new int[length][2];
        //初始条件
        dp[0][1] = -prices[0];
        dp[0][0] = 0;
        for (int i = 1; i < length; i++) {
            //递推公式
            dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);//当天手里没有任何交易 or 当天把手里的股票给卖了  i+1天交易完之后手里没有股票的最大利润
            dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);//第i+1天交易完之后手里持有股票的最大利润。一种是当天没有任何交易  还一种是当天买入了股票
        }
        //最后一天肯定是手里没有股票的时候，利润才会最大，
        //只需要返回dp[length - 1][0]即可
        return dp[length - 1][0];
    }
}
```

**涉及知识点：动态规划**

#### 3.旋转数组



#### 4.存在重复元素



#### 5.只出现一次的数字



#### 6.两个数组的交集



#### 7.加一



#### 8.移动零



#### 9.两数之和



#### 10.有效的数独



#### 11.旋转图像



#### 12.移除元素

**题目描述：**

```
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2]
解释：函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。你不需要考虑数组中超出新长度后面的元素。例如，函数返回的新长度为 2 ，而 nums = [2,2,3,3] 或 nums = [2,2,0,0]，也会被视作正确答案。
```

**题解：**

```java
class Solution {
  public int removeElement(int[] nums, int val) {
​    int n = nums.length;
​    int left = 0;
​    for (int right = 0; right < n; right++) {
​      if (nums[right] != val) {
​        nums[left] = nums[right];
​        left++;
​      }
​    }
​    return left;
  }
}
```

**思路：调换索引位置**

### 1.2字符串（9题）

#### 1.反转字符串

#### 2.整数反转

#### 3.字符串中的第一个唯一字符

#### 4.有效的字母异同位

#### 5.验证回文串

#### 6.字符串转化整数

#### 7.实现strStr（）

#### 8.外观数列

#### 9.最长公共前缀

### 1.3链表（6题）

#### 1.删除链表中的节点

#### 2.删除链表的倒数第N个节点

#### 3.反转链表

#### 4.合并两个有序链表

#### 5.回文链表

#### 6.环形链表

### 1.4树（5题）

#### 1.二叉树的最大深度

#### 2.验证二叉搜索树

#### 3.对称二叉树

#### 4.二叉树的层序遍历

#### 5.将有序数组转化为二叉搜索树

#### 6.二叉树的遍历

**二叉树的前序遍历**

```
public static void preOrderTraveral(TreeNode node){
        if(node == null){
            return;
        }
        System.out.print(node.data+" ");
        preOrderTraveral(node.leftChild);
        preOrderTraveral(node.rightChild);
    }
```

**二叉树中序遍历**

    public static void inOrderTraveral(TreeNode node){
        if(node == null){
            return;
        }
        inOrderTraveral(node.leftChild);
        System.out.print(node.data+" ");
        inOrderTraveral(node.rightChild);
    }

 **二叉树后序遍历**

```
public static void postOrderTraveral(TreeNode node){
        if(node == null){
            return;
        }
        postOrderTraveral(node.leftChild);
        postOrderTraveral(node.rightChild);
        System.out.print(node.data+" ");
    }
```



### 1.5排序和搜索（2题）

#### 1.合并两个有序数组

#### 2.第一个错误的版本

### 1.6动态规划（4题）

#### 1.爬楼梯

#### 2.买卖股票的最佳时机

#### 3.最大子序和

#### 4.打家劫舍

### 1.7设计问题（2题）

#### 1.打乱数组

#### 2.最小栈

### 1.8数学（4题）

#### 1.Fizz Buzz

#### 2.计数质数

#### 3.3的幂

#### 4.罗马数字转整数

### 1.9其他（6题）

#### 1.位1的个数

#### 2.汉明距离

#### 3.颠倒二进制位

#### 4.杨辉三角

#### 5.有效的括号

#### 6.缺失数字




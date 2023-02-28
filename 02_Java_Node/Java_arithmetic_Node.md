[TOC]

> [算法基础知识](D:\04_学习笔记\02_Java_Node\刷题总结.md)



## 基础算法

#### 1.求最大公约数（辗转相除法）和最小公倍数

```java
public static int gcd (int a,int b) {
      int c;
      while (b!=0) {
        c = a%b;
        a = b;
        b = c;
	  }
    return a;
}

//最小公倍数-两数相乘后除以最大公约数
a*b/gcd(a,b)
```

## 第一章：数组

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
        for(int j = 1; j < nums.length; j++){    //右指针始终向右移动，而左指针在特定条件下才会移动
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

![image-20221207120704883](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221207120704883.png)

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

![image-20221207120632034](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221207120632034.png)

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

```java
public class test2 {
    public static void main(String[] args) {
        int[] arr1=new int[]{1,2,6,9,1,5};
        System.out.println(maxProfit(arr1));
    }

    public static int maxProfit(int[] prices) {
        int max = 0;
        for (int i = 1; i < prices.length; i++) {
            if (prices[i] > prices[i - 1]) {
                max = max + (prices[i] - prices[i - 1]);
            }
        }
        return max;
    }
}

```

![image-20221207120515423](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221207120515423.png)

#### 3.旋转数组

给你一个数组，将数组中的元素向右轮转 k 个位置，其中 k 是非负数。

示例 1:

```
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
```

```java
public class test {
    public static void main(String[] args) {

        int k=3;
        int[]arr=new int[]{1,2,3,4,5,6,7};//5 6 7 1 2 3 4
        Solution solution = new Solution();
        solution.twoSum(arr,k);
    }
}

class Solution {
    public void rotate(int[] nums, int k) {
        int length=nums.length;
        k=k%nums.length;
        reverse(nums, 0, length - 1);//旋转全部7 6 5 4 3 2 1 
        reverse(nums,0,k-1); //旋转前部分 5 6 7 4 3 2 1
        reverse(nums,k,length-1); //旋转后部分 5 6 7 1 2 3 4

    }

    public void reverse(int [] nums, int start ,int end){
		/**
         * 6->0
         * 5->1
         * 4->2
         */
        while (start<end){
            int tmp=nums[start];
            nums[start]=nums[end];
            nums[end]=tmp;
            start++;
            end--;
        }
    }
}
```

```java
class Solution {
    public void rotate(int[] nums, int k) {
        
        int[] tmp=new int[nums.length];

        int j=0;
        for(int i=0;i<nums.length;i++){
            tmp[j++]=nums[i];
        }

        for(int e=0;e<nums.length;e++){
            nums[(e+k)%nums.length]=tmp[e];
        }

    }
}
```

![image-20221231171745458](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221231171745458.png)





![image-20221207120454868](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221207120454868.png)

#### 4.存在重复元素

**示例 1：**

```
输入：nums = [1,2,3,1]
输出：true
```

**示例 2：**

```
输入：nums = [1,2,3,4]
输出：false
```

**示例 3：**

```
输入：nums = [1,1,1,3,3,4,3,2,4,2]
输出：true
```

```java
import java.util.Arrays;

public class test {
    public static void main(String[] args) {
        Solution solution=new Solution();
        int[]arr=new int[]{2,1,5,2,4,5,6};
        solution.containsDuplicate(arr);
    }
}

//先排序后比较
class Solution {
    public boolean containsDuplicate(int[] nums) {
        Arrays.sort(nums); //对数组进行排序
        for (int ind = 1; ind < nums.length; ind++) {
            if (nums[ind] == nums[ind - 1]) {
                return true;
            }
        }
        return false;
    }
}
```

![image-20221207120429116](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221207120429116.png)



```java
   public boolean containsDuplicate(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            //因为集合set中不能有重复的元素，如果有重复的
            //元素添加，就会添加失败
            if (!set.add(num))
                return true;
        }
        return false;
    }
```

![image-20230101190325013](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230101190325013.png)

#### 5.找出只出现一次的元素

给你一个 非空 整数数组 nums ，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

你必须设计并实现线性时间复杂度的算法来解决此问题，且该算法只使用常量额外空间。

示例 1 ：

```
输入：nums = [2,2,1]
输出：1
```

示例 2 ：

```
输入：nums = [4,1,2,1,2]
输出：4
```

示例 3 ：

```
输入：nums = [1]
输出：1
```

```java
public class test {
    public static void main(String[] args) {
        Solution solution=new Solution();
        int[]arr=new int[]{1,2,1,2,3};
        solution.singleNumber(arr);
    }
}

class Solution {
    public int singleNumber(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            if (!set.add(num)) {
                //如果添加失败，说明这个值
                //在集合Set中存在，我们要
                //把他给移除掉
                set.remove(num);
            }
        }
        //最终集合Set中只有一个元素，我们直接返回
        return (int) set.toArray()[0];
    }
}
```

![image-20221207120407628](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221207120407628.png)

```java
class Solution {
    public int singleNumber(int[] nums) {

        Arrays.sort(nums);

        if(nums.length==1){
            return nums[0];
        }
        
        for(int i=0;i<nums.length-1;i++){
            if(nums[i]==nums[i+1]){
                i++;
            }else{
                return nums[i];
            }
        }  
    return nums[nums.length-1];
    }
}
```

![image-20230102142455948](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230102142455948.png)

```java
public class test {
    public static void main(String[] args) {

        /**
         * 2 0010
         * 2 0010 0000
         * 3 0011 0011
         * 3 0011 0000
         * 4 0100 0100
         */

        System.out.println(2^2^3^3^4);

    }
}

```



```java
class Solution {
    public int singleNumber(int[] nums) {
        int reduce=0;
        for(int num:nums){
            reduce=reduce^num;
        }
        return reduce;
    }
}
```

![image-20230102164323089](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230102164323089.png)

#### 6.两个数组的交集

```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {

        int len1=nums1.length,len2=nums2.length;
        int len=len1<len2?len1:len2;
        int[] ans=new int[len];

        Arrays.sort(nums1);
        Arrays.sort(nums2);
        int i=0,j=0,k=0;
        while(i<len1&& j<len2){
            if(nums1[i]==nums2[j]){
                ans[k++]=nums1[i];
                i++;
                j++;
            }else if(nums1[i]<nums2[j]){
                i++;
            }else{
                j++;
            }
        }
    return Arrays.copyOfRange(ans,0,k);
    }
}
```

![image-20230102180134275](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230102180134275.png)

#### 7.移动零

```java
class Solution {
    public void moveZeroes(int[] nums) {

        int i=0;

        for(int j=0;j<nums.length;j++){
            if(nums[j]==0){
                i++;
            }else if(i!=0){
                nums[j-i]=nums[j];
                nums[j]=0;
            }
        }
    }
}
```

![image-20230103160453807](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230103160453807.png)

#### 8.加一

```java
class Solution {
    public int[] plusOne(int[] digits) {

        for(int i=digits.length-1;i>=0;i--){
            if(digits[i]!=9){
                digits[i]++;
                return digits;
            }else{
                digits[i]=0;
            }
        }
    
    int[] tmp=new int[digits.length+1];
    tmp[0]=1;
    return tmp;
    }
}
```

![image-20230103160516580](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230103160516580.png)



#### 9.两数之和

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        
        for(int i=0;i<nums.length;i++){
            for(int j=i+1;j<nums.length;j++){
                if(nums[i]+nums[j]==target){
                    return new int[]{i,j};
                }
            
            }
        }
        return new int[]{-1,-1};

    }
}
```

![image-20230108163856173](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230108163856173.png)

#### 10.有效的数独

```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        for(int i=0;i<9;i++){
            HashSet set1=new HashSet();
            HashSet set2=new HashSet();
            HashSet set3=new HashSet();
            for(int j=0;j<9;j++){
                if(board[i][j]!='.'&&!set1.add(board[i][j])){
                    return false;
                }
                if(board[j][i]!='.'&&!set2.add(board[j][i])){
                    return false;
                }
                int a=(i/3)*3+j/3;
                int b=(i%3)*3+j%3;
                if(board[a][b]!='.'&&!set3.add(board[a][b])){
                    return false;
                }

            }
        }
    return true;
    }
}
```

![image-20230108163921765](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230108163921765.png)

#### 11.旋转图像

```java
class Solution {
    public void rotate(int[][] matrix) {

        int length=matrix.length;
        for(int i=0;i<matrix.length/2;i++){
            int[] tmp=matrix[i];
            matrix[i]=matrix[length-1];
            matrix[length-1]=tmp;
            length--;
        }
        for(int i=0;i<matrix.length;i++){
            for(int j=i+1;j<matrix.length;j++){
                int tmp=matrix[i][j];
                matrix[i][j]=matrix[j][i];
                matrix[j][i]=tmp;
            }
        }

    }
}
```

## 第二章：字符串

#### 12.反转字符串

```java
class Solution {
    public void reverseString(char[] s) {
        int length=s.length;

        for(int i=0;i<s.length/2;i++){
            char tmp=s[i];
            s[i]=s[length-1];
            s[length-1]=tmp;
            length--;
        }
    }
}
```

![image-20230108164331921](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230108164331921.png)

#### 13.整数反转

```java
class Solution {
    public int reverse(int x) {
        long i=0;

        while(x!=0){
            i=i*10+x%10;
            x=x/10;
        }
    return (int)i==i?(int)i:0;

    }
}
```

![image-20230108164351617](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230108164351617.png)

#### 14.字符串中的第一个唯一字符

```java
class Solution {
    public int firstUniqChar(String s) {
       int[] count=new int[26];

       char[] chars=s.toCharArray();

       for(int i=0;i<s.length();i++){
           count[chars[i]-'a']++;
       }

       for(int i=0;i<s.length();i++){
           if(count[chars[i]-'a']==1){
               return i;
           }
       }
    return -1;

    }
}
```

![image-20230108164411823](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230108164411823.png)

#### 15.有效的字母异位词

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length())
            return false;
        int[] letterCount = new int[26];
        //统计字符串s中的每个字符的数量
        for (int i = 0; i < s.length(); i++)
            letterCount[s.charAt(i) - 'a']++;
        //减去字符串t中的每个字符的数量
        for (int i = 0; i < t.length(); i++)
            letterCount[t.charAt(i) - 'a']--;
        //如果数组letterCount的每个值都是0，返回true，否则返回false
        for (int count : letterCount)
            if (count != 0)
                return false;
        return true;
    }
}
```

![image-20230108164520690](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230108164520690.png)

#### 16.验证回文串

```java
public boolean isPalindrome(String s) {
        if (s == null || s.length() == 0)
            return true;
        s = s.toLowerCase();
        for (int i = 0, j = s.length() - 1; i < j; i++, j--) {
            while (i < j && !Character.isLetterOrDigit(s.charAt(i)))
                i++;
            while (i < j && !Character.isLetterOrDigit(s.charAt(j)))
                j--;
            if (s.charAt(i) != s.charAt(j))
                return false;
        }
        return true;
    }
```

![image-20230109191750498](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230109191750498.png)

#### 17.字符串转换整数 (atoi)

```java
class Solution {
    public int myAtoi(String s) {
            char[] chars = s.toCharArray();
            int length = chars.length;
            int index = 0;
            // 先去除空格
            while (index < length && chars[index] == ' '){
                index++;
            }
            // 再判断符号
            int sign =  1;
            if(chars[index] == '-' || chars[index] == '+'){
                if(chars[index] == '-'){
                    sign = -1;
                }
                index++;
            }

            int result = 0;
            int temp = 0;
            while (index < length){
                int num = chars[index] - '0';
                if(num > 9 || num < 0){
                    break;
                }
                temp = result;
                result = result * 10 + num;
                if(result/10 !=temp){
                    if(sign > 0){
                        return Integer.MAX_VALUE;
                    }else {
                        return Integer.MIN_VALUE;
                    }
                }
                index++;
            }
            return result*sign;
    }
}
```

![image-20230109191617435](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230109191617435.png)

#### 18.实现strStr()

```java 
public int strStr(String haystack, String needle) {
        if (needle.length() == 0)
            return 0;
        int i = 0;
        int j = 0;
        while (i < haystack.length() && j < needle.length()) {
            if (haystack.charAt(i) == needle.charAt(j)) {
                i++;
                j++;
            } else {
                //如果不匹配，就回退，从第一次匹配的下一个开始，
                i = i - j + 1;
                j = 0;
            }
            if (j == needle.length())
                return i - j;
        }
        return -1;
    }

```

![image-20230109191702275](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230109191702275.png)

#### 19.外观数列

```java
class Solution {
    public String countAndSay(int n) { //4

        String res="1"; //11 21 1211

        //确定描述的次数
        for(int i=2;i<=n;i++){
            int index=0;
            StringBuilder stringBuilder=new StringBuilder();//可变的字符串

            while(index<res.length()){
                int count=1;

                while(index<res.length()-1 && res.charAt(index)==res.charAt(index+1)){
                    index++;
                    count++;
                }

                stringBuilder.append(count);
                stringBuilder.append(res.charAt(index));
                index++;
            }

        res=stringBuilder.toString();

        }
    return res;
    }
}
```

![image-20230112210107918](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230112210107918.png)

#### 20.最长公共前缀

```java
class Solution {
      public String longestCommonPrefix(String[] strs) {
        //边界条件判断
        if (strs == null || strs.length == 0)
            return "";
        //默认第一个字符串是他们的公共前缀
        String pre = strs[0];
        int i = 1;
        while (i < strs.length) {
            //不断的截取
            while (strs[i].indexOf(pre) != 0)
                pre = pre.substring(0, pre.length() - 1);
            i++;
        }
        return pre;
    }
}
```

![image-20230112210055631](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230112210055631.png)

## 第三章：链表

#### 21.删除链表中的节点

![image-20230114163200191](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163200191.png)

```java
class Solution {
    public void deleteNode(ListNode node) {

        node.val=node.next.val;
        node.next=node.next.next;
        
    }
}
```



#### 22.删除链表的倒数第N个节点

![image-20230114163213836](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163213836.png)

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode pre = head;
    int last = length(head) - n;
    //如果last等于0表示删除的是头结点
    if (last == 0)
        return head.next;
    //这里首先要找到要删除链表的前一个结点
    for (int i = 0; i < last - 1; i++) {
        pre = pre.next;
    }
    //然后让前一个结点的next指向要删除节点的next
    pre.next = pre.next.next;
    return head;
}

//求链表的长度
private int length(ListNode head) {
    int len = 0;
    while (head != null) {
        len++;
        head = head.next;
    }
    return len;
}
```



#### 23.反转链表

![image-20230114163223754](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163223754.png)

思路一：表节点一个个入栈，当全部入栈完之后再一个个出栈，出栈的时候在把出栈的结点串成一个新的链表

```java
class Solution {
    public ListNode reverseList(ListNode head) {

        //将链表放入栈中
        Stack<ListNode> stack=new Stack<>();
        while(head!=null){
            stack.push(head);
            head=head.next;
        }

        ListNode tmp=new ListNode(0);
        ListNode tmp1=tmp;
        while(!stack.isEmpty()){
            tmp1.next=stack.pop();
            tmp1=tmp1.next;
        }
    tmp1.next=null;
    return tmp.next;
    }
}
```

**思路二：**

```java
public ListNode reverseList(ListNode head) {
    //新链表
    ListNode newHead = null;
    while (head != null) {
        ListNode temp = head.next;
        head.next = newHead;
        newHead = head;
        head = temp;
    }
    //返回新链表
    return newHead;
}
```

![image-20230114163452213](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163452213.png)

#### 24.合并两个有序链表

![image-20230114163235729](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163235729.png)

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {

        if(list1==null){
            return list2;
        }
        if(list2==null){
            return list1;
        }

        ListNode tmp=new ListNode(0);
        ListNode tmp1=tmp;
        while(list1!=null&&list2!=null){
            if(list1.val<=list2.val){
                tmp1.next=list1;
                list1=list1.next;
            }else{
                tmp1.next=list2;
                list2=list2.next;
            }
            tmp1=tmp1.next;
        }
    tmp1.next=list1==null?list2:list1;
    return tmp.next;

    }
}
```

![image-20230114163517518](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163517518.png)

#### 25.回文链表

![image-20230114163245995](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163245995.png)

**思路一：求链表长度找中间值**

```java
class Solution {
    public boolean isPalindrome(ListNode head) {

        //定义慢指针
        ListNode fast=head;

        //求出链表长度
        int length=length(head);

        //找到中点
        if(length%2==0){
            for (int i = 0; i < length/2; i++) {
                fast=fast.next;
            }
        }else {
            for (int i = 0; i < length / 2 + 1; i++) {
                fast = fast.next;

            }
        }

        //反转后半部分
        fast=reversal(fast);

        while (fast!=null&&head!=null){
            if(fast.val==head.val){
                fast=fast.next;
                head=head.next;
            }else {
                return false;
            }
        }

        return true;
    }

     public int length(ListNode head){
        int len=0;
        while(head!=null){
            head=head.next;
            len++;
        }
     return len;
    }

    public  ListNode reversal(ListNode node){

        ListNode tmp=null;
        while(node!=null){
            ListNode tmp1=node.next;
            node.next=tmp;
            tmp=node;
            node=tmp1;
        }
        return tmp;
    }
}
```

**思路二：快慢指针找中间值**

```java
public boolean isPalindrome(ListNode head) {
    ListNode fast = head, slow = head;
    //通过快慢指针找到中点
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;
    }
    //如果fast不为空，说明链表的长度是奇数个
    if (fast != null) {
        slow = slow.next;
    }
    //反转后半部分链表
    slow = reverse(slow);

    fast = head;
    while (slow != null) {
        //然后比较，判断节点值是否相等
        if (fast.val != slow.val)
            return false;
        fast = fast.next;
        slow = slow.next;
    }
    return true;
}

//反转链表
public ListNode reverse(ListNode head) {
    ListNode prev = null;
    while (head != null) {
        ListNode next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

![image-20230114174802376](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114174802376.png)

![image-20230114174813255](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114174813255.png)

#### 26.环形链表

![image-20230114163258167](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114163258167.png)

**思路一：快慢指针**

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
    
    //快慢两个指针
    ListNode slow = head;
    ListNode fast = head;
    while (fast != null && fast.next != null) {
    
        fast = fast.next.next;
        slow = slow.next;
        
        if (slow == fast)
            return true;
    }
    //否则就是没环
    return false;
    }
}
```

**2.思路二：存放集合**

```java
public boolean hasCycle(ListNode head) {
    Set<ListNode> set = new HashSet<>();
    while (head != null) {
        //如果重复出现说明有环
        if (set.contains(head))
            return true;
        //否则就把当前节点加入到集合中
        set.add(head);
        head = head.next;
    }
    return false;
}
```

## 第四章：树（递归）

#### 27.二叉树的最大深度

![image-20230116203955374](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230116203955374.png)

```
class Solution {
    public int maxDepth(TreeNode root) {

        if(root==null){
            return 0;
        }

        int left=maxDepth(root.left);
        int right=maxDepth(root.right);

        return 1+(left>=right?left:right);
    }
}
```

#### 28.验证二叉搜索树

![image-20230116204046130](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230116204046130.png)

```
class Solution {
    TreeNode pre;
    public boolean isValidBST(TreeNode root) {
        if(root==null){
            return true;
        }
        //二叉树的中序遍历
        
        if(!isValidBST(root.left)){
            return false;
        }

        if(pre!=null&&pre.val>=root.val){
            return false;
        }
        pre=root;

        if(!isValidBST(root.right)){
            return false;
        }

    return true;
    }
}
```

#### 29.对称二叉树

![image-20230116204101727](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230116204101727.png)

```
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root==null){
            return true;
        }

        return isSymmetricHelper(root.left,root.right);
    }

    public boolean isSymmetricHelper(TreeNode left,TreeNode right){
        if(left==null&&right==null){
            return true;
        }


        if(left==null||right==null||left.val!=right.val){
            return false;
        }
        
        return isSymmetricHelper(left.left,right.right)&& isSymmetricHelper(left.right,right.left);
    }
}
```

#### 30.二叉树的层序遍历

![image-20230117210458226](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230117210458226.png)

```java

class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root==null){
            return new ArrayList<>();
        }

        Queue<TreeNode> queue=new LinkedList<>();
        List<List<Integer>> res=new ArrayList<>();

        queue.add(root);
        while(!queue.isEmpty()){
            int queuelen=queue.size();
            List<Integer> subList=new ArrayList<>();

            for(int i=0;i<queuelen;i++){
                TreeNode node=queue.poll();
                subList.add(node.val);
                if(node.left!=null){
                    queue.add(node.left);
                }
                if(node.right!=null){
                    queue.add(node.right);
                }
            }
        res.add(subList);
        }
    return res;
    }
}
```



#### 31.将有序数组转换为二叉搜索树

![image-20230131181000352](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230131181000352.png)

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if(nums.length==0){
            return null;
        }

        return sortedArrayToBST(nums,0,nums.length-1);
    }

    public TreeNode sortedArrayToBST(int[] nums,int start,int end){
        if(start>end){
            return null;
        }
        int mid=(start+end)/2;
        TreeNode root=new TreeNode(nums[mid]);
        root.left=sortedArrayToBST(nums,start,mid-1);
        root.right=sortedArrayToBST(nums,mid+1,end);
        return root;
        }
}
```

#### L2-004 这是二叉搜索树吗？ (25 分)

#### L2-006 树的遍历 (25 分) 

#### L2-011 玩转二叉树 (25 分) 

#### L2-035 完全二叉树的层序遍历 (25 分) 

## 第五章：排序

>排序算法的稳定性：能保证排序前两个相等的数其在序列的前后位置顺序与排序后它们的前后位置顺序一致。从而在一定程度上减少了排序的时间

#### 1.冒泡排序（稳定）

两个for循环，比数组大小-1次，每次比较数组大小-1次，从数组的第一个数字开始与下一个比较，如果第一个比第二个大则交换，否则第二个跟第三个比较

![动图](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/v2-33a947c71ad62b254cab62e5364d2813_b.webp)

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] nums={1,2,7,9,5,42,3,6,7,8};


        for (int i = 0; i <nums.length-1; i++) {            //小于元素的个数
            for (int j = 0; j <nums.length-i-1 ; j++) {    //每一次排序完成后就少进行一次替换
                while(nums[j]>nums[j+1]){
                    int tmp=nums[j+1];
                    nums[j+1]=nums[j];
                    nums[j]=tmp;
                }
            }
        }

        System.out.println(Arrays.toString(nums));

    }
}

```



#### 2.选择排序（不稳定）

![动图](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/v2-1c7e20f306ddc02eb4e3a50fa7817ff4_b.webp)

将数组中的数据遍历，先拿第一个进行比较，看后面的有没有比这更小的，有的话交换，没有就第二个进行比，依次比较，一共需要比数组大小-1次

```java
import java.util.Arrays;

public class SelectSort {
    public static void main(String[] args) {
        //模拟数据
        int[] array = {52,63,14,59,68,35,8,67,45,99};
        selectSort(array);
        System.out.println(Arrays.toString(array));

    }

    /**
     * 首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。
     * 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。
     * 重复第二步，直到所有元素均排序完毕。
     *
     */
    public static void selectSort(int[] arr){
        for(int i = 0; i < arr.length-1; i++){ //最后一次排序完成两个值的排序

            int min = i; //先设置一个最小的值

            for(int j = i+1; j <arr.length ;j++){   //在里面找一个更小的值
                if(arr[j]<arr[min]){
                    min = j;  //如果有比上一个值更小就进行替换
                }
            }

            if(min!=i){
                int temp = arr[i];
                arr[i] = arr[min];
                arr[min] = temp;
            }
        }
    }
}

```



#### 3.插入排序（稳定）

![动图](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/v2-f87ad7d8ad54379dd81f02fcf9b91f49_b.webp)

将数组中的数据遍历，先拿第一个进行比较，看后面的有没有比这更小的，有的话交换，没有就第二个进行比，依次比较，一共需要比数组大小-1次

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        //以下情况需要进行改进
          int[] nums1={10,12,13,2,4,6,8};

//        for (int i = 1; i <nums1.length ; i++) {
//            for (int j = i; j>0 ; j--) {
//                if(nums1[j]<nums1[j-1]){
//                    int tmp=nums1[j-1];
//                    nums1[j-1]=nums1[j];
//                    nums1[j]=tmp;
//                }
//            }
//        }
//


        //对插入排序的改进情况
        for (int i = 1; i <nums1.length ; i++) {  //起始值必须为1
            int tmp=nums1[i]; //需要进行插入的那个值
            int j;
            for (j = i; j>0&&nums1[j-1]>tmp ; j--) {  //其实值为i，原因是要比较当前元素之前的内容
                nums1[j]=nums1[j-1];
            }
            nums1[j]=tmp;
        }
        System.out.println(Arrays.toString(nums1));
    }
}

```



#### 4.归并排序（稳定）

![动图](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/v2-cdda3f11c6efbc01577f5c29a9066772_b.webp)

归并排序是稳定排序，它也是一种十分高效的排序，能利用完全二叉树特性的排序一般性能都不会太差。java中Arrays.sort()采用了一种名为TimSort的排序算法，就是归并排序的优化版本。从上文的图中可看出，每次合并操作的平均时间复杂度为O(n)，而完全二叉树的深度为|log2n|。总的平均时间复杂度为O(nlogn)。而且，归并排序的最好，最坏，平均时间复杂度均为O(nlogn)。

```java
import java.util.Arrays;

public class Main {

    public static void main(String[] args) {
        int[] arr = {11,44,23,67,88,65,34,48,9,12};
        int[] tmp = new int[arr.length];    //新建一个临时数组存放
        mergeSort(arr,0,arr.length-1,tmp); //传入四个参数  先将数组进行切分
        System.out.println(Arrays.toString(arr));
    }

    //将数组进行拆分
    public static void mergeSort(int[] array,int left,int right,int[] tmp){
        //递归条件
        if (left>=right) {
            return;
        }
        int mid=(left+right)/2;
        mergeSort(array,left,mid,tmp); //对左边进行拆分
        mergeSort(array,mid+1,right, tmp);//对右面进行拆分
        //当所有的拆分完成后需要进行合并
        merge(array, left,mid,right ,tmp );
    }

    //对数组进行合并
    public static void merge(int[] array,int left,int mid,int right,int[] tmp){
        //先确定几个值
        int i=0;//用于tmp数组的索引
        int j=left; //左起始点
        int k=mid+1;//右起始点


        while(j<=mid&&k<=right){      //当其中有一个不满足条件的时候说明，后面的值就不用比较直接放入临时数组中
            if(array[j]<array[k]){    //找出当前值较小的那一个放入临时数组中
                tmp[i++]=array[j++];
            }else {
                tmp[i++]=array[k++];
            }
        }

        //如果左边有剩余就直接放入临时数组中
        while (j<=mid){
            tmp[i++]=array[j++];
        }

        //如果右边有剩余就直接放入临时数组中
        while (k<=right){
            tmp[i++]=array[k++];
        }

        //将临时数组中的值放到原数组中
        for (int t=0;t<i;t++){
            array[left+t]=tmp[t];
        }
    }
}

```



#### 5.快速排序（不稳定）

![动图](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/v2-c411339b79f92499dcb7b5f304c826f4_b.webp)

特点就是递归，通过一个中间数，将该数的左右分别排序好，先找一个**基准值**，一般为数组大小/2，左边找到一个比基准值大的数，右边找到一个比基准值小的数，然后进行交换，算完之后左边的都为比基准值小的，右边都为比基准值大的，但不能保证他们是有序的，所以还需要对左右生成的数据进行二次排序

```java
import java.util.Arrays;

public class QuickSort {
    public static void quickSort(int[] arrays,int left,int right){

        //设计递归结束条件
        if(left>right){
            return;
        }
        //定义新的左右指针
        int i=left;
        int j=right;

        //确定基准位置
        int tmp=arrays[i];
        //设置缓存
        int h;

        //让大于基准数的数放在右面
        //让小于基准数的数放在左面
        while(i<j){
            //让右指针先走
            //为什么先让右指针移动，只有先进行右指针的运动，才可以保证在相遇处的数字小于基准数
            while(i<j&&arrays[j]>=tmp){
                j--;
            }

            //左指针继续走
            while(i<j&&arrays[i]<=tmp){
                i++;
            }

            //符合条件的情况下
            if(i<j){
                h=arrays[i];
                arrays[i]=arrays[j];
                arrays[j]=h;
            }
        }

        //记得把基准数放回来
        arrays[left]=arrays[i];
        arrays[i]=tmp;


        //继续进行递归
        quickSort(arrays, left, j-1);
        quickSort(arrays, j+1, right);

    }


    public static void main(String[] args){
        int[] arr={2,4,5,21,23,43,54,65,7,687,543,1,12};
        quickSort(arr, 0, arr.length-1);
        System.out.println(Arrays.toString(arr));
    }
}
```



#### L2-009 抢红包 (25 分)

#### L2-015 互评成绩 (25 分) 

#### L2-019 悄悄关注 (25 分) 

#### L2-021 点赞狂魔 (25 分) 

#### L2-027 名人堂与代金券 (25 分) 

#### L2-034 口罩发放 (25 分) 



## 第六章：栈

#### L2-032 彩虹瓶 (25 分) 

#### L2-033 简单计算器 (25 分) 

## 第七章：队列

## 第八章：图

## 第十章：搜索算法

#### L2-016 愿天下有情人都是失散多年的兄妹 (25 分) 

#### L2-020 功夫传人 (25 分) 

#### L2-030 冰岛人 (25 分) 

#### L2-031 深入虎穴 (25 分) 

## 第十一章：查找算法

#### 1.二分查找

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        //定义左指针和右指针
        int left=0,right=nums.length-1;

        while(left<=right){
            int mid=(left+right)/2;
            if(nums[mid]==target){
                return mid;
            }else if(nums[mid]<target){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }
        return left;

    }
}
```



## 天梯赛（基础题目）

#### L1-003 个位数统计（数组字符串）

![image-20230114174839683](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114174839683.png)

解法一：

```java
import java.util.Scanner;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        String s=scanner.next();
        int[] count=new int[10];

        for(int i=0;i<s.length();i++){
            count[s.charAt(i)-'0']++;
        }
        for (int i = 0; i <count.length ; i++) {
            if(count[i]!=0){
                System.out.printf("%d:%d\n",i,count[i]);
            }
        }

    }
}

```

解法二：

```java
import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
      Scanner scanner=new Scanner(System.in);
      String s= scanner.next();
      
      //字符串数组转化为int数组
      String str[]=s.split("");

      int array[]=new int[str.length];
      for (int i=0;i<str.length;i++){
          array[i]=Integer.parseInt(str[i]);
      }
      Arrays.sort(array);

      int index=0;
      while(index<array.length){
          int count=1;
          while (index<array.length-1 && array[index]==array[index+1]){
              index++;
              count++;
          }
          System.out.println(array[index]+":"+count);
          index++;
      }
    }
}
```



![image-20230114174918001](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114174918001.png)

```java
package 天梯赛.L1_05_考试座位号;


import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);

        int num=scanner.nextInt();

        long[][] num1=new long[num][3];
        for (int i=0;i<num;i++){
            for (int j = 0; j < 3; j++) {
                num1[i][j]=scanner.nextLong();
            }
        }

        int m=scanner.nextInt();
        for (int index = 0; index <m; index++) {
            int tmp=scanner.nextInt();
            for (int i=0;i<num;i++){
                if(num1[i][1]==tmp){
                    System.out.println(num1[i][0]+" "+num1[i][2]);
                }
            }
        }
    }
}

```



#### L1-006 连续因子（求因子）

![image-20230131165000451](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230131165000451.png)

```
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
      Scanner scanner=new Scanner(System.in);

      long n=scanner.nextInt();
      long sum=0;
      long len=0;
      long start=0;
      for (long i=2;i<Math.sqrt(n);i++){
          sum=1;
          for (long j = i; sum <=n; j++) {//i是起始值 j是终止值
              sum=sum*j;
              if(n%sum==0&&j-i+1>len){
                  start=i;
                  len=j-i+1;
              }
          }
      }

      if(start==0){
          len=1;
          start=n;
      }

      System.out.println(len);
      for (int k = 0; k < len-1; k++) {
          System.out.print(start+k+"*");
      }

      System.out.println(start+len-1);
    }
}

```

0131172414330](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230131172414330.pn

#### L1-008 求整数段和

![image-20230131174044609](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230131174044609.png)

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int a=scanner.nextInt();
        int b=scanner.nextInt();
        int count=0;
        int sum=0;
        for (int i = a; i <b+1; i++) {
            if(count<5){
                System.out.printf("%5d",i);
                count++;
                sum=sum+i;
            }
            else {
                System.out.println();
                System.out.printf("%5d",i);
                count=1;
                sum=sum+i;
            }
        }
        System.out.println();
        System.out.println("Sum = "+sum);
    }
}

```

#### L1-010 比较大小

![image-20230131180819483](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230131180819483.png)

```Java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int a=scanner.nextInt();
        int b=scanner.nextInt();
        int c=scanner.nextInt();


        int[] array=new int[3];
        array[0]=a;
        array[1]=b;
        array[2]=c;

        for (int i = 0; i < array.length-1; i++) {
            for (int j = 0; j < array.length-i-1; j++) {
               if(array[j]>array[j+1]){
                   int tmp=array[j];
                   array[j]=array[j+1];
                   array[j+1]=tmp;
               }
            }
        }

        for (int i = 0; i < array.length-1; i++) {
            System.out.print(array[i]+"->");
        }
        System.out.println(array[array.length-1]);
    }
}
```

#### L1-011 A-B 

![image-20230202165249022](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230202165249022.png)

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        String s=in.readLine();
        String s2=in.readLine();

        char[] chars=new char[s.length()];
        for (int i = 0; i < s.length(); i++) {
            chars[i]=s.charAt(i);
        }

        boolean c=true;

        for (int j = 0; j < s.length(); j++) {
            c=true;
            for (int i = 0; i < s2.length(); i++) {
                if(chars[j]==s2.charAt(i)){
                    c=false;
                }
            }
            if(c!=false){
                System.out.print(chars[j]);
            }
        }
    }
}

```

第二种方法：

```java
import java.util.Scanner;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Main1 {

    public static void main(String[] args) throws IOException {

        BufferedReader in=new BufferedReader(new InputStreamReader(System.in));

        String s=in.readLine(); //第一行字母
        String b=in.readLine(); //第二行字母
        char[] bs=b.toCharArray(); //将第二行字母放到字符数组中
        String news=s;   //对第一行字母重新赋值

        for (int i = 0; i < b.length(); i++) {
            String c = String.valueOf(bs[i]); //char转化为string
            news=news.replace(c, "");
        }
        System.out.println(news);

    }

}
```

#### L1-013 计算阶乘和

![image-20230201203241773](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201203241773.png)

```Java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        long b=scanner.nextLong();

        long c=b;
        long sum2=0;
        for (int i = 0; i <b; i++) {
            sum2=sum2+jie(c);
            c--;
        }
        System.out.println(sum2);


    }
    public static long jie(long a){

        long sum=a;
        while (a!=1){
            sum=sum*(a-1);
            a--;
        }
        return sum;
    }
}

```



#### L1-016 查验身份证

![image-20230201194544013](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201194544013.png)

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int a=scanner.nextInt();

        String[] chars=new String[a];
        int[] arr=new int[a];
        char[] x = { '1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2' };
        int[] qz = { 7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2 };

        for (int i = 0; i < a; i++) {
            chars[i]=scanner.next();
        }

        int b=0;
        for (int i = 0; i <a ; i++) {
            for (int j = 0; j <17 ; j++) {
                if(chars[i].charAt(j)-'0'>9){
                    arr[i]++;
                    b++;
                }
            }
        }


        int d=0;
        for (int i = 0; i < a; i++) {
            d=0;
            for (int j = 0; j < 17; j++) {
                d=d+(chars[i].charAt(j)-'0')* qz[j];
            }
            d=d%11;
            if(x[d]!=chars[i].charAt(17)){
                arr[i]++;
                b++;
            }
        }

        for (int j = 0; j < arr.length; j++) {
            if(arr[j]!=0){
                System.out.println(chars[j]);
            }
        }

        if (b == 0) {
            System.out.println("All passed");
        }
    }
}

```

方法二：

```java
import java.util.Scanner;

public class Main1 {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int a=scanner.nextInt();

        String[] chars=new String[a];

        char[] x = { '1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2' };
        int[] qz = { 7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2 };

        for (int i = 0; i < a; i++) {
            chars[i]=scanner.next();
        }

        int b=0;
        int d=0;
        for (int i = 0; i < a; i++) {
            d=0;
            for (int j = 0; j < 17; j++) {
                d=d+(chars[i].charAt(j)-'0')* qz[j];
            }
            d=d%11;
            if(x[d]!=chars[i].charAt(17)){
                System.out.println(chars[i]);
                b++;
            }
        }

        if (b == 0) {
            System.out.println("All passed");
        }
    }
}


```

#### L1-017 到底有多二

![image-20230131163911102](C:\Users\90632\AppData\Roaming\Typora\typora-user-images\image-20230131163911102.png)

![image-20230114174934112](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114174934112.png)

```java
import java.util.*;
public class Main{
    public static void main(String[] args){

        Scanner scanner=new Scanner(System.in);
        String num =scanner.next();
        double mul=100;
        int length=0;
        int count=0;


        //判断正负
        if(num.charAt(0)=='-'){
            mul*=1.5;
            length--;
        }

        //判断是否2的倍数
        if((num.charAt(num.length()-1)-'0')%2==0){
            mul*=2.0;
        }
        //判断2的个数
        for(int i=0;i<num.length();i++){
            if(num.charAt(i)=='2'){
                count++;
            }
        }

        //输出
        System.out.printf("%.2f%%",count*mul/(num.length()+length));
    }
}
```

#### L1-019 谁先倒

![image-20230114212334189](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114212334189.png)

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args){
       Scanner scanner=new Scanner(System.in);
       int num1=scanner.nextInt(); //甲的酒量
       int num2=scanner.nextInt(); //乙的酒量

        int num3=0;
        int num4=0;

        int num5=scanner.nextInt();
        for (int i=0;i<num5;i++){
            int[] array=new int[4];
            for (int j = 0; j <array.length ; j++) {
                array[j]=scanner.nextInt();
            }

            if(array[0]+array[2]==array[1]&&array[0]+array[2]!=array[3]){
                num1--;
                num3++;
            }

            if(array[0]+array[2]==array[3]&&array[0]+array[2]!=array[1]){
                num2--;
                num4++;
            }

            if(num1<0){
                System.out.println("A");
                System.out.println(num4);
                break;
            }

            if(num2<0){
                System.out.println("B");
                System.out.println(num3);
                break;
            }
        }
    }
}
```

#### L1-020 帅到没朋友

![image-20230205112312611](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230205112312611.png)

```java
import java.util.ArrayList;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int num=scanner.nextInt();
        ArrayList<Integer> arrays1=new ArrayList<>();
        //接收朋友圈
        for (int i = 0; i<num; i++) {
            int num1=scanner.nextInt();
            if(num1!=1){
                for (int j = 0; j <num1; j++) {
                    int h=scanner.nextInt();
                    if(!arrays1.contains(h)){
                        arrays1.add(h);
                    }
                }
            }else{
                int p=scanner.nextInt();
            }
        }

        //接收要判断的内容并进行去重
        int num2=scanner.nextInt();
        ArrayList<Integer> arrays2=new ArrayList<>();
        for (int i = 0; i <num2 ; i++) {
            int z=scanner.nextInt();
            if(!arrays2.contains(z)){
                arrays2.add(z);
            }
        }

        //用于输出时作出判断
        for (int i = 0; i <arrays1.size() ; i++) {
                if(arrays2.contains(arrays1.get(i))){
                    arrays2.remove(arrays1.get(i));
                }
        }

        if (arrays2.size()== 0) {
            System.out.print("No one is handsome");
        }else{
            for (int i = 0; i < arrays2.size()-1; i++) {
                System.out.printf("%05d",arrays2.get(i));
                System.out.print(" ");
            }
            System.out.printf("%05d",arrays2.get(arrays2.size()-1));
        }
    }
}
```

#### L1-023 输出GPLT

![image-20230205112413008](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230205112413008.png)

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Locale;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        String s=in.readLine();

        s=s.toUpperCase(Locale.ROOT);
        System.out.println(s);

        int[] array=new int[4];

        for (int i = 0; i < s.length(); i++) {
            if(s.charAt(i)=='G'){
                array[0]++;
            }else if(s.charAt(i)=='P'){
                array[1]++;
            }else if(s.charAt(i)=='L'){
                array[2]++;
            }else if(s.charAt(i)=='T'){
                array[3]++;
            }
        }

        for (int i = 0; i < 10000000; i++) {
            if(array[0]!=0){
                System.out.print("G");
                array[0]--;
            }
            if (array[1]!=0){
                System.out.print("P");
                array[1]--;
            }
            if (array[2] != 0) {
                System.out.print("L");
                array[2]--;
            }
             if (array[3] != 0) {
                System.out.print("T");
                array[3]--;
            }
        }
    }
}
```

#### L1-025 正整数A+B

![image-20230205112458447](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230205112458447.png)

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        String s=scanner.nextLine();
        s = s.trim();
        String[] num=s.split("\\s+");
        boolean a=true;
        boolean d=true;
        int num1=0;
        int num2=0;

        if(num.length>2){
            d=false;
        }
        System.out.println(num.length);

        try {
            num1=Integer.valueOf(num[0]);
            if(num1<1||num1>1000){
                a=false;
            }

        }catch (Exception e){
           a=false;
        }


        try {
            num2=Integer.valueOf(num[1]);
            if(num2<1 || num2>1000){
                d=false;
            }

        }catch (Exception e){
            d=false;
        }

        if(a!=false&&d!=false){
            int num3=num1+num2;
            System.out.println(num1+" + "+num2+" = "+num3);
        }



        if(a==false&&d!=false){
            System.out.println("?"+" + "+num2+" = "+"?");
        }

        if(a!=false&&d==false){
            System.out.println(num1+" + "+"?"+" = "+"?");
        }

        if(a==false&&d==false){
            System.out.println("?"+" + "+"?"+" = "+"?");
        }
    }
}
```

#### L1-027 出租

![image-20230205112545695](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230205112545695.png)

```
import java.util.HashSet;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        String s=scanner.next();

        HashSet<Integer> set=new HashSet<>();

        //将数字放入集合中
        for (int i = 0; i < s.length(); i++){
            int b=s.charAt(i)-'0';
            set.add(b);
        }

        //逆序输出
        int[] array1=new int[set.size()];
        int c=set.size()-1;
        for (int a:set) {
            array1[c--]=a;
        }

        //输出号码中包含的数字
        System.out.print("int[] arr = new int[]{");
        for (int i = 0; i < set.size()-1 ; i++) {
            System.out.print(array1[i]+",");
        }
        System.out.println(array1[array1.length-1]+"};");

        //输出对应号码的索引
        System.out.print("int[] index = new int[]{");
        for (int i = 0; i <s.length() ; i++) {
            for (int j = 0; j < array1.length; j++) {

                int z=s.charAt(i)-'0';

                if(z==array1[j] && i!=s.length()-1){
                    System.out.print(j+",");
                }
                if(z==array1[j]&&i==s.length()-1){
                    System.out.print(j+"};");
                }
            }
        }
    }
}

```



#### L1-028 判断素数

![image-20230205112704080](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230205112704080.png)

```java
import java.util.Scanner;
//素数，只有一和他本身可以除尽的数字
public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int num1=scanner.nextInt();


        boolean a=true;
        for (int i = 0; i <num1 ; i++) {
            a=true;
            int num=scanner.nextInt();
            if (num == 1||num==0) {
                System.out.println("No");
                a=false;
            }



            for (int j = 2; j <=Math.sqrt(num); j++) {
                if(num%j==0){
                    System.out.println("No");
                    a=false;
                    break;
                }
            }

            if(a==true){
                System.out.println("Yes");
            }
        }
    }
}
```



#### L1-030 一帮一

![image-20230205112752864](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230205112752864.png)

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);

        int num=scanner.nextInt();

        String[] string1=new String[num/2];
        String[] string2=new String[num/2];
        String s=scanner.nextLine();

        for (int i = 0; i < num/2; i++) {
            string1[i]=scanner.nextLine();
        }

        for (int i = 0; i < num/2; i++) {
            string2[i]=scanner.nextLine();
        }

        for (int i = 0; i <num/2; i++) {
            for (int j = num/2-1; j >=0; j--) {
                if(!string1[i].substring(0,1).equals(string2[j].substring(0,1)) && string2[j]!="NO"){

                    System.out.println(string1[i].substring(2)+":"+string2[j].substring(2));

                    string2[j]="NO";

                    break;
                }
            }
        }
    }
}

```

#### L1-033 出生年

![image-20230213160747421](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230213160747421.png)

```
import java.util.HashSet;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        String s1 =scanner.next(); //出生年份
        int num2 =scanner.nextInt(); //几位数字不相同
        int index=0;
        String s =s1;

        HashSet<Integer> set=new HashSet<>();

        while(true){
            if(s.length()<4){
                s="0"+s;
            }

            int long1=s.length();
            for (int i = 0; i <long1 ; i++) {
                set.add(s.charAt(i)-'0');
            }

            index=long1-(long1-set.size()); //多少个不重复的

            if (index == num2) {
                System.out.print(Integer.valueOf(s)-Integer.valueOf(s1)+" ");
                System.out.printf("%04d",Integer.valueOf(s));
                break;
            }

            Integer num3=Integer.valueOf(s)+1;
            s=num3.toString();
            set.clear();
        }
    }
}

```

另外一种方法：

```
import java.util.Scanner;

public class Main2 {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int y = sc.nextInt();
        int n = sc.nextInt();
        int x = 0;
        int year =0;
        int b[] = new int [4];
        for(int i=y;;i++) {
            int count=1;
            int m=0;
            b[0] = i/1000;//千位
            b[1] = i%1000/100;//百位
            b[2] = i%100/10;//十位
            b[3] = i%10;//个位
            if (b[0]!=b[1] && b[0]!=b[2] && b[0]!=b[3]) count++;
            if (b[1]!=b[2] && b[1]!=b[3]) count++;
            if (b[2]!=b[3]) count++;
            if (count==n){
                year = i;
                break;//找到满足的数字后跳出循环输出结果
            }
        }
        System.out.print(year-y+" ");
        System.out.printf("%04d",year);
    }
}
```



#### L1-034 点赞

![image-20230213160934877](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230213160934877.png)

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);

        int num=scanner.nextInt();

        int[] arrays=new int[1001];
        int max=0;
        int index=0;
        for (int i = 0; i <num ; i++) {
            int a=scanner.nextInt();
            for (int j = 0; j < a; j++) {

                int b=scanner.nextInt();

                arrays[b]++;

                if(arrays[b]>max){
                    index=b;
                    max=arrays[b];
                }else if (arrays[b] == max && b>index) {
                    index=b;
                }
            }
        }

        System.out.println(index+" "+max);
    }
}

```



#### L1-039 古风排版

![image-20230213161018610](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230213161018610.png)

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int num=scanner.nextInt();
        String s0=scanner.nextLine();
        String s=scanner.nextLine();
        int num1=0;

        if (s.length()%num==0) {
            num1=s.length()/num;
        }
        if (s.length()%num!=0) {
            num1=s.length()/num+1;
        }

        int index=0;

        String[][] chars=new String[num][num1];

        for (int j = num1-1; j >=0 ; j--) {
            for (int i = 0; i < num; i++) {
                chars[i][j]=" ";
                if(index<s.length()){
                    chars[i][j]= String.valueOf(s.charAt(index));
                    index++;
                }
            }
        }

        for (int i = 0; i <num ; i++) {
            if(i!=0){
                System.out.println();
            }
            for (int j = 0; j <num1 ; j++) {
                System.out.print(chars[i][j]);
            }
        }

    }
}
```



#### L1-043 阅览室

![image-20230213161126548](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230213161126548.png)

![image-20230213161139446](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230213161139446.png)

```java
import java.util.ArrayList;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);

        int num=scanner.nextInt();
        String s1=scanner.nextLine();
        int index=0;


        while (index<num){
            ArrayList<String> strings=new ArrayList<>();
            int count=0; //记录次数
            int time=0;  //记录时间
            while(true){
                String s=scanner.nextLine();
                strings.add(s);
                if (s.substring(0,1).equals("0")) {
                    break;
                }

            }

            for (int i = 0; i < strings.size(); i++) {
                for (int j = i+1; j < strings.size()-1; j++) {
                    String book1=strings.get(i).substring(0,1);
                    String book2=strings.get(j).substring(0,1);
                    String tai1=strings.get(i).substring(2,3);
                    String tai2=strings.get(j).substring(2,3);

                    //计算时间和次数
                    if(book1.contains(book2) &&!tai1.equals(tai2)){
                        count++;
                        int hour1=Integer.valueOf(strings.get(i).substring(4,6));
                        int min1=Integer.valueOf(strings.get(i).substring(7,9));

                        int hour2=Integer.valueOf(strings.get(j).substring(4,6));
                        int min2=Integer.valueOf(strings.get(j).substring(7,9));

                        time=time+((hour2*60+min2)-(hour1*60+min1));
                    }
                }
            }

            if (count!=0){
                if (time%count==0){
                    System.out.println(count+" "+time/count);
                }else if(time%count!=0){
                    int time1=(time/count)+1;
                    System.out.println(count+" "+time1);
                }
            } else if(count==0){
                System.out.println(0+" "+0);
            }
            index++;
        }
    }
}

```



#### L1-046 整除光棍

![image-20230213161217167](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230213161217167.png)



```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Main1 {

    public static void main(String[] args) throws NumberFormatException, IOException {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        int x = Integer.parseInt(in.readLine());
        int s = 0, cnt = 0;
        // 这一步的目的是避免s<x时s/x的结果为0
        while (s < x) {
            s = s * 10 + 1;
            cnt++;
        }
        int i;
        // 逐渐增加光棍的位数，直到可以整除x为止
        for (i = cnt;; i++) {
            System.out.print(s / x);
            if (s % x == 0) {
                break;
            }
            // 模拟除法取余数
            s = s % x;
            // 除数是由"1"组成的数，因此借位下来的也是"1"，即余数连接“1”的形式
            s = s * 10 + 1;
        }
        System.out.println(" " + i);
    }
}
```



## 蓝桥杯

### 2021

#### L01_ASC（打卡题）

```
public class L01_ASC {
    public static void main(String[] args) {
        System.out.println("76");
    }
}
```

#### L02_卡片（数组模拟）

```
import java.util.Scanner;

public class L02_卡片 {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);

        int[] array=new int[10];

        //将所有值赋值为2021
        for (int i = 0; i < array.length; i++) {
            array[i]=2021;
        }

        String s="1";
        boolean a=false;

        while(true){
            for (int i = 0; i < s.length(); i++) {
                int num=s.charAt(i)-'0';
                if (array[num] != 0) {
                    array[num]--;
                }else{
                    System.out.println(Integer.valueOf(s)-1);
                    a=true;
                    break;
                }
            }

            if (a == true) {
                break;
            }

            Integer num1=Integer.valueOf(s);
            num1=num1+1;
            s=num1.toString();
        }
        scan.close();
    }
}
```

#### L03_直线（模拟）

```
import java.util.HashSet;

public class L03_直线 {
    public static void main(String[] args) {

        String[] strings=new String[420];
        //1.创建HashSet存放斜率
        HashSet<String> hashSet=new HashSet<>();

        //2.将所有点放到数组中
        int k=0;
        for (int i = 0; i <20; i++) {
            for (int j = 0; j <21 ; j++) {
                    strings[k++]=i+":"+j;
            }
        }

        //3.进行斜率计算
        //System.out.println(strings[]);
        for (int i = 0; i <420; i++) {
            String[] chars1=strings[i].split(":");
            int x1=Integer.valueOf(chars1[0]);
            int y1=Integer.valueOf(chars1[1]);

            for (int j = i+1; j <420; j++) {
                String[] chars2=strings[j].split(":");
                int x2=Integer.valueOf(chars2[0]);
                int y2=Integer.valueOf(chars2[1]);

                //斜率计算
                int y3=y1-y2;
                int x3=x1-x2;

                //同一条竖线的情况存入
                if(x3==0){
                    hashSet.add("x="+x1);
                    continue;
                }

                //求斜率
                int yueshu=yue(y3,x3);
                String K=y3/yueshu+"/"+x3/yueshu;

                //y=kx+b; b=y-kx; 对B进行处理
                int jieju=y1*x3-x1*y3;
                int jiejuyue=yue(jieju,x3);
                String B=jieju/jiejuyue+"/"+x3/jiejuyue;
                hashSet.add(K+":"+B);

            }

        }
        System.out.println(hashSet.size());
    }

    //辗转相除求最大公约数
    static int yue(int x4,int y4){
        int tmp=0;
        while(y4!=0){
            tmp=x4%y4;
            x4=y4;
            y4=tmp;
        }
        return x4;
    }
}
```

#### L04_货物摆放（求因数）

```java
import java.util.ArrayList;

public class L04_货物摆放 {
    public static void main(String[] args) {

        long l1=2021041820210418l;
        int count=0;
        ArrayList<Long> arrayList=new ArrayList<>();

        for (long i = 1; i < Math.sqrt(l1); i++) {
            if(l1%i==0){
                arrayList.add(i);
                if(l1/i!=i){
                    arrayList.add(l1/i);
                }
            }
        }
        for (long a:arrayList) {
            for (long b:arrayList) {
                for (long c:arrayList) {
                    if (a*b*c ==l1) {
                        count++;
                    }
                }
            }
        }
        System.out.println(count);
    }
}
```

#### L05路径——图_动规

#### L06_时间显示（时间转化求余）

```
import java.util.Scanner;

public class L06_时间显示 {
    public static void main(String[] args) {
        //1秒等于1000毫秒
        Scanner scan = new Scanner(System.in);

        long s= scan.nextLong();

        long miao=s/1000;//转化为秒
        long miao1=miao%60;

        long fen=miao/60;//转化为分
        long fen1=fen%60;

        long shi=fen/60;//转化为时
        long shi1=shi%24;

        System.out.printf("%02d", shi1);
        System.out.print(":");
        System.out.printf("%02d", fen1);
        System.out.print(":");
        System.out.printf("%02d", miao1);
    }
}
```

#### L07_最少砝码（数学规律）

```
import java.util.Scanner;
//找规律
public class L07_最少砝码 {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        long l1=scan.nextLong();
        int ce=1;
        int num=1;
        while(ce<l1){
            ce=ce*3+1;
            num++;
        }
        System.out.println(num);
        scan.close();
    }
}
```

#### L08_杨辉三角（40%）（超大值）

```
public class L08_杨辉三角 {
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        int num=scan.nextInt();

        if(num==1){
            System.out.println(1);
        }else{
            //实现一个杨辉三角
            int[][] array=new int[num+1][num+1];

            //完成1的填充
            for (int i = 0; i < array.length; i++) {
                array[i][0]=1;
                array[i][i]=1;
            }


            int count=3;
            boolean tf=true;
            //完成其他值的填充 从第三行的第二个值进行入手
            for (int i = 2; i < array.length ; i++) { //控制行
                count++;
                for (int j = 1; j < i ; j++) { //控制列
                    array[i][j]=array[i-1][j-1]+array[i-1][j];
                    count++;
                    if(array[i][j]==num){
                        System.out.println(count);
                        return;
                    }
                }

                count++;
            }
        }
        scan.close();
    }
}
```

### 2022

#### L01_星期计算（打卡题）

#### L02_山（大数运算）

#### L03_字符统计（数组模拟）

#### L04_最少刷题数（数学规律）

#### L05_求阶乘（数学规律，大数）

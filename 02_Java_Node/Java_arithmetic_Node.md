[TOC]

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

## 第四章：树

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

## 第五章：排序

#### 1.冒泡排序

![在这里插入图片描述](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/20201209193006204.gif)

```java
public class MaoPaoPX {

	public static void main(String[] args) {
		int []arr= {3,2,8,4,7};//创建数组
		System.out.println("排序前");
		showArr(arr);//打印显示排序前
		//循环实现冒泡排序
		for(int i=0;i<arr.length-1;i++) {
			for(int j=0;j<arr.length-i-1;j++) {
				if(arr[j]>arr[j+1]) {
					int temp=arr[j];
					arr[j]=arr[j+1];
					arr[j+1]=temp;
				}
				
			}
		}
		System.out.println("排序后");
		showArr(arr);
	}
	//打印方法
	private static void showArr(int []arr) {
		//增强for循环打印
		for(int a:arr) {
			System.out.print(a+"\t");
		
		}
		System.out.println();
		
	}

}
```



## 天梯赛

#### L1-003 个位数统计

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



#### L1-004 计算摄氏温度

![image-20230114174903307](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114174903307.png)

```java
package 天梯赛.L1_04_计算摄氏温度;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int num=scanner.nextInt();

        System.out.println("Celsius= "+5*(num-32)/9);

    }
}

```



#### L1-005 考试座位号

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



#### L1-006 连续因子

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

#### L1-007 念数字

![image-20230131172414330](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230131172414330.png)

```
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        String s=scanner.next();


        int[] chars=new int[s.length()];

        for (int i = 0; i < chars.length; i++) {
            chars[i]=s.charAt(i)-'0';
        }

        for (int j = 0; j < chars.length; j++) {

            if (j==chars.length-1){
                switch (chars[j]){
                    case -3:
                        System.out.print("fu");
                        break;
                    case 0:
                        System.out.print("ling");
                        break;
                    case 1:
                        System.out.print("yi");
                        break;
                    case 2:
                        System.out.print("er");
                        break;
                    case 3:
                        System.out.print("san");
                        break;
                    case 4:
                        System.out.print("si");
                        break;
                    case 5:
                        System.out.print("wu");
                        break;
                    case 6:
                        System.out.print("liu");
                        break;
                    case 7:
                        System.out.print("qi");
                        break;
                    case 8:
                        System.out.print("ba");
                        break;
                    case 9:
                        System.out.print("jiu");
                        break;
                }
            }
            if(j!=chars.length-1){
                switch (chars[j]){
                    case -3:
                        System.out.print("fu ");
                        break;
                    case 0:
                        System.out.print("ling ");
                        break;
                    case 1:
                        System.out.print("yi ");
                        break;
                    case 2:
                        System.out.print("er ");
                        break;
                    case 3:
                        System.out.print("san ");
                        break;
                    case 4:
                        System.out.print("si ");
                        break;
                    case 5:
                        System.out.print("wu ");
                        break;
                    case 6:
                        System.out.print("liu ");
                        break;
                    case 7:
                        System.out.print("qi ");
                        break;
                    case 8:
                        System.out.print("ba ");
                        break;
                    case 9:
                        System.out.print("jiu ");
                        break;
                }
            }

        }
    }
}
```

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

#### L1-012 计算指数

![image-20230201164827519](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201164827519.png)

```Java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int s=scanner.nextInt();

        int a=1;
        for (int i = 0; i < s; i++) {
            a=a*2;
        }
        System.out.println("2"+"^"+s+" = "+a);
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



#### L1-015 跟奥巴马一起画方块

![image-20230201170046840](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201170046840.png)

```Java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        double a=scanner.nextInt();
        String s=scanner.next();



        for (int i = 0; i <Math.round(a/2) ; i++) {
            if (i != 0) {
                System.out.println();
            }

            for (int j = 0; j <a ; j++) {
                System.out.print(s);
            }
        }

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



#### L1-018 大笨钟

![image-20230114175009637](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114175009637.png)

```java
import java.util.Scanner;
public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        String s=scanner.next();

        String[] s1=s.split(":");

        int hour=Integer.parseInt(s1[0]); //字符串转数字
        int min=Integer.parseInt(s1[1]);

        if(hour>=0&&hour<12){
            System.out.print("Only "+s+".  Too early to Dang.");
        }

        if(hour==12&&min==0){
            System.out.print("Only "+s+".  Too early to Dang.");
        }

        if(hour==12&&min>0){
            System.out.print("Dang");
        }

        if(hour>=13){
            if(min==0){
                for (int i = 0; i < hour-12; i++) {
                    System.out.print("Dang");
                }
            }else {
                for (int i = 0; i < hour-11; i++) {
                    System.out.print("Dang");
                }
            }
        }
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

#### L1-074 两小时学完C语言

![image-20230114175031444](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230114175031444.png)

```java
import java.io.*;
import java.util.*;

public class Main
{
    public static void main(String[] args)
    {   
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int k = sc.nextInt();
        int m = sc.nextInt();

        System.out.println(n - k * m);     
    }
}
```

## 基础知识

![image-20230119104608992](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230119104608992.png)

## 参考

https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/px2ds5/



## Day1 栈与队列（简单）



### 1.  用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

```java
// 耗费时间36ms
public class CQueue {
    private LinkedList<Integer> aList;
    private LinkedList<Integer> bList;

    public CQueue() {
        aList = new LinkedList<>();
        bList = new LinkedList<>();
    }

    public void appendTail(int value) {
        aList.add(value);
    }

    public int deleteHead() {
        if (!bList.isEmpty()) {
            return bList.removeLast();
        }
        if (aList.isEmpty()) {
            return -1;
        }
        while (!aList.isEmpty()) {
            bList.add(aList.removeLast());
        }
        return bList.removeLast();
    }

    public static void main(String[] args) {
        CQueue obj = new CQueue();
        obj.appendTail(3);
        int param_2 = obj.deleteHead();
        System.out.println(param_2);  // 3
        int param_3 = obj.deleteHead();
        System.out.println(param_3);  // -1
    }
}
```

```java
// 耗费时间58ms
public class CQueue {
    private Stack<Integer> aStack;
    private Stack<Integer> bStack;

    public CQueue() {
        aStack = new Stack<>();
        bStack = new Stack<>();
    }

    public void appendTail(int value) {
        aStack.push(value);
    }

    public int deleteHead() {
        if (!bStack.isEmpty()) {
            return bStack.pop();
        }
        if (aStack.isEmpty()) {
            return -1;
        }
        while (!aStack.isEmpty()) {
            bStack.add(aStack.pop());
        }
        return bStack.pop();
    }

    public static void main(String[] args) {
        CQueue obj = new CQueue();
        obj.appendTail(3);
        int param_2 = obj.deleteHead();
        System.out.println(param_2);  // 3
        int param_3 = obj.deleteHead();
        System.out.println(param_3);  // -1
    }
}
```



### 2. 包含 min 函数的栈

定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

```java
public class MinStack {

    private LinkedList<Integer> aList;
    private LinkedList<Integer> bList;

    /** initialize your data structure here. */
    public MinStack() {
        aList = new LinkedList<>();
        bList = new LinkedList<>();
    }

    public void push(int x) {
        aList.add(x);
        if (bList.isEmpty() || x <= bList.getLast()) {
            bList.add(x);
        }
    }

    public void pop() {
        Integer integer = aList.removeLast();
        if (integer.equals(bList.getLast())) {
            bList.removeLast();
        }
    }

    public int top() {
        return aList.getLast();
    }

    public int min() {
        return bList.getLast();
    }

    public static void main(String[] args) {
        MinStack minStack = new MinStack();
        minStack.push(0);
        minStack.push(1);
        minStack.push(0);
        System.out.println(minStack.min());  // -3
        minStack.pop();
        System.out.println(minStack.min());  // -2
    }
}
```





## Day2 链表（简单）

### 1. 从尾到头打印链表
输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

 示例 1：

输入：head = [1,3,2]
输出：[2,3,1]

```java
public class Solution {
    public int[] reversePrint(ListNode head) {
        LinkedList<Integer> resultList = new LinkedList<>();
        while (head != null) {
            resultList.add(head.val);
            head = head.next;
        }

        int[] result = new int[resultList.size()];
      	// 这里需要特别注意，for循环中的result.length， 不能用resultList.size()代替
        // 因为resultList.removeLast()后，resultList.size()的值会变化！！！
        for (int i = 0; i < result.length; i++) {
            result[i] = resultList.removeLast();
        }
        return result;
    }

    public static void main(String[] args) {
        ListNode listNode = new ListNode(1);
        listNode.next = new ListNode(3);
        listNode.next.next = new ListNode(2);

        Solution solution = new Solution();
        System.out.println(Arrays.toString(solution.reversePrint(listNode)));
    }
}

```

```java
// 方式二， 递归
class Solution {
    ArrayList<Integer> tmp = new ArrayList<Integer>();
    public int[] reversePrint(ListNode head) {
        recur(head);
        int[] res = new int[tmp.size()];
        for(int i = 0; i < res.length; i++)
            res[i] = tmp.get(i);
        return res;
    }
    void recur(ListNode head) {
        if(head == null) return;
        recur(head.next);
        tmp.add(head.val);
    }
}
```



### 2. 反转链表
定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

```
public class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode previousNode = null;
        while (head != null) {
            ListNode nextNode = head.next;
            head.next = previousNode;
            previousNode = head;
            head = nextNode;
        }
        return previousNode;
    }

    public static void main(String[] args) {
        ListNode listNode = new ListNode(1);
        listNode.next = new ListNode(2);
        listNode.next.next = new ListNode(3);
        listNode.next.next.next = new ListNode(4);
        listNode.next.next.next.next = new ListNode(5);

        Solution solution = new Solution();
        ListNode reverseListNode = solution.reverseList(listNode);
        while (reverseListNode != null){
            System.out.println(reverseListNode.val);
            reverseListNode = reverseListNode.next;
        }
    }
}

```

```java
// 方式二， 递归
class Solution {
    public ListNode reverseList(ListNode head) {
        return recur(head, null);    // 调用递归并返回
    }
    private ListNode recur(ListNode cur, ListNode pre) {
        if (cur == null) return pre; // 终止条件
        ListNode res = recur(cur.next, cur);  // 递归后继节点
        cur.next = pre;              // 修改节点引用指向
        return res;                  // 返回反转链表的头节点
    }
}
```



### 3. 复杂链表的复制
请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

```java
//方式一
class Solution {
    public Node copyRandomList(Node head) {
        if(head == null) return null;
        Node cur = head;
        Map<Node, Node> map = new HashMap<>();
        // 3. 复制各节点，并建立 “原节点 -> 新节点” 的 Map 映射
        while(cur != null) {
            map.put(cur, new Node(cur.val));
            cur = cur.next;
        }
        cur = head;
        // 4. 构建新链表的 next 和 random 指向
        while(cur != null) {
            map.get(cur).next = map.get(cur.next);
            map.get(cur).random = map.get(cur.random);
            cur = cur.next;
        }
        // 5. 返回新链表的头节点
        return map.get(head);
    }
}

```



```java
//方式二
class Solution {
    public Node copyRandomList(Node head) {
        if(head == null) return null;
        Node cur = head;
        // 1. 复制各节点，并构建拼接链表
        while(cur != null) {
            Node tmp = new Node(cur.val);
            tmp.next = cur.next;
            cur.next = tmp;
            cur = tmp.next;
        }
        // 2. 构建各新节点的 random 指向
        cur = head;
        while(cur != null) {
            if(cur.random != null)
                cur.next.random = cur.random.next;
            cur = cur.next.next;
        }
        // 3. 拆分两链表
        cur = head.next;
        Node pre = head, res = head.next;
        while(cur.next != null) {
            pre.next = pre.next.next;
            cur.next = cur.next.next;
            pre = pre.next;
            cur = cur.next;
        }
        pre.next = null; // 单独处理原链表尾节点
        return res;      // 返回新链表头节点
    }
}

```



## Day3 字符串（简单）


### 1. 替换空格
请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

示例 1：

输入：s = "We are happy."
输出："We%20are%20happy."

```java
class Solution {
    public String replaceSpace(String s) {
        StringBuilder res = new StringBuilder();
        for(Character c : s.toCharArray())
        {
            if(c == ' ') res.append("%20");
            else res.append(c);
        }
        return res.toString();
    }
}

```



### 2. 左旋转字符串
字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

示例 1：

输入: s = "abcdefg", k = 2
输出: "cdefgab"

```java
// 方法1 切片
class Solution {
    public String reverseLeftWords(String s, int n) {
        return s.substring(n, s.length()) + s.substring(0, n);
    }
}

```

```java
// 方法2
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder res = new StringBuilder();
        for(int i = n; i < s.length(); i++)
            res.append(s.charAt(i));
        for(int i = 0; i < n; i++)
            res.append(s.charAt(i));
        return res.toString();
    }
}


利用求余运算，可以简化代码。
class Solution {
    public String reverseLeftWords(String s, int n) {
        StringBuilder res = new StringBuilder();
        for(int i = n; i < n + s.length(); i++)
            res.append(s.charAt(i % s.length()));
        return res.toString();
    }
}

```



## Day4 查找算法（简单）

### 1.  数组中重复的数字
找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3

```java
// 利用hash表
class Solution {
    public int findRepeatNumber(int[] nums) {
        Set<Integer> dic = new HashSet<>();
        for(int num : nums) {
            if(dic.contains(num)) return num;
            dic.add(num);
        }
        return -1;
    }
}

```

```java
// 方法二：原地交换
class Solution {
    public int findRepeatNumber(int[] nums) {
        int i = 0;
        while(i < nums.length) {
            if(nums[i] == i) {
                i++;
                continue;
            }
            if(nums[nums[i]] == nums[i]) return nums[i];
            int tmp = nums[i];
            nums[i] = nums[tmp];
            nums[tmp] = tmp;
        }
        return -1;
    }
}

```



### 2. 在排序数组中查找数字 I

统计一个数字在排序数组中出现的次数。

示例 1:

输入: nums = [5,7,7,8,8,10], target = 8
输出: 2
示例 2:

输入: nums = [5,7,7,8,8,10], target = 6
输出: 0

```java
// 思路： target的右边界坐标 减去 (target-1)的右边界坐标
public int search(int[] nums, int target) {
        return searchRight(nums, target) - searchRight(nums, target-1);
    }

    private int searchRight(int[] nums, int target) {
        int start = 0;
        int end = nums.length - 1;
        while (start <= end) {
            int middle = (start + end) / 2;
            if (nums[middle] <= target) {
                start = middle + 1;
            } else {
                end = middle - 1;
            }
        }
        return start;
    }
```



复习一下二分查找

```java
public static int search2(int[] nums, int target) {
    int start = 0;
    int end = nums.length - 1;
    while (start <= end) {
      int middle = (start + end) / 2;
      if (nums[middle] < target) {
        start = middle + 1;
      } else if (nums[middle] > target) {
        end = middle - 1;
      } else {
        return middle;
      }
    }
    return -1;
}
```



### 3. 0～n-1 中缺失的数字
一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

```java
// 暴力法
class Solution {
    public int missingNumber(int[] nums) {
        for (int i=0; i<nums.length; i++) {
            if (i != nums[i]) {
                return i;
            }
        }
        return nums.length;
    }
}
```

```java
// 二分法
// 若 nums[m] == m ，则 缺失的元素 一定在闭区间 [m + 1, j]中，因此执行 i = m + 1
// 若 nums[m] != m ，则 缺失的元素 一定在闭区间 [i, m - 1]中，因此执行 j = m - 1
// i指的是start, j指的是end;

class Solution {
    public int missingNumber(int[] nums) {
        int start = 0;
        int end = nums.length - 1;
        while (start <= end) {
            int middle = (start + end) / 2;
            if (nums[middle] == middle) {
                start = middle + 1;
            } else {
                end = middle - 1;
            }
        }
        return start;
    }
}
```





## Day5 查找算法（中等）



### 1. 二维数组中的查找
在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。 

示例:

现有矩阵 matrix 如下：

[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
给定 target = 5，返回 true。

给定 target = 20，返回 false。

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix.length == 0) {
            return false;
        }
        int i = matrix.length - 1;
        int j = 0;
        while (i >=0 && j < matrix[0].length) {
            if(target < matrix[i][j]) {
                i--;
            } else if (target>matrix[i][j]) {
                j++;
            } else {
                return true;
            }
        }
        return false;
    }
}
```







### 2. 旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。

给你一个可能存在 重复 元素值的数组 numbers ，它原来是一个升序排列的数组，并按上述情形进行了一次旋转。请返回旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一次旋转，该数组的最小值为1。  

示例 1：

输入：[3,4,5,1,2]
输出：1
示例 2：

输入：[2,2,2,0,1]
输出：0

```java
// 暴力法
class Solution {
    public int minArray(int[] numbers) {
        for (int i = numbers.length-1; i>0; i--) {
            if (numbers[i-1] > numbers[i]) {
                return numbers[i];
            }
        }
        return numbers[0];
    }
}
```

```java
//二分法
class Solution {
    public int minArray(int[] numbers) {
        int i = 0;
        int j = numbers.length - 1;
        while (i != j) {
            int m = (i+j) / 2;
            if (numbers[m] > numbers[j]) {
                i = m + 1;
            } else if (numbers[m] < numbers[j]) {
                j = m;
            } else {
                j--;
            }
        }
        return numbers[i];
    }
}
```







### 3. 第一个只出现一次的字符
在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

示例 1:

输入：s = "abaccdeff"
输出：'b'
示例 2:

输入：s = "" 
输出：' '

```java
//方法一：哈希表
class Solution {
    public char firstUniqChar(String s) {
        HashMap<Character, Boolean> map = new HashMap<>();
        char[] chars = s.toCharArray();
        for (char aChar : chars) {
            if (!map.containsKey(aChar)) {
                map.put(aChar, true);
            } else {
                map.put(aChar, false);
            }
        }
        for (char aChar : chars) {
            if (map.get(aChar)) {
                return aChar;
            }
        }
        return ' ';
    }
}

```

```java
//方法二：有序哈希表
class Solution {
    public char firstUniqChar(String s) {
        Map<Character, Boolean> dic = new LinkedHashMap<>();
        char[] sc = s.toCharArray();
        for(char c : sc)
            dic.put(c, !dic.containsKey(c));
        for(Map.Entry<Character, Boolean> d : dic.entrySet()){
           if(d.getValue()) return d.getKey();
        }
        return ' ';
    }
}

```




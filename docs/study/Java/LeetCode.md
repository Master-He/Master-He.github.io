## Day1



1. 用两个栈实现队列

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



2.包含 min 函数的栈

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





## Day2

1. 从尾到头打印链表
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



2.反转链表
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



3.复杂链表的复制
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

作者：Krahets
链接：https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/9plk45/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
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

作者：Krahets
链接：https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/9plk45/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```


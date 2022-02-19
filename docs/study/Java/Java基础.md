## List操作总结



//升序排序

```java
public static void testListSort() {
    List<Integer> integers = Arrays.asList(2, 5, 9, 4, 1, 3, 2);
    Collections.sort(integers);
    System.out.println(integers);  // [1, 2, 2, 3, 4, 5, 9]

    integers = Arrays.asList(2, 5, 9, 4, 1, 3, 2);
    integers.sort(Comparator.comparing(Integer::intValue));
    System.out.println(integers);  // [1, 2, 2, 3, 4, 5, 9]
}

public static void testListSortReverse() {
    List<Integer> integers = Arrays.asList(2, 5, 9, 4, 1, 3, 2);
    integers.sort(Comparator.comparing(Integer::intValue).reversed());
    System.out.println(integers);  // [9, 5, 4, 3, 2, 2, 1]
}
```

//求和

```java
public static void testListSum() {
    List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
    int sum = integers.stream().mapToInt(Integer::intValue).sum();
    System.out.println(sum);
}
```
// 统计列表中出现最多的元素

```java
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("2");
        list.add("2");
        list.add("2");
        list.add("2");
        list.add("2");
        list.add("2");
        list.add("3");
        list.add("3");
        list.add("4");
        list.add("1");
        list.add("1");
        list.add("1");
        list.add("1");
        Map<String, Long> map = list.stream().collect(Collectors.groupingBy(x -> x, Collectors.counting()));
        System.out.println(JSON.toJSONString(map));
        Map<String, Long> result = new LinkedHashMap<>();
        map.entrySet().stream().sorted(Map.Entry.<String, Long>comparingByValue().reversed()).forEachOrdered(e -> result.put(e.getKey(), e.getValue()));
        System.out.println(JSON.toJSONString(result));
    }

```

```java
public static void getElementCount() {
        List<Integer> integers = Arrays.asList(1, 1, 1, 2, 2, 2, 2, 4, 4, 4, 4, 4, 4, 4);
        Map<Integer, Long> map = integers.stream().collect(Collectors.groupingBy(Integer::intValue, Collectors.counting()));
        System.out.println(map);
        LinkedHashMap<Integer, Long> result = new LinkedHashMap<>();
        map.entrySet().stream().sorted(Map.Entry.<Integer, Long>comparingByValue().reversed()).limit(10).forEachOrdered(e-> result.put(e.getKey(), e.getValue()));
        System.out.println(result);
        for (Integer integer : result.keySet()) {
            System.out.println(result.get(integer));
        }
    }
```



// 列表转数组

```java
public static void main(String[] args) {
        ArrayList<Integer> integers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        int[] ints = integers.stream().mapToInt(Integer::intValue).toArray();
        System.out.println(Arrays.toString(ints));  // [1, 2, 3, 4, 5]
    }
```



### LinkedList

LinkedList是双向链接串列(doubly LinkedList)。

LinkedList插入和删除比ArrayList更快，但是需要更多的内存，访问速度更慢。



## Map操作总结

map初始化

```java
 private static final Map<String,String> urlMap =new HashMap<String, String>() {
        {
            put("url1", "http://news.sina.com.cn");//新浪新闻
            put("url2", "http://news.163.com");//网易新闻
            put("url3", "http://news.qq.com");//腾讯新闻
            put("url4", "http://news.baidu.com");//百度新闻
            put("url5", "http://www.ifeng.com");//凤凰网
        }
    };
```

map转list

```java
public static void main(String[] args) {
    Map<String, String> map = new HashMap<>();
    map.put("123","===123===");
    map.put("456","===456===");
    List<String> list = new ArrayList<>(map.values());
    System.out.println(list);
}
```







**LinkedHashMap**



## Set操作总结

```java
// 测试Set<Integer> contains int
public static final int RCODE_OK = 0;
public static final int RCODE_ERROR = 2;
public static final int RCODE_NOT_FIND = 3;
public static final Set<Integer> RCODE_TYPES = new HashSet<>(Arrays.asList(RCODE_OK, RCODE_NOT_FIND));
```

```java
System.out.println("abc.com".substring(3));
System.out.println(RCODE_TYPES.contains(0));
```







## Stream操作总结

参考  https://www.jianshu.com/p/0687e7003eb2



List<Float> 转换成 float[]   参考：https://stackoverflow.com/questions/4837568/java-convert-arraylistfloat-to-float

```java
// List<Integer> 和 List<Double> 转换成 int[], double[]可以通过stream mapToInt mapToDouble
ArrayList<Integer> integers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
int[] ints = integers.stream().mapToInt(Integer::intValue).toArray();
System.out.println(Arrays.toString(ints));  // [1, 2, 3, 4, 5]

List<Double> doubles = new ArrayList<>(Arrays.asList(1d,2d,3d,4d,5d));
double[] doubles1 = doubles.stream().mapToDouble(Double::doubleValue).toArray();
System.out.println(Arrays.toString(doubles1));

// 但是List<Float> 转换成 float[] 不能！ 因为不存在mapToFloat!!! 网上找发现： 
ArrayList<Float> floats = new ArrayList<>(Arrays.asList(1f, 2f, 3f, 4f, 5f));
// new Float[0]生成是长度为0的空Float[]
float[] floats1 = ArrayUtils.toPrimitive(floats.toArray(new Float[0]), 0.0f); 
System.out.println(Arrays.toString(floats1));
```



字符串去重

```java
Set<Character> collect = "abcdefabcdef".chars().mapToObj(i -> (char) i).collect(Collectors.toSet());
        System.out.println(collect);

```



提取出list中bean的某一属性

```java
// DetectData是一个bean
List<String> domainList = detectDataList.stream().map(DetectData::getDomain).collect(Collectors.toList());
```







## Stack操作总结

栈是Vector的一个子类，它实现了一个标准的后进先出的栈。

```java
//常用的方法
Object peek( ) // 查看堆栈顶部的对象，但不从堆栈中移除它。
Object pop( ) // 移除堆栈顶部的对象，并作为此函数的值返回该对象。
Object push(Object element)  // 把项压入堆栈顶部。
```



## Vector操作总结

暂无



## Queue操作总结

队列接口方法

```java
add // 增加一个元索 如果队列已满，则抛出一个IIIegaISlabEepeplian异常
offer // 添加一个元素并返回true 如果队列已满，则返回false
put // 添加一个元素 如果队列满，则阻塞

remove // 移除并返回队列头部的元素 如果队列为空，则抛出一个NoSuchElementException异常
poll // 移除并返问队列头部的元素 如果队列为空，则返回null
take // 移除并返回队列头部的元素 如果队列为空，则阻塞
element // 返回队列头部的元素 如果队列为空，则抛出一个NoSuchElementException异常
peek // 返回队列头部的元素 如果队列为空，则返回null
```

实现队列接口的类

```java
Java 集合中的 Queue 继承自 Collection 接口 ， Deque, LinkedList, PriorityQueue, BlockingQueue 等类都实现了它。
```

队列方法总结

| 总结    | 抛出异常  | 返回特殊值 |
| ------- | --------- | ---------- |
| Insert  | add(e)    | offer(e)   |
| Remove  | remove()  | poll()     |
| Examine | element() | peek()     |







## 字符串操作

**字符串自带的常用方法** (参考菜鸟教程https://www.runoob.com/java/java-string.html)

```java
char charAt(int index);  //返回指定索引处的 char 值。
int compareTo(String anotherString) // 按ASCII字典顺序比较两个字符串。返回ASCII差值， 如果两个字符前面相同则返回长度差值
int compareToIgnoreCase(String str) // 按字典顺序比较两个字符串，不考虑大小写。
String concat(String str)  // 将指定字符串连接到此字符串的结尾。
boolean startsWith(String prefix)  // 测试此字符串是否以指定的前缀开始。
boolean endsWith(String suffix) // 测试此字符串是否以指定的后缀结束。
boolean equals(Object anObject)  // 将此字符串与指定的对象比较。
boolean equalsIgnoreCase(String anotherString) // 将此 String 与另一个 String 比较，不考虑大小写。
byte[] getBytes(String charsetName) // 使用指定的字符集将此 String 编码为 byte 序列，并将结果存储到一个新的 byte 数组中。
int indexOf(String str) // 返回指定字符在字符串中第一次出现处的索引，如果此字符串中没有这样的字符，则返回 -1
int lastIndexOf(String str) // 返回指定子字符串在此字符串中最右边出现处的索引，如果此字符串中没有这样的字符，则返回 -1。
int length() // 方法用于返回字符串的长度。
boolean matches(String regex)  // 效果等同于Pattern.matches(regex, str) 用于检测字符串是否匹配给定的正则表达式。
String replace(char searchChar, char newChar) // 方法通过用 newChar 字符替换字符串中出现的所有 searchChar 字符，并返回替换后的新字符串。
String replaceAll(String regex, String replacement) // 使用给定的参数 replacement 替换字符串所有匹配给定的正则表达式的子字符串。
String replaceAll(String regex, String replacement) // 使用给定的参数 replacement 替换字符串第一个匹配给定的正则表达式的子字符串。
String[] split(String regex, int limit) // split() 方法根据匹配给定的正则表达式来拆分字符串。
//注意： . 、 $、 | 和 * 等转义字符，必须得加 \\。
//注意：多个分隔符，可以用 | 作为连字符。
String trim() // 返回字符串的副本，忽略前导空白和尾部空白。
String valueOf(primitive data type x) //返回给定data type类型x参数的字符串表示形式。
```



**判断是否是全字母**  https://www.itranslater.com/qa/details/2136843502934819840

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

```java
// StringUtils 是org.apache.commons的commons-lang3包
public static void stringAllIsAlpha() {
    String str = "test1";
    System.out.println("test1 is all alpha? ==>" + StringUtils.isAlpha(str));  // false

    str = "test";
    System.out.println("test is all alpha? ==>" + StringUtils.isAlpha(str));  // true
}
```

```java
boolean allLetters = someString.chars().allMatch(Character::isLetter);  // chars()是stream流
```

```
public boolean isAlpha(String name) {
    return name.matches("[a-zA-Z]+");
}
```



**StringBuilder** 

StringBuffer 之间的最大不同在于 StringBuilder 的方法不是线程安全的（不能同步访问）。

StringBuffer常用方法

```java
public StringBuffer append(String s) // 将指定的字符串追加到此字符序列。
public StringBuffer reverse() // 将此字符序列用其反转形式取代。
replace(int start, int end, String str) // 使用给定 String 中的字符替换此序列的子字符串中的字符。
char charAt(int index) // 返回此序列中指定索引处的 char 值。
```













## 数组操作

// String转 ==》 char[]数组 ==》  转List
// 即String 转 List<Character>

```java
public static void stringToSetAndList() {
        String domain = "aabbcc";
        Set<Character> collect = domain.chars().mapToObj(i -> (char) i).collect(Collectors.toSet());
        List<Character> collect1 = domain.chars().mapToObj(i -> (char) i).collect(Collectors.toList());
        System.out.println(collect);
        System.out.println(collect1);
    }
```



数组初始化

```java
// 初始化空数组
new int[0];
new Integer[0];
// 初始化数组
new int[] {1,2,3};

```



二维数组的维度会改变， 并不是固定死的

```java
public static void main(String[] args) {
    float[][] floats = new float[5][10];
    System.out.println(Arrays.deepToString(floats));  // 打印出来是5*10
    for (int i = 0; i < floats.length; i++) {
        floats[i] = new float[]{i * 1f, i * 2f, i * 3f, i * 4f, i * 5f};
    }
    System.out.println(Arrays.deepToString(floats));  // 打印出来是5*5
}
```



降维

```java
// float[5][1]降维成float[5]
private static void test() {
    float[][] rawResults = new float[][] {{1f},{2f},{3f},{4f},{5f}};
    System.out.println(Arrays.deepToString(rawResults));
    // 降维
    List<Float> blackDomainProbabilities = new ArrayList<>();
    for (float[] rawResult : rawResults) {
        blackDomainProbabilities.add(rawResult[0]);
    }
    System.out.println(blackDomainProbabilities);
}
```







数组长度不足则高位填充0 , 比如要求输出是int[10]，给的输入是[1,2,3] 。 输出结果是[0,0,0,0,0,0,0,1,2,3]

下面给出一个类似的例子

```java
public class demo {
    public static void main(String[] args) {
        float[][] floats = padSequences(new float[][]{{1.0f, 2.0f, 3.0f}}, 10);
        System.out.println(Arrays.deepToString(floats));
    }
    /**
     * @param domainVectorArray 域名向量数组
     * @param n 需要返回的数组维度
     * @return 返回高位填充0的域名向量数组 domainVectorArrayWithPad
     * @Description: 例如要求返回数组float[10]，若输入[[1.0, 2.0, 3.0]] 则输出[[0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 2.0, 3.0]]
     */
    private static float[][] padSequences(float[][] domainVectorArray, int n) {
        for (int i = 0; i < domainVectorArray.length; i++) {
            float[] domainVector = domainVectorArray[i];
            float[] domainVectorWithPad = new float[n];
            for (int j = 1; j <= domainVector.length; j++) {
                domainVectorWithPad[domainVectorWithPad.length - j] = domainVector[domainVector.length - j];
            }
            domainVectorArray[i] = domainVectorWithPad;
        }
        return domainVectorArray;
    }
}
```



数组填充默认值

```java
Long[] tsArray = new Long[10];
Arrays.fill(tsArray, 1645065478000L);
List<Long> tsList = Arrays.stream(tsArray).collect(Collectors.toList());
// 或者 List<Long> tsList = Arrays.asList(tsArray);
// 参考https://blog.csdn.net/yhf597869822/article/details/104626838
```







## 正则操作

参考https://www.runoob.com/java/java-regular-expressions.html

```java
import java.util.regex.Pattern;

// 大小写不敏感， 
public static final Pattern REGISTER_URI_PATTERN = Pattern.compile("[./](regist|register|reg(\\d)?\\.|reg/)", Pattern.CASE_INSENSITIVE);


REGISTER_URI_PATTERN.matcher(uri).find()
```

java.util.regex 包主要包括以下三个类：

- Pattern 类：

  pattern 对象是一个正则表达式的编译表示。Pattern 类没有公共构造方法。要创建一个 Pattern 对象，你必须首先调用其公共静态编译方法，它返回一个 Pattern 对象。该方法接受一个正则表达式作为它的第一个参数。

  Pattern.matches(pattern, content)方法 查找字符串content中是否包了pattern

  Pattern.compile(pattern); 创建对象

- Matcher 类：

  Matcher 对象是对输入字符串进行解释和匹配操作的引擎。与Pattern 类一样，Matcher 也没有公共构造方法。你需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象。

  Matcher m = Pattern.compile(pattern).matcher(content); 查找content字符串

  m.find() 方法查找字符串content中是否包了pattern

  m.groupCount ()方法来查看表达式有多少个分组。

  m.group(0)方法获取整体匹配到的

- PatternSyntaxException：

  PatternSyntaxException 是一个非强制异常类，它表示一个正则表达式模式中的语法错误。



Matcher 类的比较重要方法

```java
public boolean find()  // 尝试查找与该模式匹配的输入序列的下一个子序列。
public boolean matches()  // 尝试将整个区域与模式匹配。
//  尝试将从区域开头开始的输入序列与该模式匹配。 lookingAt方法不需要整句都匹配，但要从第一个字符开始匹配。
public boolean lookingAt()
    
public int start() // 返回以前匹配的初始索引。
public int end()  // 返回最后匹配字符之后的偏移量。  
    
// 替换模式与给定替换字符串相匹配的输入序列的每个子序列。
public String replaceAll(String replacement)  
// 替换模式与给定替换字符串匹配的输入序列的第一个子序列。
public String replaceFirst(String replacement)
    

```









## 日志操作

// 日志打印错误信息， 打印错误堆栈信息

```java
public static final Logger logger = LoggerFactory.getLogger(指定类.class);
logger.error(e.getMessage());
logger.error(org.apache.commons.lang3.exception.ExceptionUtils.getStackTrace(e));
```



## 异常处理

```java
// 抛出异常和返回一个值类似，可以直接抛出一个异常类，也可以返回一个异常对象，对象也可以自定义异常信息 	
	@Rule
    public ExpectedException thrown = ExpectedException.none();

    @Test
    public void testThenThrow() {
        List mockedList = mock(List.class);
        when(mockedList.size()).thenReturn(2);
        when(mockedList.get(0)).thenReturn("foo");
        when(mockedList.get(1)).thenReturn("bar");
        when(mockedList.get(2)).thenThrow(IndexOutOfBoundsException.class);
        when(mockedList.remove(2)).thenThrow(new IndexOutOfBoundsException("Max index is 1."));

        Assert.assertEquals(2, mockedList.size());
        Assert.assertEquals("foo", mockedList.get(0));
        Assert.assertEquals("bar", mockedList.get(1));

        thrown.expect(IndexOutOfBoundsException.class);
        mockedList.get(2);

        thrown.expect(IndexOutOfBoundsException.class);
        thrown.expectMessage("Max index is 1.");
        mockedList.remove(2);
    }
```



## 时间





## 文件IO操作

```java
import org.apache.commons.io.FileUtils;

// 实际路径是resources/acdgadetect/wordsCh.json 
String wordsPath = /acdgadetect/wordsCh.json  
    
String path = 指定类.class.getResource(wordsPath).getPath();
String jsonStr = FileUtils.readFileToString(new File(path), StandardCharsets.UTF_8);
```

```java
//导入图
byte[] graphBytes = IOUtils.toByteArray(new FileInputStream(TestModel.class.getResource(PB_FILE_PATH).getPath()));
graph.importGraphDef(graphBytes);
```

```java
// 按行读文件，读成set
public static void main(String[] args) {
    String path = FileDemo.class.getResource("/a.txt").getPath();
    try {
        System.out.println(new HashSet<>(FileUtils.readLines(new File(path), StandardCharsets.UTF_8)));
    } catch (IOException e) {
        logger.error("loadWords [" + path + "] failed.");
        logger.error(ExceptionUtils.getStackTrace(e));
    }
}
```



## java 对象销毁时操作

// 重写finalize方法， jvm会调用

```java
// Java code to show the
// overriding of finalize() method

import java.lang.*;

// Defining a class demo since every java class
// is a subclass of predefined Object class
// Therefore demo is a subclass of Object class
public class demo {
	protected void finalize() throws Throwable
	{
		try {
			System.out.println("inside demo's finalize()");
		}
		catch (Throwable e) {
			throw e;
		}
		finally {
			System.out.println("Calling finalize method" + " of the Object class");
			// Calling finalize() of Object class
			super.finalize();
		}
	}

	// Driver code
	public static void main(String[] args) throws Throwable
	{
		// Creating demo's object
		demo d = new demo();
		// Calling finalize of demo
		d.finalize();
	}
}

```



为什么尽量不要Override finalize方法

https://yfsyfs.github.io/2019/06/28/%E4%B8%BA%E4%BB%80%E4%B9%88%E5%B0%BD%E9%87%8F%E4%B8%8D%E8%A6%81Override-finalize%E6%96%B9%E6%B3%95/



可以使用try(){}语句 : try()括号中的资源会自动调用close()方法



## Maven

菜鸟教程 https://www.runoob.com/maven/maven-pom.html

视频https://www.bilibili.com/video/BV1Fz4y167p5?p=10&spm_id_from=pageDriver

常用命令

```shell
mvn compile  #编译
mvn exec:java -Dexec.mainClass="com.xxx.demo.Hello"  # 启动项目
mvn package -Dmaven.test.skip=true  # 打包跳过测试
mvn packge -Pdev  # 指定dev开发环境的profile进行打包（还有测试环境，生产环境）
```

 



## 为啥要实现Serializable接口？
https://zhuanlan.zhihu.com/p/66210653





## 单例

```java
// 懒汉模式， 静态
public class PwdUEBAKeyword {
	private static final class InstanceHolder {
        static final PwdUEBAKeyword instance = new PwdUEBAKeyword();
    }

    public static PwdUEBAKeyword getInstance() {
        return InstanceHolder.instance;
    }
}
```



## 排序

参考https://zhuanlan.zhihu.com/p/376672600

java种默认的排序用的是什么算法？

**Arrays.sort()** 看底层代码，对于int[]数组，当数组长度len<47用插入排序，47<len<286的用双轴快排， len>286的用归并排序

**Collections.sort()**用的是Arrays.sort() 的其中一种重载方法, Arrays.sort()根据情况使用TimSort排序或者传统的归并排序

双轴快速排序， TimSort， 归并排序介绍文章https://blog.51cto.com/u_15103028/2647024

排序算法有9种

**分类**

- **插入排序：** 直接插入排序、二分法插入排序、希尔排序
- **选择排序：** 简单选择排序、堆排序
- **交换排序：** 冒泡排序、快速排序(分治)
- **归并排序**
- **基数排序**

```java
// 快速排序： 选一个基准元素，把小于基准元素的放在左边，大于基准元素的放在右边。 然后左右两边的按同样的方式递归
public class QuickSort {
    public static void main(String[] args) {
        int[] nums = new int[]{3, 1, 4, 2, 5};
        System.out.println("排序前" + Arrays.toString(nums));
        quickSort(nums, 0, nums.length - 1);
        System.out.println("排序后" + Arrays.toString(nums));
    }

    private static void quickSort(int[] nums, int l, int r) {
        if (l >= r) {
            return;
        }
        int i = l;
        int j = r;
        int tmp = nums[i];
        while (i < j) {
            // 基准元素是nums[l]
            while (i < j && nums[j] >= nums[l]) {
                j--;
            }
            while (i < j && nums[i] <= nums[l]) {
                i++;
            }
            tmp = nums[i];
            nums[i] = nums[j];
            nums[j] = tmp;
        }
        // 这个时候i=j, num[i]和基准元素互换(tmp=num[i]在上面已经定义)
        nums[i] = nums[l];
        nums[l] = tmp;
        quickSort(nums, l, i - 1);
        quickSort(nums, i + 1, r);
    }
}
```


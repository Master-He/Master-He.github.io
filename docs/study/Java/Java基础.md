## Java 基础语法
### List操作总结



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





### Map操作总结

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





**LinkedHashMap**



### Set操作总结



### Stream操作总结

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







### 字符串操作

判断是否是全字母  https://www.itranslater.com/qa/details/2136843502934819840

```java
public static void stringAllIsAlpha() {
    String str = "test1";
    System.out.println("test1 is all alpha? ==>" + StringUtils.isAlpha(str));  // false

    str = "test";
    System.out.println("test is all alpha? ==>" + StringUtils.isAlpha(str));  // true

}
```

```java
boolean allLetters = someString.chars().allMatch(Character::isLetter);
```

```
public boolean isAlpha(String name) {
    return name.matches("[a-zA-Z]+");
}
```







### 数组操作

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







### 正则操作

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









### 日志操作

// 日志打印错误信息， 打印错误堆栈信息

```java
public static final Logger logger = LoggerFactory.getLogger(指定类.class);
logger.error(e.getMessage());
logger.error(org.apache.commons.lang3.exception.ExceptionUtils.getStackTrace(e));
```



### 异常处理

```java

```



### 文件IO操作

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





### float double精度





### java 对象销毁时操作

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
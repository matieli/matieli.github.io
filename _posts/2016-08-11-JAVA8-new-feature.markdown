---
layout: post
title: JAVA8新特性
date: 2016-08-11 15:32:24.000000000 +09:00
tag: JAVA
---


Java8新特性，函数式编程，使用Lambdas我们能做到什么？
>- 遍历集合(List、Map等)、Sum、Max、Min、Avg、Sort、Distinct等等
>- 函数接口
>- 谓词(Predicate)使用
>- 实现Map和Reduce
>- 实现事件处理/简化多线程

### 内部循环和外部循环
``` java
public static void test1() {
	List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

	for (int number : numbers) {
		System.out.println(number);
   }
}
```
上述代码是传统方式的遍历一个List的写法，简单来说主要有3个不足：
>-  只能顺序处理list中的数据(process one by one)
>-  不能充分利用多核cpu
>-  不利于编译器优化(jit)

下面看看使用Lambda怎么写

```
//使用lambda表达式以及函数操作(functional operation)  
public static void test2(){
	List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
	numbers.forEach((Integer value)-> System.out.println(value));
}
```

或者

```
//在Java8中使用双冒号操作符(double colon operator) 
public static void test3(){
	List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
	numbers.forEach(System.out::println);
}
```

这样就能规避上面的三个问题：
>- 不一定需要顺序处理List中的元素，顺序可以不确定
>- 可以并行处理，充分利用多核CPU的优势
>- 有利于JIT编译器对代码进行优化
>- 代码看起来更简洁，完全交给编译器内部循环

### 传递行为，而不仅仅是传值

```
//sumAll算法很简单，完成的是将List中所有元素相加。
public static int sumAll(List<Integer> numbers) {
	int total = 0;
	for (int number : numbers) {
		total += number;
	}
	return total;
}
```

sumAll算法很简单，完成的是将List中所有元素相加。某一天如果我们需要增加一个对List中所有偶数求和的方法sumAllEven,那么就产生了sumAll2，如下：

```
public static int sumAll2(List<Integer> numbers) {
	int total = 0;
	for (int number : numbers) {
		if (number % 2 == 0) {
			total += number;  
		}  
	}
	return total;
}
```

又有一天，我们需要增加第三个方法：对List中所有大于3的元素求和，那是不是继续加下面的方法呢?sumAll3

```
public static int sumAll3(List<Integer> numbers) {
	int total = 0;
	for (int number : numbers) {
		if (number > 3) {
			total += number;  
		}  
	}
	return total;
}
```
观察这三个方法我们发现，有很多重复内容，唯一不同的是方法中的if条件不一样(第一个可以看成if(true))，如果让我们优化，可能想到的第一种重构就是策略模式吧，代码如下：

```
public interface Strategy {
    public boolean test(int num);
}
 
public class SumAllStrategy implements Strategy {
    @Override
    public boolean test(int num) {
        return true;
    }
}
 
public class SumAllEvenStrategy implements Strategy {
    @Override
    public boolean test(int num) {
        return num % 2 == 0;
    }
}
 
public class SumAllGTThreeStrategy implements Strategy {
    @Override
    public boolean test(int num) {
        return num > 3;
    }
}
public class BodyClass {
    private Strategy stragegy = null;
    private final static Strategy DEFAULT_STRATEGY = new SumAllStrategy();
 
    public BodyClass() {
        this(null);
    }
 
    public BodyClass(Strategy arg) {
        if (arg != null) {
            this.stragegy = arg;
        } else {
            this.stragegy = DEFAULT_STRATEGY;
        }
    }
 
    public int sumAll(List<Integer> numbers) {
        int total = 0;
        for (int number : numbers) {
            if (stragegy.test(number)) {
                total += number;
            }
        }
        return total;
    }
}

//调用
BodyClass  bc = new BodyClass();
bc.sumAll(numbers);
```

这无疑使用设计模式的方式优化了冗余代码，但是可能要额外增加几个类，以后扩展也要新增，下面看看使用lambda如何实现，声明方法：第一个参数还是我们之前传递的List数组，第二个看起来可能有点陌生，通过查看jdk可以知道，这个类是一个谓词（布尔值的函数）

```
public static int sumAllByPredicate(List<Integer> numbers, Predicate<Integer> p) {  
        int total = 0;  
        for (int number : numbers) {  
            if (p.test(number)) {  
                total += number;  
            }  
        }  
        return total;  
    }  
//调用：
sumAllByPredicate(numbers, n -> true);
sumAllByPredicate(numbers, n -> n % 2 == 0);
sumAllByPredicate(numbers, n -> n > 3);
```

代码是不是比上面简洁了很多？语义也很明确，重要的是不管以后怎么变，都可以一行代码就修改了。。。万金油啊
### 使用lambdas 来实现 Runnable接口

```
// 1.1使用匿名内部类  
new Thread(new Runnable() {  
	@Override  
	public void run() {  
		System.out.println("Hello world !");  
	}  
}).start();


// 1.2使用 lambda expression  
new Thread(()->System.out.println("Hello lambda !")).start();

// 2.1使用匿名内部类  
Runnable race1 = new Runnable() {  
	@Override  
	public void run() {  
		System.out.println("race1,Hello world  !");  
	}  
};
//直接调用 run方法(没开新线程哦!)  
race1.run();

// 2.2使用 lambda expression  
Runnable race2 = () -> System.out.println("race2,Hello lambda !");  
// 直接调用run方法(没开新线程哦!)  
race2.run();
```

### Consumer与Loan Pattern

```
class Resource {  
      
    public Resource() {  
        System.out.println("Opening resource");  
    }  
  
    public void operate() {  
        System.out.println("Operating on resource");  
    }  
  
    public void dispose() {  
        System.out.println("Disposing resource");  
    }  
}
```

因为对资源对象resource执行operate方法时可能抛出RuntimeException，所以需要在finally语句块中释放资源，防止可能的内存泄漏。

```
Resource resource = new Resource();  
try {  
    resource.operate();  
} finally {  
    resource.dispose();  
}
```
但是有一个问题，如果很多地方都要用到这个资源，那么就存在很多段类似这样的代码，这很明显违反了DRY（Don't Repeat It Yourself）原则。而且如果某位程序员由于某些原因忘了用try/finally处理资源，那么很可能导致内存泄漏。那咋办呢？Java 8提供了一个Consumer接口，代码改写为如下:

```
class ResourceJava8 {  
      
    private ResourceJava8() {  
        System.out.println("Opening resource");  
    }  
  
    public void operate() {  
        System.out.println("Operating on resource");  
    }  
  
    public void dispose() {  
        System.out.println("Disposing resource");  
    }  
  
    public static void withResource(Consumer<ResourceJava8> consumer) {  
        ResourceJava8 resource = new ResourceJava8();  
        try {  
            consumer.accept(resource);  
        } finally {  
            resource.dispose();  
        }  
    }  
}
```
外部要访问Resource不能通过它的构造函数了（private），只能通过withResource方法了，这样代码清爽多了，而且也完全杜绝了因人为疏忽而导致的潜在内存泄漏。调用:

```
ResourceJava8.withResource(resourceJava8 -> resourceJava8.operate());
```

### Streams与集合
Stream是对集合的包装,通常和lambda一起使用。 使用lambdas可以支持许多操作,如 map, filter, limit, sorted, count, min, max, sum, collect 等等。 同样,Stream使用懒运算,他们并不会真正地读取所有数据,遇到像getFirst() 这样的方法就会结束链式语法，通过下面一系列例子介绍：
比如我有个Person类，就是一个简单的pojo:

```
class Person{
    private String name, job, gender;  
    private int salary, age;  
    public Person(String name,String job,String gender,int age,int salary){
        this.name = name;
        this.job = job;
        this.gender = gender;
        this.salary = salary;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getJob() {
        return job;
    }
    public void setJob(String job) {
        this.job = job;
    }
    public String getGender() {
        return gender;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
    public int getSalary() {
        return salary;
    }
    public void setSalary(int salary) {
        this.salary = salary;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person [name=" + name + ", job=" + job + ", gender=" + gender
                + ", salary=" + salary + ", age=" + age + "]";
    }
     
}
```

一系列测试：

```
List<Person>  persons = new ArrayList<Person>() {
    private static final long serialVersionUID = 1L;
    {
        add(new Person("张三", "Java", "female", 25, 1000));
        add(new Person("李四", "Java", "male", 29, 1200));
        add(new Person("王五", "测试", "female", 25, 1400));
        add(new Person("赵六", "Java", "male", 31, 1800));
        add(new Person("张三三", "设计", "male", 33, 1900));
        add(new Person("李四四", "需求", "female", 30, 2000));
        add(new Person("王五五", "Java", "female", 29, 2100));
        add(new Person("赵六六", "Java", "male", 43, 2800));
    }
};
//1.使用foreach输出上述列表
System.err.println("1------------");
Consumer<Person> print = e -> System.out.println(e.toString());
persons.forEach(print);  
//2.将所有员工工资涨10%,使用foreach
System.err.println("2------------");
Consumer<Person> raise = e -> e.setSalary(e.getSalary()/100*10+e.getSalary());
persons.forEach(raise);
persons.forEach(print);
//3.显示工资低于1500的员工，使用stream().filter()
System.err.println("3------------");
persons.stream().filter((p) -> (p.getSalary()< 1500)).forEach(print);
//4.显示工资大于2000 job=java 年龄>29 的女生
System.err.println("4------------");
Predicate<Person> salaryPredicate = e -> e.getSalary() > 2000;
Predicate<Person> jobPredicate = e -> "Java".equals(e.getJob());
Predicate<Person> agePredicate = e -> e.getAge() >= 29;
Predicate<Person> genderPredicate = e -> "female".equals(e.getGender());
persons.stream().filter(salaryPredicate)
                .filter(jobPredicate)
                .filter(agePredicate)
                .filter(genderPredicate)
                .forEach(print);
//5.限制结果条数limit
System.err.println("5------------");
persons.stream().filter(genderPredicate).limit(2).forEach(print);
//6.按照年龄排序
System.err.println("6------------");
persons.stream().sorted((p1,p2)-> (p1.getAge() - p2.getAge()))
                //.sorted((p1,p2)->(p1.getName().compareTo(p2.getName())))
                .forEach(print);
//7.找出工资最高max()，年龄最小的min()
System.err.println("7------------");
System.out.println(persons.stream().min((p1,p2)->(p1.getSalary()-p2.getSalary())).get().toString());
System.out.println(persons.stream().max((p1,p2)->(p1.getSalary()-p2.getSalary())).get().toString());
 
//8.计算所有人的工资parallel()并行的计算
System.err.println("8------------");
System.out.println("所有人的工资总和："+persons.stream().parallel().mapToInt(p->p.getSalary()).sum());
 
//9.将人员姓名存放到TreeSet\set\String中
System.err.println("9------------");
String str = persons.stream().map(Person::getName).collect(Collectors.joining(";"));
System.out.println(str);
TreeSet<String> ts = persons.stream().map(Person::getName).collect(Collectors.toCollection(TreeSet::new));
System.out.println("ts.toString():"+ts.toString());
Set<String> set = persons.stream().map(Person::getName).collect(Collectors.toSet());
System.out.println("set.toString():"+set.toString());
//10.统计结果summaryStatistics()
System.err.println("10------------");
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);  
IntSummaryStatistics stats = numbers  
          .stream()  
          .mapToInt((x) -> x)  
          .summaryStatistics();  
   
System.out.println("List中最大的数字 : " + stats.getMax());  
System.out.println("List中最小的数字 : " + stats.getMin());  
System.out.println("所有数字的总和   : " + stats.getSum());  
System.out.println("所有数字的平均值 : " + stats.getAverage());  
//11.去除重复元素，创建新数组
System.err.println("11------------");
List<Integer> numbers1 = Arrays.asList(9, 10, 3, 4, 7, 3, 4);
List<Integer> distinct = numbers1.stream().distinct().collect(Collectors.toList());
System.out.println(distinct);
```

### 日期处理
Java8新增了LocalDate和LocalTime接口，为什么要搞一套全新的处理日期和时间的API？因为旧的java.util.Date实在是太难用了。
>- java.util.Date月份从0开始，一月是0，十二月是11，变态吧！java.time.LocalDate月份和星期都改成了enum，就不可能再用错了。
>- java.util.Date和SimpleDateFormatter都不是线程安全的，而LocalDate和LocalTime和最基本的String一样，是不变类型，不但线程安全，而且不能修改。
>- java.util.Date是一个“万能接口”，它包含日期、时间，还有毫秒数，如果你只想用java.util.Date存储日期，或者只存储时间，那么，只有你知道哪些部分的数据是有用的，哪些部分的数据是不能用的。在新的Java8中，日期和时间被明确划分为LocalDate和LocalTime，LocalDate无法包含时间，LocalTime无法包含日期。

当然，LocalDateTime才能同时包含日期和时间。

新接口更好用的原因是考虑到了日期时间的操作，经常发生往前推或往后推几天的情况。用java.util.Date配合Calendar要写好多代码，而且一般的开发人员还不一定能写对。

```
public class Test1 {
	public static void main(String[] args) {
		
		// 1.Clock时钟
		// Clock类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代 System.currentTimeMillis()
		// 来获取当前的微秒数。某一个特定的时间点也可以使用Instant类(为Final类)来表示，Instant类也可以用来创建老的java.util.Date对象。
		System.err.println("Clock时钟");
		Clock c = Clock.systemDefaultZone();
		System.out.println(System.currentTimeMillis());
		System.out.println(c.millis());
		Date date = Date.from(c.instant());
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		System.out.println(sdf.format(date));
		// instant精确到纳秒，比原来的date的毫秒要更精确
		// 获取当前时间
		Instant in = Instant.now();
		System.out.println(in);
		// 将现在的时间增加3小时2分,将产生新的实例
		Instant in1 = in.plus(Duration.ofHours(3).plusMinutes(2));
		System.out.println(in1);
		System.out.println(in1 == in);
		// 关于计算的例子
		in.minus(5, ChronoUnit.DAYS);// 计算5天前
		in.minus(Duration.ofDays(5));// 计算5天前

		// 计算两个Instant之间的分钟数
		long diffAsMinutes1 = ChronoUnit.MINUTES.between(in, in1); // 方法2
		System.out.println(diffAsMinutes1);
		// instant是可比较的，有isAfter和isBefore
		System.out.println(in.isAfter(in1));
		System.out.println(in.isBefore(in1));

		// 2.LocalDate和LocalTime、LocalDateTime(均为Final类,不带时区)
		System.err.println("2.LocalDate和LocalTime、LocalDateTime");
		LocalDate today = LocalDate.now();//取当前日期
		System.out.println(today);
		//获得2005年的第86天 (27-Mar-2005)
		LocalDate localDate = LocalDate.ofYearDay(2005, 86); 
		System.out.println(localDate);
		// 根据年月日取日期 2013年8月10日
		localDate = LocalDate.of(2013, Month.AUGUST, 10); 
		localDate = LocalDate.of(2013, 8, 10);
		
		// 根据字符串取
		LocalDate.parse("2014-02-28"); // 严格按照ISO yyyy-MM-dd验证，02写成2都不行
		LocalDate.parse("2014-02-29"); // 无效日期无法通过：DateTimeParseException: Invalid date
		
		// 取本月第1天：
		LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth()); // 2014-12-01
		// 取本月第2天：
		LocalDate secondDayOfThisMonth = today.withDayOfMonth(2); // 2014-12-02
		// 取本月最后一天，再也不用计算是28，29，30还是31：
		LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth()); // 2014-12-31
		// 取下一天：
		LocalDate firstDayOf2015 = lastDayOfThisMonth.plusDays(1); // 变成了2015-01-01
		// 取2015年1月第一个周一，这个计算用Calendar要死掉很多脑细胞：
		LocalDate firstMondayOf2015 = LocalDate.parse("2015-01-01").with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY)); // 2015-01-05
		
		//LocalTime
		LocalTime now = LocalTime.now();//带纳秒，如果想清除纳秒
		LocalTime now1 = LocalTime.now().withNano(0);
		System.out.println(now);
		System.out.println(now1);
		LocalTime localTime = LocalTime.of(22, 33); // 10:33 PM
		
		System.out.println(localTime);
		localTime = LocalTime.ofSecondOfDay(4503); // 返回一天中的第4503秒 (1:15:30 AM)
		System.out.println(localTime);

		LocalDateTime localDateTime0 = LocalDateTime.now();
		System.out.println(localDateTime0);
		//当前时间加上25小时３分钟

		LocalDateTime inTheFuture = localDateTime0.plusHours(25).plusMinutes(3);
		System.out.println(inTheFuture);
		//同样也可以用在localTime和localDate中

		System.out.println(localDateTime0.toLocalTime().plusHours(25).plusMinutes(3));

		System.out.println(localDateTime0.toLocalDate().plusMonths(2));

		//也可以使用实现TemportalAmount接口的Duration类和Period类

		System.out.println(localDateTime0.toLocalTime().plus(Duration.ofHours(25).plusMinutes(3)));
		System.out.println(localDateTime0.toLocalDate().plus(Period.ofMonths(2)));
		LocalDateTime localDateTime1 = LocalDateTime.MIN;
		System.out.println(localDateTime1);
		
		//LocalDateTime和Instant两者很像都是不带时区的日期和时间，Instant中是不带时区的即时时间点。可能会有疑问，即时的时间点不就是日期＋时间么
		//举个例子：两个人都是2016年4月14日出生的，一个出生在北京，一个出生在纽约；看上去他们是一起出生的(LocalDateTime的语义)，其实他们是有时间差的(Instant的语义)
	}
}
```
### Default 方法

```
public class Test1 {
	public static void main(String[] args) {
		Formula formula = new Formula() {
			@Override
			public double calculate(int a) {
				return sqrt(a * 100);
			}
		};
		System.out.println(formula.calculate(100)); // 100.0
		System.out.println(formula.sqrt(16)); // 4.0
	}
}

interface Formula {
	double calculate(int a);

	default double sqrt(int a) {
		return Math.sqrt(a);
	}
}
```

### 字符串拼接

```
String joined = String.join("/", "usr","local","bin");
String joided1="usr/"+"local/"+"bin/";
System.out.println(joined);

String ids = String.join(", ", ZoneId.getAvailableZoneIds());
System.out.println(ids);
```

### Objects 类

```
String aa = null;
		
Objects.requireNonNull(aa," aa must be not null");


Object a = null;
Object b = new Object();
if(a.equals(b)){
	
}
if(Objects.equals(a, b)){
}
```

### Base64编码的规范化

```
Base64.Encoder encoder = Base64.getEncoder();
Base64.Decoder decoder = Base64.getDecoder();
String str = encoder.encodeToString("你好".getBytes(StandardCharsets.UTF_8));
System.out.println(str);
System.out.println(new String(decoder.decode(str),StandardCharsets.UTF_8));
```

### java 7新的捕获异常的类，以上异常的父类

```
try {
	Class<?> forName = Class.forName("java.lang.Object");
	
	Method method = forName.getMethod("toString");
	
	System.out.println(method.invoke(forName)+"====");
} catch (IllegalAccessException | IllegalArgumentException
		| InvocationTargetException | NoSuchMethodException
		| SecurityException | ClassNotFoundException e) {
	e.printStackTrace();
}catch (ReflectiveOperationException e) {// 新的捕获异常的类，以上异常的父类
	e.printStackTrace();
}
```
### JAVA 7 自动资源管理

之前写法

```
FileInputStream in = null;  
FileOutputStream out = null;  
try {  
  in = new FileInputStream("xanadu.txt");  
  out = new FileOutputStream("outagain.txt");  
  int c;  
  while ((c = in.read()) != -1)  
    out.write(c);  
} finally {  
  if (in != null)  
    in.close();  
  if (out != null)  
    out.close();  
}
```

JAVA 7写法

```
try (  
  FileInputStream in = new FileInputStream("xanadu.txt");  
  FileOutputStream out = new FileOutputStream("outagain.txt")  
) {  
  int c;  
  while((c=in.read()) != -1 )  
    out.write();  
}
```























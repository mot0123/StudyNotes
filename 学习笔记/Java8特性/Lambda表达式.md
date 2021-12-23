## Lambda表达式

#### 一、使用的过程

最初的代码：

```text
package com.isea.java;
public class TestLambda {
       public static void main(String[] args) {
       Thread thread = new Thread(new MyRunnable());
       thread.start();
       thread.close();
    }
}
class MyRunnable implements Runnable{
	    @Override
        public void run() {
            System.out.println("Hello");
        }
}
```

匿名内部类重构：

```text
package com.isea.java;
public class TestLambda {
    public static void main(String[] args) {
        new Thread(new Runnable() {
        //这里的new Runnable()，这里new 了接口，在这个new的接口里面，我们写了这个接口的实现类。
        //这里可以看出，我们把一个重写的run()方法传入了一个构造函数中。
            @Override
            public void run() {
                System.out.println("Hello");
            }
        }).start();
    }
}
```

最终简化：

```text
package com.isea.java;
public class TestLambda {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("Hello")).start();
    }
}
```

**lambda表达式的分类：**

1、无参无返回值

```text
package com.isea.java;
public class TestLambda {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("Hello"));
    }
}
```

//()中无参数，也不能省略；{}中只有一句话，建议省略。

2、有参无返回值

```text
package com.isea.java;
import java.util.ArrayList;
public class TestLambda {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("AAAAA");
        list.add("BBBBB");
        list.add("CCCCC");
        list.add("DDDDD");
//形参的类型是确定的，可省略；只有一个形参，()可以省略；
        list.forEach(t -> System.out.print(t + "\t"));
//或者更简洁的方法引用：list.forEach(System.out::println);
        //打印结果：AAAAA	BBBBB	CCCCC	DDDDD
    }
}
public void forEach(Consumer<? super E> action)
```

3、无参有返回值

```text
package com.isea.java;
import java.util.Random;
import java.util.stream.Stream;
public class TestLambda {
    public static void main(String[] args) {
        Random random = new Random();
        Stream<Integer> stream = Stream.generate(() ->random.nextInt(100));
        stream.forEach(t -> System.out.println(t));
    }//只有一个return，可以省略return；该方法将会不断的打印100以内的正整数。
}//Stream.generate()方法创建无限流，该方法要求传入一个无参有返回值的方法。
public static<T> Stream<T> generate(Supplier<T> s) //来自源码
```

4、有参有返回值

```text

public class TestLambda {
    public static void main(String[] args) {
        Collator collator = Collator.getInstance();
        TreeSet<Student> set = new TreeSet<>((s1,s2) -> collator.compare(s1.getName(),s2.getName()));
        set.add(new Student(10,"张飞"));
        set.add(new Student(3,"周瑜"));
        set.add(new Student(1,"宋江"));
        set.forEach(student -> System.out.println(student));
    }
}//这里的Collator是一个抽象类，但是提供了获取该类实例的方法getInstance()
 
class Student{
    private int id;
    private String name;
    
    public Student(int id, String name) {
        this.id = id;
        this.name = name;
    }
 
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        Home | This.ID = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

#### 二、函数式接口

##### 1、什么是函数式接口？

（1）、只包含一个抽象方法的接口，称为**函数式接口**。

（2）、你可以通过Lambda表达式来创建该接口的对象。（若Lambda表达式抛出一个受检异常，那么该异常需要在目标接口的抽象方法上进行声明）。

（3）、我们可以在任意函数式接口上使用@FunctionalInterface注解，这样做可以检查它是否是一个函数式接口，同时javadoc也会包含一条声明，说明这个接口是一个函数式接口。

##### 2、作为参数传递Lambda表达式

```java
@FunctionalInterface
public interface MyFunction<T>{
    public  T  getValue(T t);
}
```

```rust
public String strHandler(String str, MyFunction mf) {
        return mf.getValue(str);
    }
作为参数传递Lambda表达式：
String trimStr = strHandler("\t\t 你是大傻逼       ", (str) -> str.trim());
String upperStr = strHandler("abcdefg", (str) -> str.toUpperCase());
String newStr = strHandler("我大望江威武", (str) -> str.substring(2, 5));
```

##### 3、Java内置四大函数式接口

![img](https://upload-images.jianshu.io/upload_images/5654620-54f3bb66b35cd761.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

示例：

###### （1）Consumer<T> 

: 消费型接口，void accept(T t);

```csharp
@Test
    public void test() {
        happy(10000,(m) -> System.out.println("大保健花了："+m));
    }
    public void happy(double  money,Consumer<Double> con) {
        con.accept(money);
    }
```

###### （2） Supplier<T>

 : 供给型接口，T get();

```csharp
@Test
    public void test1() {
        List<Integer> numList = getNumList(10, ()->(int)(Math.random()*100 ));
        for (Integer integer : numList) {
            System.out.println(integer);
        }
    }
    
    
    //需求：产生指定个数的整数，并放入集合中
    public List<Integer> getNumList(int num,Supplier<Integer> sup){
            List<Integer> list = new ArrayList<>();
            for(int i=0;i<num;i++) {
                
                Integer n = sup.get();
                list.add(n);
                
            }
            return list;
    }
```

###### （3）Function<T, R> 

: 函数型接口，R apply(T t);

```kotlin
@Test
    public void  test2() {
        String trimStr=strHandler("\t\t  你好，world！   ",(str) -> str.trim());
        System.out.println(trimStr);
        
        
        String sumString=strHandler("Helloworld!",(str)->str.substring(2, 4));
        System.out.println(sumString);
    }
    //需求：用于处理字符串
    public  String strHandler(String str,Function<String,String> fun) {
        return fun.apply(str);
    }
    
```

###### （4）Predicate<T> 

: 断言型接口，boolean test(T t);
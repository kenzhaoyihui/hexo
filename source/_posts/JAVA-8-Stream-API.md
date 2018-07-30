title: JAVA 8 -- Stream API
author: Kenzhaoyihui
tags:
  - JAVA
categories:
  - JAVA
date: 2018-07-27 15:25:00
---
今天来总结一下 Java8 新特性Stream API的使用。

### Stream API

Stream流程： 创建流--> 中间操作 --> 终止操作

* #### 创建流的几种方法

	1. Collection 系列的stream() 方法
    2. Arrays的stream() 方法
    3. Stream 接口的of方法
    4. Stream 无限流的创建， 利用迭代和生成
    
```
package com.yzhao.cks.stream;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

public class CreateStream {
    public static void main(String[] args) {
        //1. Collections
        List<String> list = new ArrayList<>();
        Stream<String> stringStream = list.stream();

        //2. Arrays
        String[] strings = new String[10];
        Stream<String> stringStream1 = Arrays.stream(strings);

        //3. Stream of
        Stream<String> stringStream2 = Stream.of("aa", "bb", "cc");
        stringStream2.forEach(System.out::println);

        //4. iterate and generate
        Stream<Integer> stream = Stream.iterate(5, x->x+3);
        stream.limit(5).forEach(System.out::println);

        Stream<Double> stream1 = Stream.generate(()->Math.random());
        stream1.limit(10).forEach(System.out::println);
    }
}
```


创建一个Employee class 用于后面的测试：

```
package com.yzhao.cks.stream;


public class Employee {

    private String name;
    private Integer age;
    private Double salary;

    private Status status;

    public enum Status{
        BUSY,
        FREE,
        VOCATION;
    }

    public Employee(String name, Integer age, Double salary) {
        this.salary = salary;
        this.name = name;
        this.age = age;
    }

    public Employee(String name, Integer age, Double salary, Status status) {
        this.name = name;
        this.age = age;
        this.salary = salary;
        this.status = status;
    }

    public Status getStatus() {
        return status;
    }

    public void setStatus(Status status) {
        this.status = status;
    }

    public String getName() {
        return name;
    }

    public Integer getAge() {
        return age;
    }

    public Double getSalary() {
        return salary;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setName(String name) {
        this.name = name;
    }


    public void setSalary(Double salary) {
        this.salary = salary;
    }

    @Override
    public int hashCode() {
        //return super.hashCode();
//        final int prime = 31;
//        int result = 1;
//        result = prime * result + ((age==null)? 0:age.hashCode());
//        result = prime * result + ((name==null)? 0:name.hashCode());
//        result = prime * result + ((salary==null)?0:salary.hashCode());
//        return result;
        return age.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        //return super.equals(obj);
        Employee employee = (Employee) obj;
        return age.equals(employee.age);
    }

    @Override
    public String toString() {
        return "Employee [" +
                "name=" + name +
                ", age=" + age +
                ", salary=" + salary +
                "]";
    }
}

```


* #### Stream流的中间操作
	1. filter, distinct, skip, limit
    2. map, flatMap
    3. sorted, sorted(Comparator comparator)
    4. reduce
    5. collect
    
```
package com.yzhao.cks.stream;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Stream;

public class TestStream1 {

    public static void main(String[] args) {

        List<Employee> emps = Arrays.asList(
                new Employee("zyh", 23, 1000.0),
                new Employee("lj", 12, 2000.0),
                new Employee("hah", 34, 3000.0),
                new Employee("xixi", 24, 300.0),
                new Employee("fefe", 45, 4000.0),
                new Employee("zyh", 23, 1000.0)
        );

        //filter, distinct, skip, limit
        emps.stream().filter(e -> {
            //System.out.println("hello filter");
            return e.getAge()>20;
        }).limit(10)
                .distinct() // from hasCode and equals, so need to re-write the hasCode() and equals() method
                .skip(2)
                    .forEach(System.out::println);// lazy load



        //map, flatMap
        List<String> list = Arrays.asList("fff", "aaa", "bbb", "ccc", "ddd", "eee");

        list.stream().map(str -> str.toUpperCase())
                .forEach(System.out::println);

        emps.stream().map(Employee::getName).forEach(System.out::println);

        System.out.println("------------------------------");

        Stream<Character> stream = list.stream()
                .flatMap(TestStream1::filterCharacter);

        stream.forEach(System.out::println);

        System.out.println("----------------------------------");

        //Sorted, Sorted(Comparator comparator)
        list.stream().sorted().forEach(System.out::println);
        emps.stream().sorted((e1, e2) -> {
            return e1.getAge().compareTo(e2.getAge());
        }).forEach(System.out::println);

        emps.stream().sorted(Comparator.comparing(Employee::getSalary)).forEach(System.out::println);

    }


    public static Stream<Character> filterCharacter(String str) {
        List<Character> list = new ArrayList<>();

        for (Character character : str.toCharArray()) {
            list.add(character);
        }
        return list.stream();
    }
}
```

-------------------------------------------------------------------
-------------------------------------------------------------------
```
package com.yzhao.cks.stream;

import java.util.*;
import java.util.stream.Collectors;

public class TestStream3 {

    public static void main(String[] args) {
        List<Employee> emps = Arrays.asList(
                new Employee("zyh", 23, 1000.0, Employee.Status.FREE),
                new Employee("lj", 12, 2000.0, Employee.Status.BUSY),
                new Employee("hah", 34, 3000.0, Employee.Status.VOCATION),
                new Employee("xixi", 24, 300.0, Employee.Status.VOCATION),
                new Employee("fefe", 45, 4000.0, Employee.Status.FREE),
                new Employee("zyh1", 24, 1010.0, Employee.Status.FREE),
                new Employee("zyh", 23, 1000.0, Employee.Status.FREE)
        );

        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9);

        //reduce
        Integer sum = list.stream().reduce(30, (x,y) -> x+y);
        System.out.println(sum);

        Optional<Double> op = emps.stream().map(Employee::getSalary)
                .reduce(Double::sum);
        System.out.println(op.get());

        //collect

        List<String> list1 = emps.stream()
                .distinct()
                .map(Employee::getName)
                .collect(Collectors.toList());
        list1.forEach(System.out::println);

        System.out.println("-----------------------");
        HashSet<String> list2 = emps.stream()
                .distinct()
                .map(Employee::getName)
                .collect(Collectors.toCollection(HashSet::new));
        list2.forEach(System.out::println);


        Map<Employee.Status, List<Employee>> map = emps.stream()
                .collect(Collectors.groupingBy(Employee::getStatus));
        System.out.println(map);


        Optional<Integer> count = emps.stream()
                .map(e->1)
                .reduce(Integer::sum);
        System.out.println(count.get());
    }
}

```
    
    

* #### Stream流终止操作
	1. allMatch, anyMatch, noneMatch, 
    2. findFirst, findAny,
    3. count
    4. max, min
    
    
```
package com.yzhao.cks.stream;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Optional;

public class TestStream2 {

    public static void main(String[] args) {
        List<Employee> emps = Arrays.asList(
                new Employee("zyh", 23, 1000.0, Employee.Status.FREE),
                new Employee("lj", 12, 2000.0, Employee.Status.BUSY),
                new Employee("hah", 34, 3000.0, Employee.Status.VOCATION),
                new Employee("xixi", 24, 300.0, Employee.Status.VOCATION),
                new Employee("fefe", 45, 4000.0, Employee.Status.FREE),
                new Employee("zyh1", 24, 1010.0, Employee.Status.FREE)
        );

        //allMatch, anyMatch, noneMatch, findFirst, findAny, count, max, min
        boolean b1 = emps.stream()
                .allMatch(e -> e.getStatus().equals(Employee.Status.FREE));

        System.out.println(b1);

        boolean b2 = emps.stream()
                .anyMatch(e -> e.getStatus().equals(Employee.Status.FREE));
        System.out.println(b2);

        boolean b3 = emps.stream()
                .noneMatch(e -> e.getStatus().equals(Employee.Status.FREE));
        System.out.println(b3);

        Optional<Employee> optional = emps.stream().sorted(Comparator.comparing(Employee::getSalary))
                .findFirst();
        System.out.println(optional.get());

        System.out.println("-----------------------------");

        Optional<Employee> optional1 = emps.parallelStream().filter(e->e.getStatus().equals(Employee.Status.FREE))
                .findAny();
        System.out.println(optional1.get());

        long count = emps.stream().count();
        System.out.println(count);

        Optional<Employee> optional2 = emps.stream().max(Comparator.comparing(Employee::getSalary));
        System.out.println(optional2.get());

        Optional<Employee> optional3 = emps.stream().min(Comparator.comparing(Employee::getSalary));
        System.out.println(optional3.get().getSalary());
    }
}

```


 
* #### Fork/Join 框架和Java8 的并行流

例子： 计算总和

1. Fork/Join 需要扩展RecursiveTask， 来实现fork/join相关操作， 将大任务不断fork成小任务， 直到临界值不再fork， 最后将每个fork的任务执行结果综合起来。

2. 并行流： `parallel`


```
package com.yzhao.cks.stream;

import java.util.concurrent.RecursiveTask;

public class ForkJoinCalculate extends RecursiveTask<Long> {

    private static final long serialVersionUID = 1L;

    private long start;
    private long end;

    private static final long THRESHOLD = 1000L;


    public ForkJoinCalculate(long start, long end) {
        this.start = start;
        this.end = end;
    }


    @Override
    protected Long compute() {
        long length = end - start;

        if (length < THRESHOLD) {
            long sum = 0;

            for (long i=start; i<=end; i++) {
                sum += i;
            }
            return sum;
        }else {
            long middle = (start+end) /2;

            ForkJoinCalculate left = new ForkJoinCalculate(start, middle);
            left.fork();

            ForkJoinCalculate right = new ForkJoinCalculate(middle+1, end);
            right.fork();

            return left.join() + right.join();
        }
    }
}

```
----------------------------------
----------------------------------
----------------------------------

TestForkJoin.java, 这里写了三种方法， 第一种fork/join， 第二种for循环， 第三种java8并行流操作

```
package com.yzhao.cks.stream;

import java.time.Duration;
import java.time.Instant;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;

public class TestForkJoin {

    public static void test1() {
        Instant start = Instant.now();
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinCalculate(0L, 1000000000L);

        long sum = forkJoinPool.invoke(task);
        System.out.println(sum);

        Instant end  = Instant.now();

        System.out.println("ForkJoin Time: " + Duration.between(start, end).toMillis());
    }

    public static void test2() {
        Instant start = Instant.now();
        long sum = 0;
        for (long i=0L; i<=1000000000L; i++) {
            sum += i;
        }
        System.out.println(sum);

        Instant end = Instant.now();
        System.out.println("Time: " + Duration.between(start,end ).toMillis());
    }

    public static void test3() {
        Instant start = Instant.now();

        long sum = LongStream.rangeClosed(0L, 1000000000L)
                .parallel().sum();
        System.out.println(sum);

        Instant end = Instant.now();

        System.out.println("Java8 parallel Time: " + Duration.between(start, end).toMillis());

    }

    public static void main(String[] args) {
        test1();
        System.out.println("---------");
        test2();
        System.out.println("---------");
        test3();
    }
}

```


Result：
当计算的数值足够大， 或者说任务量到达一定程度时， java 8 并行流表现出很高的效率。

```
500000000500000000
ForkJoin Time: 2466
---------
500000000500000000
Time: 1355
---------
500000000500000000
Java8 parallel Time: 495
```


    
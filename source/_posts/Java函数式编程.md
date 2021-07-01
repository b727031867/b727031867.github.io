---
title: Java函数式编程体验
date: 2021-07-15 10:41:10
tags:
- Java 
categories:
- Java
---
# Java 8 函数式编程读书笔记

1. ### 函数式接口：只有单个抽象方法的接口

   1. 使用示例：

      ```java
      /**
       * @author 龚秀峰
       * @version 1.0
       * @date 2020/9/29 22:00
       */
      @FunctionalInterface
      public interface Test<T> {
          //上述定义了一个函数接口，其中有一个方法test，该方法接收两个泛型对象参数，返回一个泛型对象
          T test(T param1,T param2);
          //函数接口中只能定义一个方法，定义多个方法会导致注解FunctionalInterface报错
          //Multiple non-overriding abstract methods found in xxx
          //这是因为，Java中存在的类型重载，会让javac挑选最明确的类型，但是拉姆达表达式是一段代码，而不是一种类型
          //所以无法进行挑选。为了避免歧义，让函数式接口中只能存在一个接口方法
      }
      
      class MyClass implements Test<String>{
      
          public static void main(String[] args) {
              String resStr = new MyClass().test("a", "b");
              //下面想要通过拉姆达表达式创建类或抽象类的对象，让对象具有不同的行为，不允许！
              //new MyClass().test((param1,param2) -> param1 + param2 * param1);
              //会报错：Target type of a lambda conversion must be an interface
              System.out.println(resStr);
              //使用拉姆达表达式，传入另外一种行为，使得接口的ｔｅｓｔ方法具有其他的行为
              Test<Integer> iTest = (param1,param2) -> param1 + param2 * param1;
              Integer res = iTest.test(6, 1);
              System.out.println(res);
          }
      
          //传统的实现接口方式
          @Override
          public String test(String param1, String param2) {
              return param1 + param2;
          }
      }
      
      
      ```

   2. 运行结果：

      ![image-20200929221059823](/assets/image-20200929221059823.png)

   3. 常用的函数接口

      - **Supplier**<T>：无参数，返回一种结果，T为结果类型
      - **Consumer<T>**:接收一个参数，但无返回值，T为参数类型
      - **Function<T,R>**： 接收一种参数，返回一种结果，T为参数类型，R为结果类型
      - **Predicate<T>** ：接收参数，返回Boolean值
      
   4. 高级函数接口：就是定义的参数中包含函数接口，使得高级函数可以传入普通函数（拉姆达表达式）

      ```java
      @FunctionalInterface
      public interface Test<T> {
          //此方法是高级函数，它的第二个参数可以接收一个拉姆达表达式
          T test(T param1,Supplier<T> handler);
      }
      ```

   5. 默认方法：为了解决接口新增方法时，所有实现类都必须实现新增的方法的问题，不实现抽象方法的类则采用默认方法中的实现。下图是jdk中集合遍历的默认实现：

      ![image-20201005222721432](/assets/image-20201005222721432.png)

      默认方法的特点：

      - 需要添加default关键字
      - 只能使用子类的方法修改子类自身，无需知晓子类的具体实现（因为接口本身没有成员变量）
      - 子类中存在该方法，那么会采用子类中的方法（类中重写的方法比默认方法更具体，因此优先级高）
      - 接口可以多继承，多重继承时，出现相同优先级的默认方法，则编译不通过，除非手动选择方法（例如：通过在实现方法中，使用XXX.super关键字，指明XXX是谁，调用的默认方法就是谁的）

   6. 接口中的静态方法：

      1. ```java
         Stream<T> of(T t)//从一个集合中获取流，按照顺序有序生成
         Stream<T> of(T... values)//从数组中获取流，按照顺序有序生成
         ```

      2. ```java
         Stream<T> iterate(final T seed, final UnaryOperator<T> f) //生成流，第一个参数为初始值，第二个参数为根据初始值生成新的初始值，此方法一般要配合limit使用，否则会无限生成流
         Stream<T> generate(Supplier<T> s)//通过函数生成流对象
         ```

      3. ```java
         Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)//连接两个流
         ```

   7. Optional：可以看作是一个值容器，并且能够容纳null值

      1. 用法：
      
         ```java
         class B<T>{
         
             public static void main(String[] args) {
                 B<String> stringB = new B<>();
                 stringB.testOptional("666");
             }
         
             public void testOptional(T obj){
                 //构造一个空optional对象
                 Optional<T> optional = Optional.empty();
                 //将传入对象化为optional对象
                 Optional<T> optional2 = Optional.ofNullable(obj);
                 //判断这两个optional对象是否相同
                 boolean equals = optional.equals(optional2);
                 System.out.println(equals);
                 //判断obj对象的optional对象与obj对象是否相同
                 equals = optional2.equals(obj);
                 System.out.println(equals);
                 //如果optional不为空，那么执行拉姆达表达式的内容
                 optional.ifPresent(System.out::println);
                 //如果optional存在，那么返回optional，否则返回参数中的值
                 T t = optional.orElse(obj);
                 System.out.println("orElse值是：" + t);
                 //如果optional2有值，那么会返回值，否则使用拉姆达表达式返回的值(上面的t)
                 Supplier<T> supplier = ()->t;
                 T t1 = optional2.orElseGet(supplier);
                 System.out.println("orElseGet值是：" + t1);
             }
         }
         ```
      
         
      
      2. 注意：
      
         - 当希望使用Optional作为参数时，最好使用采用重载实现，因为用Optional作为参数会让调用方复杂
         - 当返回值为Optional类型时，调用方必须去处理，以此减少空指针的问题

2. ### 拉姆达表达式

   1. 作用：传递一种行为（代码块），使得同一个函数接口具有不同的行为

   2. 基本语法：

      - 无参拉姆达表达式：

      ```java
      //执行的代码只有一行
      Runnable noParamsForOneRow = ()-> System.out.println("Hello");
      //多行执行代码
      Runnable noParamsForManyRow = ()-> {
          System.out.println("Hello");
          System.out.println("World");
                                         };
      //以上的noParamsForOneRow、noParamsForManyRow作为函数接口，可以传入高阶函数（高阶函数能够接收拉姆达表达式），让高阶函数具有不同的行为。以上方法都没有返回值，相当于重写Runnable中的run方法
      ```

      - 一个或多个参数的拉姆达表达式：

      ```java
      package leetcode.editor.cn;
      
      /**
       * @author 龚秀峰
       * @version 1.0
       * @date 2020/9/29 22:00
       */
      @FunctionalInterface
      public interface Test<T,R,C> {
          R test(T param1,C param2);
      }
      
      class MyClass implements Test<String,Long,Integer>{
      
          public static void main(String[] args) {
              //这里Double作为返回类型，Double、Long作为参数类型，通过拉姆达表达式传入不同的行为
              Test<Double,Double,Long> iTest = (param1,param2) -> param1 + param2 * param1;
              //不省略参数的类型
              Test<Double,Double,Long> iTest2 = (Double param1,Long param2) -> param1 + param2 * param1;
              //多行要执行的语句，依旧是返回Double类型的值
              Test<Double,Double,Long> iTest3 = (Double param1,Long param2) -> {
                  double temp = 0D;
                  temp += param1 + param2 * param1;
                  return temp;
              };
              Double res = iTest.test(6D, 1L);
              System.out.println(res);
          }
      
          //传统实现接口的方式
          @Override
          public Long test(String param1, Integer param2) {
              return (long)(param1.length() + param2);
          }
      }
      ```

      - 方法引用

      ```java
      import org.checkerframework.checker.nullness.qual.Nullable;
      
      import java.util.function.Function;
      import java.util.function.Supplier;
      
      /**
       * @author GXF
       * @version 0.1.0
       * @date 2020-10-05 12:40
       * @since 0.1.0
       **/
      public class Test {
          public static void main(String[] args) {
              //静态方法引用，将test方法的方法体作为run方法的方法体
              Runnable test = A::test;
              new Thread(test).start();
              //构造器引用，将一个参数的构造器作为创建A对象的生产者
              Function<String, @Nullable A> constructor = name -> name != null ? new A(name) : null;
              A a2 = constructor.apply("777");
              assert a2 != null;
              a2.sayName();
              //构造器引用，将默认构造器作为创建A对象的生产者
              A a = A.create(A::new);
              //实例方法引用，将sayName代码块作为run方法的执行体，
              Runnable sayName = a::sayName;
              new Thread(sayName).start();
          }
      }
      
      class A {
          private String name = "默认名称";
      
          public A(String name) {
              this.name = name;
          }
      
          public A() {
      
          }
      
          public static void test() {
              System.out.println("666");
          }
      
          public void sayName() {
              System.out.println(this.name);
          }
      
          public static A create(Supplier<A> constructor){
              return constructor.get();
          }
      
      }
      
      ```

   3. 可用拉姆达表达式解决的问题

      1. 处理基于事件驱动的回调
   2. 实现基于消息传递架构的系统
      3. 进行响应式编程
      
   4. 注意：

      1. **拉姆达表达式中，如果要引用外部的变量，那么该变量必须是*既定事实上不变的***（所以拉姆达表达式中引用外部的变量实际上是引用外部的值）

         ```java
         //例如:下面的拉姆达表达式使用了变量t ，但是t不被修改，因此即使t没有final关键字修饰，也不会报错
         double t = 20D
         Test<Double,Double,Long> iTest3 = (Double param1,Long param2) -> {
                     double temp = 0D;
                  temp += (param1 + param2 * param1) - t;
                     return temp;
                 };
         //下面是报错的情况
         Test<Double,Double,Long> iTest3 = (Double param1,Long param2) -> {
                     double temp = 0D;
             		//这里修改了外部变量t，所以会报错
             		t = 10D
                     temp += (param1 + param2 * param1) - t;
                     return temp;
                 };
         ```

      2. 为何拉姆达表达式中可以省略参数类型呢？

         ```java
         //根据javac中的类型推断，可以从拉姆达表达式的上下文中推断出参数类型、返回值类型，因此可以省略不写参数类型、返回类型
         Test<Double,Double,Long> iTest = (param1,param2) -> param1 + param2 * param1;
         ```

      3. 拉姆达表达式可以看作代码块，通过传递代码块来传递行为，同一个函数接口由于传入了不同的行为产生了不同的功能，这是一种多态的体现，可以用于优化某些设计模式，例如：策略模式、命令模式、观察者模式等，无需创建过多的子类，也能达成同一个接口多种行为的目的，但是，对于复杂的接口，不建议采用拉姆达表达式进行简化，应该使用不同的类来实现接口

3. ### Steam(流)

   1. 作用：用函数式编程更加清晰简洁的操作集合

   2. 特性：
      1. 流不可复用：用于进行过滤、求值操作后，再进行其他操作则会报错
      2. 惰性求值：只有当使用**及时求值**方法的时候，才会真正进行计算
      
   3. 常用流操作：

      1. 过滤器（filter）：用于筛选某些流对象，不会立即求值

      2. 转换器（map）：用于将流对象转换成另外的流对象，一对一转换，不会立即求值

      3. 缩放转换器（flatmap）：用于将流对象转换成另外的流对象，一对多转换，不会立即求值

      4. 累积操作器（reduce）：设置一个初始值，将初始值和流集合中的第一个值进行操作，得到的值作为第二次操作的初始值，与流集合中的第二个值进行操作，以此类推到流集合中最后一个值为止，会及时求值

      5. 收集器（Collect）：设置一种规则，将流集合中的流对象转成某种具体的集合

         ```java
         //简单使用示例：
         public static void main(String[] args) {
                 List<String> list = new ArrayList<>();
                 list.add("a");
                 list.add("a");
                 list.add("b");
                 list.add("c");
                 Stream<String> strStream1 = list.stream();
                 Stream<String> strStream2 = list.stream();
                 //过滤，只留下 a，将a转换a,b，收集新的集合
                 String res = strStream1.filter("a"::contains).map(value -> value + "," + "b").flatMap(value -> {
                     //将a，b分割成list，转成成流对象加入到原有的流集合中
                     String[] temp = value.split(",");
                     List<String> strings = new ArrayList<>();
                     Collections.addAll(strings,temp);
                     return strings.stream();
                     //reduce 将每次操作的字符串加上下一个字符串，第一次操作是 "" + 第一个流对象 == 第二次开始操作的值
                 }).reduce("", (current, next) -> current + ":" + next);
                 System.out.println(res);
                 //直接将流转成具体的字符串集合，并且输出
                 List<String> strList = strStream2.collect(toList());
                 strList.forEach(System.out::println);
             }
         ```

         运行结果：

         ![image-20201007111153823](/assets/image-20201007111153823.png)

   4. 并行流：parallelStream

      1. 使用方法：将使用stream的地方换为paralleStream即可
      2. 使用场景：
         1. 与执行顺序无关：执行顺序有关会严重影响速度
         2. 任务之间应该独立，不竞争同一资源：竞争资源会导致线程安全问题
         3. 是否需要并行：数据量小则不需要，实际情况需要自行进行基准测试，分析
      3. 注意事项：
         1. 并行流采用ForkJoinPool（共享线程池），默认会配分CPU个数的线程，若某个任务很慢，则会堵塞其他想要并行的任务，适合CPU密集型的应用，若应用属于IO密集型，则需要自行修改成其他线程池
         2. 事务会失效：Spring的事务通过ThreadLocal进行维护，而并行流会开出多个线程执行，非主线程没有办法获取事务的上下文，事务便会失效
         3. allMatch：全部满足条件返回true，但是空的流集合调用也是返回true
         4. 线程安全问题：若并行执行的任务会使用临界资源，那么可能会导致线程安全问题
         5. 某些操作在并行流上的性能不高：例如：findFirst、limit等
         6. 关注并行的代价：切分任务、执行任务、合并任务结果，每一步的性能消耗，最终是否导致并行的执行效率不如串行

   5. 其他：

      - 基本类型应该使用其对应的流，避免自动装箱、拆箱，降低性能
      - 流对象的顺序与元素的出现顺序相同，即与集合顺序一致，但是当集合无序，那么生成的流也无序，**流的顺序会影响流操作的效率，也会影响并行操作的结果**

4. ### 重构旧代码与测试拉姆达表达式

   1. 重构候选项

      - 未封装的局部状态：通过拉姆达传入表达式，可以在被调用的方法内部检查状态，无需调用者进行检查

        ```java
        class A {
            private String name = "默认名称";
        
            public A(String name) {
                this.name = name;
            }
        
            public A() {
        
            }
        
            public static void main(String[] args) {
                //此时，需要调用isDebugEnabled检查log对象内部的状态，然后输出666
                A a = new A();
                if(a.checkName()){
                    System.out.println("是默认名称，才执行这个操作1");
                }
                //使用拉姆达表达式进行重构，这样调用方无需检查
                a.checkName(()-> System.out.println("是默认名称，才执行这个操作2"));
            }
        
            public static void test() {
                System.out.println("666");
            }
        
            public boolean checkName() {
                return name.equals("默认名称");
            }
        
            public void checkName(Runnable checker){
                if(this.checkName()){
                    checker.run();
                }
            }
            public void sayName() {
                System.out.println(this.name);
            }
        
            public static A create(Supplier<A> constructor){
                return constructor.get();
            }
        
        }
        ```

      - 代替匿名内部类、方法：例如事件监听器、添加、继承某个类只为重写其中的一个方法等情况

      - 同样的代码出现了多次

   2. 调试

      1. 为了便于调试，可以使用方法引用，而不是直接写拉姆达表达式代码块
      2. Mock测试，可以使用拉姆达表达式模拟不同的替身对象
      3. peek方法，调用该方法可以在该方法中打断点，便于调试，即便该方法是空方法也可以（某些IDE可能不支持空方法打断点），在peek方法中打印输出想要查看的中间结果

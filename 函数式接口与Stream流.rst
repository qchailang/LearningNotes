函数式接口
=========
 概念：有且只有一个抽象方法的接口。（可以有别的方法，如：缺省方法，静态方法，私有方法)
 
 使用场景：作为方法的参数或方法的返回值。
@FunctionalInterface注解用来检测一个接口是否是一个函数式接口。

Lambda表达式有延迟执行的特点。

Predicate
Consumer


Stream流
=======

 Stream属于＂管道＂流，只能被＂消费"一次,＂消费＂过后就会被关闭,想要再次遍历必须重新生成.

两种Stream获取方式:
-----------
#. Collection集合转换成Stream流
   
   .. code:: java

        ArrayList<Integer> list = new ArrayList<>();
        Stream<Integer> stream１ = list.stream();

        HashSet<String> set = new HashSet<>();
        Stream<String> stream2 = set.stream();

        HashMap<String, String> map = new HashMap<String, String>();
        // 获取键，存储到一个set集合中
        Set<String> keySet = map.keySet();
        Stream<String> stream3 = keySet.stream();
        // 获取值，礁到一个Collection中
        Collection<String> values = map.values();
        Stream<String> stream4 = values.stream();
        // 获取键值对，entrySet--键与值的映射关系
        Set<Map.Entry<String, String>> entrySet = map.entrySet();
        Stream<Map.Entry<String, String>> stream5 = entrySet.stream();

#. 数组转换成Stream流
   
   .. code:: java

        // 可变参数转换成Stream流
        Stream<Integer> stream6 = Stream.of(1, 2, 3, 4, 5, 6);
        // 可变参数可以传入数组
        String[] arr = {"a", "bb", "ccc", "ddd"};
        Stream<String> stream7 = Stream.of(arr);

Stream流的操作方法
----------
可分为：

+ **延迟方法**：返回值类型仍然是Stream，因此支持链式调用，除了终结方法外，其余的方法均为延迟方法．stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
+ **终结方法**：返回值类型不是Stream，

#. 延迟方法:
   
   + forEach(Consumer<? super T> action)
      对此流的每个元素执行操作。
   .. code:: java

        List<User> list1 = Arrays.asList(
                // name，age
                new User("张三", 11),
                new User("王五", 20),
                new User("赵六", 91),
                new User("田七", 8),
                new User("李四", 44)
        );
        list1.stream().forEach(u -> System.out.println(u));

   + filter(Predicate<? super T> predicate)
      返回由 与 此给定谓词匹配的 此流的元素 组成的流。(过滤)
   .. code:: java

      list1.stream()
        .filter(u -> u.getAge() > 30)
        .forEach(u -> System.out.println(u));

   + sorted(Comparator<? super T> comparator)
      返回由该流的元素组成的流，根据提供的 Comparator进行排序。
   .. code:: java

      list1.stream()
        .sorted(Comparator.comparing(User::getAge))
        .forEach(u -> System.out.println(u));

   + Stream<T> limit(long maxSize)
      返回由该流的元素组成的流，截断长度不能超过maxSize 。(返回该流前maxSize个元素组成的流)
   .. code:: java

    list1.stream().limit(2).forEach(u -> System.out.println(u));

   * Stream<T> skip(long n)
      与limit互斥，使用该方法跳过n个元素，如果此流包含少于n元素，那么将返回一个空流。
   .. code:: java

    list1.stream().skip(2).forEach(u -> System.out.println(u));

   + map(Function<? super T,? extends R> mapper)
      返回由给定函数应用于此流的元素的结果组成的流。(接收一个方法作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素)
   .. code:: java



#. 终结方法
   
   + max，min，sum，avg，count
      
   .. code:: java

    IntSummaryStatistics num = list1.stream()
        .mapToInt(User::getAge)
        .summaryStatistics();
    System.out.println("总共人数：" + num.getCount());
    System.out.println("平均年龄：" + num.getAverage());
    System.out.println("最大年龄：" + num.getMax());
    System.out.println("最小年龄：" + num.getMin());
    System.out.println("年龄之和：" + num.getSum());
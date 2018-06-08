---
layout: post
title: '《阿里巴巴Java开发手册》读书笔记'
subtitle: 'Java编程规范'
date: 2018-06-07
categories: Java
tags: Java
---

# 常量定义：

1. 【强制】不允许任何魔法值（即未经预先定义的常量）直接出现在代码中。  

     反例：

    ```JAVA
    String key = "Id#taobao_" + tradeId;
    cache.put(key, value);
    ```

2. 【强制】在 long 或者 Long 赋值时，数值后使用大写的 L，不能是小写的 l，小写容易跟数字 1 混淆，造成误解。 说明：Long a = 2l; 写的是数字的 21，还是 Long 型的 2?

3. 【推荐】如果变量值仅在一个固定范围内变化用 enum 类型来定义。 说明：如果存在名称之外的延伸属性应使用 enum 类型，下面正例中的数字就是延伸信息，表 示一年中的第几个季节。 正例：

   ```JAVA
   public enum SeasonEnum {
       SPRING(1), SUMMER(2), AUTUMN(3), WINTER(4);
       private int seq;
       SeasonEnum(int seq){
           this.seq = seq;
       }
   }
   ```

# OOP规约：

1. 【强制】Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals。
   正例："test".equals(object); 反例：object.equals("test"); 说明：推荐使用 java.util.Objects#equals（JDK7 引入的工具类）

2. 【强制】所有的相同类型的包装类对象之间值的比较，全部使用 equals 方法比较。
   说明：对于 Integer var = ?  在-128 至 127 范围内的赋值，Integer 对象是在 IntegerCache.cache 产生，会复用已有对象，这个区间内的 Integer 值可以直接使用==进行 判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑， 推荐使用 equals 方法进行判断。

3. 【强制】定义 DO/DTO/VO 等 POJO 类时，不要设定任何属性默认值。
   反例：POJO 类的 gmtCreate 默认值为 new Date()，但是这个属性在数据提取时并没有置入具 体值，在更新其它字段时又附带更新了此字段，导致创建时间被修改成当前时间。

4. 【强制】序列化类新增属性时，请不要修改 serialVersionUID 字段，避免反序列失败；如果完全不兼容升级，      避免反序列化混乱，那么请修改 serialVersionUID 值。
    说明：注意 serialVersionUID 不一致会抛出序列化运行时异常。

5. 【强制】构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，请放在 init 方法中。

6. 【强制】POJO 类必须写 toString 方法。使用 IDE 中的工具：source> generate toString 时，如果继承了另一个 POJO 类，注意在前面加一下 super.toString。
   说明：在方法执行抛出异常时，可以直接调用POJO的 toString()方法打印其属性值，便于排查问题。

7. 【推荐】使用索引访问用 String 的 split 方法得到的数组时，需做最后一个分隔符后有无内容的检查，否则会有抛 IndexOutOfBoundsException 的风险。
    说明：

   ```JAVA
   String str = "a,b,c,,";
   String[] ary = str.split(",");
   // 预期大于 3，结果是 3
   System.out.println(ary.length);
   ```

8. 【推荐】所有 Service 和 DAO 的 getter/setter 方法放在类体最后。

9. 【推荐】final 可以声明类、成员变量、方法、以及本地变量，下列情况使用 final 关键字：

    1） 不允许被继承的类，如：String 类。

    2） 不允许修改引用的域对象。

    3） 不允许被重写的方法，如：POJO 类的 setter 方法。

    4） 不允许运行过程中重新赋值的局部变量。

    5） 避免上下文重复使用一个变量，使用 final 描述可以强制重新定义一个变量，方便更好地进行重构。

10. 【推荐】慎用 Object 的 clone 方法来拷贝对象。

    说明：对象的 clone 方法默认是浅拷贝，若想实现深拷贝需要重写 clone 方法实现域对象的深度遍历式拷贝。

11. 【强制】如果希望将一个数组的所有值拷贝到一个新的数组中，而不是引用，使用
     Arrays 类的 copyOf 方法。

12. 【推荐】快速打印二维数组的元素列表，可以使用Arrays类的 deepToString 方法。

13. 【推荐】类成员与方法访问控制从严：  

    1） 如果不允许外部直接通过 new 来创建对象，那么构造方法必须是 private。

    2） 工具类不允许有 public 或 default 构造方法。

    3） 类非 static 成员变量并且与子类共享，必须是 protected。

    4） 类非 static 成员变量并且仅在本类使用，必须是 private。  

    5） 类 static 成员变量如果仅在本类使用，必须是 private。  

    6） 若是 static 成员变量，考虑是否为 final。  

    7） 类成员方法只供类内部调用，必须是 private。

    8） 类成员方法只对继承类公开，那么限制为 protected。

    说明：任何类、方法、参数、变量，严控访问范围。过于宽泛的访问范围，不利于模块解耦。 思考：如果是一个 private 的方法，想删除就删除，可是一个 public 的 service 成员方法或成员变量，删除一下，不得手心冒点汗吗？变量像自己的小孩，尽量在自己的视线内，变量作用域太大，无限制的到处跑，那么你会担心的。

# 集合处理

1. 【强制】关于 hashCode 和 equals 的处理，遵循如下规则：

    1） 只要重写 equals，就必须重写 hashCode。

    2） 因为 Set 存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的 对象必须重写这两个方法。

    3） 如果自定义对象作为 Map 的键，那么必须重写 hashCode 和 equals。 说明：String 重写了 hashCode 和 equals 方法，所以我们可以非常愉快地使用 String 对象 作为 key 来使用。

2. 【强制】 ArrayList的subList结果不可强转成ArrayList，否则会抛出ClassCastException 异常，即 java.util.RandomAccessSubList cannot be cast to java.util.ArrayList。

    说明：subList 返回的是 ArrayList 的内部类 SubList，并不是 ArrayList 而是 ArrayList 的一个视图，对于 SubList 子列表的所有操作最终会反映到原列表上。

3. 【强制】在 subList 场景中，高度注意对原集合元素的增加或删除，均会导致子列表的遍历、 增加、删除产生 ConcurrentModificationException 异常。

4. 【强制】使用集合转数组的方法，必须使用集合的 toArray(T[] array)，传入的是类型完全 一样的数组，大小就是 list.size()。

    说明：使用 toArray 带参方法，入参分配的数组空间不够大时，toArray 方法内部将重新分配 内存空间，并返回新数组地址；如果数组元素个数大于实际所需，下标为[ list.size() ] 的数组元素将被置为 null，其它数组元素保持原值，因此最好将方法入参数组大小定义与集 合元素个数一致。 

    正例：

    ```JAVA
    List<String> list = new ArrayList<String>(2);
    list.add("guan");
    list.add("bao");
    String[] array = new String[list.size()];
    array = list.toArray(array);
    ```
    反例：直接使用 toArray 无参方法存在问题，此方法返回值只能是 Object[]类，若强转其它 类型数组将出现 ClassCastException 错误。

5. 【强制】使用工具类 Arrays.asList()把数组转换成集合时，不能使用其修改     集合相关的方 法，它的 add/remove/clear 方法会抛出           UnsupportedOperationException 异常。 说明：asList 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。Arrays.asList 体现的是适配器模式，只是转换接口，后台的数据仍是数组。

    ```JAVA
     String[] str = new String[] { "you", "wu" };
     List list = Arrays.asList(str);
    ```

    第一种情况：list.add("yangguanbao"); 运行时异常。

    第二种情况：str[0] = "gujin"; 那么 list.get(0)也会随之修改。

6. 【强制】泛型通配符<? extends T>来接收返回的数据，此写法的泛型集合不能使用 add 方 法，而<? super T>不能使用 get 方法，作为接口调用赋值时易出错。

    说明：扩展说一下 PECS(Producer Extends Consumer Super)原则：第一、频繁往外读取内 容的，适合用<? extends T>。第二、经常往里插入的，适合用<? super T>。

7. 【强制】不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。

    正例：

    ```JAVA
    List<String> list = new ArrayList<>();
    list.add("1");
    list.add("2");
    Iterator<String> iterator = list.iterator();
    while (iterator.hasNext()) {
        String item = iterator.next();
        if (删除元素的条件) {
            iterator.remove();
        }
    }
    ```

    反例：

    ```JAVA
    for (String item : list) {
        if ("1".equals(item)) {
            list.remove(item);
        }
    }
    ```

    说明：以上代码的执行结果肯定会出乎大家的意料，那么试一下把“1”换成“2”，会是同样的结果吗？

8. 【强制】 在 JDK7 版本及以上，Comparator 实现类要满足如下三个条件，不然 Arrays.sort， Collections.sort 会报 IllegalArgumentException 异常。

    说明：三个条件如下

    1） x，y 的比较结果和 y，x 的比较结果相反。

    2） x>y，y>z，则 x>z。

    3） x=y，则 x，z 比较结果和 y，z 比较结果相同。

    反例：下例中没有处理相等的情况，实际使用中可能会出现异常： 

    ```JAVA
    new Comparator<Student>() {
         @Override
         public int compare(Student o1, Student o2) {
            return o1.getId() > o2.getId() ? 1 : -1;
         }
    };  
    ```
9. 【推荐】集合泛型定义时，在 JDK7 及以上，使用 diamond 语法或全省略。 说明：菱形泛型，即 diamond，直接使用<>来指代前边已经指定的类型。 正例：

    ```JAVA
    // <> diamond 方式
    HashMap<String, String> userCache = new HashMap<>(16);
    // 全省略方式
    ArrayList<User> users = new ArrayList(10);
    ```

10. 【推荐】集合初始化时，指定集合初始值大小。
    说明：HashMap 使用 HashMap(int initialCapacity) 初始化。 正例：initialCapacity = (需要存储的元素个数 / 负载因子) + 1。注意负载因子（即loader factor）默认为 0.75，如果暂时无法确定初始值大小，请设置为 16（即默认值）。

    反例：HashMap 需要放置 1024 个元素，由于没有设置容量初始大小，随着元素不断增加，容 量 7 次被迫扩大，resize 需要重建 hash 表，严重影响性能

11. 【推荐】使用 entrySet 遍历 Map 类集合 KV，而不是 keySet 方式进行遍历。

    说明：keySet 其实是遍历了 2 次，一次是转为 Iterator 对象，另一次是从 hashMap 中取出 key 所对应的 value。而 entrySet 只是遍历了一次就把 key 和 value 都放到了 entry 中，效率更高。

    > 如果是 JDK8，使用 Map.foreach 方法。

    正例：values()返回的是 V 值集合，是一个 list 集合对象；keySet()返回的是 K 值集合，是 一个 Set 集合对象；entrySet()返回的是 K-V 值组合集合。

12. 【推荐】高度注意 Map 类集合 K/V 能不能存储 null 值的情况，如下表格：

    |集合类|Key|Value|Super|说明|
    |:-:|:-:|:-:|:-:|:-:|
    |HashTable|不允许为null|不允许为null|Dictionary|线程安全|
    |ConcurrentHashMap|不允许为null|不允许为null|AbstractMap|锁分段技术（JDK8:CAS）|
    |TreeMap |不允许为null|允许为null|AbstractMap|线程不安全|
    |HashMap|允许为null|允许为null|AbstractMap|线程不安全|

13. 【参考】利用 Set 元素唯一的特性，可以快速对一个集合进行去重操作，避免使用 List 的 contains 方法进行遍历、对比、去重操作。

# 并发处理
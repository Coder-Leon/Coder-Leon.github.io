---
layout: post
title: '《阿里巴巴Java开发手册》读书笔记'
subtitle: 'Java编程规范'
date: 2018-06-07
categories: Java
tags: Java
---

## 常量定义：

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

## OOP规约：

1. 【强制】Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals。 
   正例："test".equals(object); 反例：object.equals("test"); 说明：推荐使用 java.util.Objects#equals（JDK7 引入的工具类）

2. 【强制】所有的相同类型的包装类对象之间值的比较，全部使用 equals 方法比较。 
   说明：对于 Integer var = ?  在-128 至 127 范围内的赋值，Integer 对象是在 IntegerCache.cache 产生，会复用已有对象，这个区间内的 Integer 值可以直接使用==进行 判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑， 推荐使用 equals 方法进行判断。

3. 【强制】定义 DO/DTO/VO 等 POJO 类时，不要设定任何属性默认值。 
   反例：POJO 类的 gmtCreate 默认值为 new Date()，但是这个属性在数据提取时并没有置入具 体值，在更新其它字段时又附带更新了此字段，导致创建时间被修改成当前时间。

4. 【强制】序列化类新增属性时，请不要修改 serialVersionUID 字段，避免反序列失败；如果完全不兼容升级，避免反序列化混乱，那么请修改 serialVersionUID 值。 
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

11. 【推荐】类成员与方法访问控制从严：  

    1） 如果不允许外部直接通过 new 来创建对象，那么构造方法必须是 private。  

    2） 工具类不允许有 public 或 default 构造方法。  

    3） 类非 static 成员变量并且与子类共享，必须是 protected。  

    4） 类非 static 成员变量并且仅在本类使用，必须是 private。  

    5） 类 static 成员变量如果仅在本类使用，必须是 private。  

    6） 若是 static 成员变量，考虑是否为 final。  

    7） 类成员方法只供类内部调用，必须是 private。   

    8） 类成员方法只对继承类公开，那么限制为 protected。 

    说明：任何类、方法、参数、变量，严控访问范围。过于宽泛的访问范围，不利于模块解耦。 思考：如果是一个 private 的方法，想删除就删除，可是一个 public 的 service 成员方法或成员变量，删除一下，不得手心冒点汗吗？变量像自己的小孩，尽量在自己的视线内，变量作用域太大，无限制的到处跑，那么你会担心的。

## 集合处理


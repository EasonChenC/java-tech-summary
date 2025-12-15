# Java Lambda 与 Stream 终极教程：从入门到精通

> “代码是写给人看的，附带能在机器上运行。” —— Hal Abelson

欢迎来到 Java Lambda 和 Stream 的世界！如果你还在为繁琐的匿名内部类和冗长的循环代码而烦恼，那么本教程将是你的救星。自 Java 8 以来，Lambda 表达式和 Stream API 的引入，彻底改变了 Java 开发者处理集合数据的方式，让代码变得前所未有的简洁、优雅和高效。

---

## 目录

1.  [**第一部分：Lambda 表达式 - 代码的“瘦身”魔法**](#第一部分-lambda-表达式---代码的瘦身魔法)
2.  [**第二部分：Stream API - 数据处理的“流水线”**](#第二部分-stream-api---数据处理的流水线)
3.  [**第三部分：Stream 实战演练 - 成为数据处理大师**](#第三部分-stream-实战演练---成为数据处理大师)
4.  [**第四部分：Optional - 告别 `NullPointerException` 的优雅之道**](#第四部分-optional---告别-nullpointerexception-的优雅之道)
5.  [**第五部分：并行流（Parallel Streams）- 释放多核 CPU 的洪荒之力**](#第五部分并行流parallel-streams---释放多核-cpu-的洪荒之力)
6.  [**第六部分：高级实战场景 - 真实世界的挑战**](#第六部分高级实战场景---真实世界的挑战)
7.  [**第七部分：总结与最佳实践**](#第七部分总结与最佳实践)

---

## 第一部分：Lambda 表达式 - 代码的“瘦身”魔法

### 什么是 Lambda？为何需要它？

想象一下，你只是想让一个人帮你跑个腿（比如，实现一个简单的比较逻辑），但按照旧的 Java 规矩，你必须先给他建一个完整的“身份档案”（匿名内部类），写上姓名、住址、职业等一堆繁文缛节。是不是很累赘？

**Lambda 表达式** 就是为了解决这个问题而生的。它允许你传递“行为”本身，而不是繁琐的“身份档案”。简单来说，**Lambda 就是一个没有名字的、可以被传递的函数**。

**使用 Lambda 的核心优势：**

*   **代码更简洁：** 告别匿名内部类的模板代码。
*   **意图更清晰：** 专注于“做什么”，而不是“怎么做”。
*   **函数式编程：** 将函数作为一等公民，可以像变量一样传递和使用。

### Lambda 语法快速上手

Lambda 的语法非常直观，就像一个箭头函数：

`(parameters) -> expression`

或者包含代码块：

`(parameters) -> { statements; }`

*   **`(parameters)`**：方法的参数列表。
*   **`->`**：箭头，Lambda 的标志。
*   **`expression` 或 `{ statements; }`**：方法体。

让我们看几个例子，感受一下它的威力：

**示例1：启动一个线程**

```java
// 传统方式
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("传统方式启动线程...");
    }
}).start();

// Lambda 方式
new Thread(() -> System.out.println("Lambda 方式启动线程...")).start();
```
> 瞬间清爽了有没有！由于 `run` 方法没有参数，所以是 `()`。方法体只有一行，所以 `{}` 和 `return` 都可以省略。

**示例2：列表排序**

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

// 传统方式
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});

// Lambda 方式
Collections.sort(names, (String a, String b) -> a.compareTo(b));

// Lambda 还可以进一步简化（类型推断）
Collections.sort(names, (a, b) -> a.compareTo(b));
```
> 编译器足够聪明，能根据上下文推断出 `a` 和 `b` 的类型是 `String`。

### 函数式接口：Lambda 的“灵魂伴侣”

Lambda 表达式不是凭空存在的，它需要一个“容器”来承载。这个容器就是 **函数式接口（Functional Interface）**。

**函数式接口** 指的是**有且仅有一个抽象方法**的接口。Java 中用 `@FunctionalInterface` 注解来标识（但不是必须的）。

`Runnable`、`Comparator` 都是典型的函数式接口。Lambda 表达式的类型就是它所实现的函数式接口的类型。

Java 8 在 `java.util.function` 包中内置了大量常用的函数式接口，方便我们使用：

*   **`Predicate<T>`**：`boolean test(T t)` - 接收一个参数，返回布尔值（常用于过滤）。
*   **`Function<T, R>`**：`R apply(T t)` - 接收一个参数，返回一个结果（常用于映射转换）。
*   **`Consumer<T>`**：`void accept(T t)` - 接收一个参数，没有返回值（常用于消费数据）。
*   **`Supplier<T>`**：`T get()` - 不接收参数，返回一个结果（常用于提供数据）。
*   **`BinaryOperator<T>`**：`T apply(T t1, T t2)` - 接收两个同类型参数，返回一个同类型结果（常用于聚合）。

![img](https://ask.qcloudimg.com/http-save/yehe-5395074/0cg68fmxc2.png)



 **Java 8 常用函数式接口**：

| 接口                | 参数 | 返回值    | 常用方法 | 作用              |
| ------------------- | ---- | --------- | -------- | ----------------- |
| `Consumer<T>`       | 1个  | 无        | `accept` | 处理数据          |
| `BiConsumer<T,U>`   | 2个  | 无        | `accept` | 处理两个数据      |
| `Function<T,R>`     | 1个  | 1个       | `apply`  | 转换数据          |
| `BiFunction<T,U,R>` | 2个  | 1个       | `apply`  | 合并/计算两个数据 |
| `Supplier<T>`       | 无   | 1个       | `get`    | 生成/提供数据     |
| `Predicate<T>`      | 1个  | `boolean` | `test`   | 条件判断          |





函数式接口要注意：

- 函数式接口里只允许声明一个抽象方法
- 函数式接口里是允许定义默认方法的
- 函数式接口里允许定义静态方法
- 函数式接口里允许定义java.lang.Object里的public方法





### 方法引用：让代码“懒”到极致

当 Lambda 表达式的方法体已经有现成的方法可以实现时，**方法引用（Method Reference）** 可以让你的代码更加简洁。它被认为是 Lambda 的一种语法糖。

主要有四种类型：

1.  **静态方法引用：`ClassName::staticMethodName`**
    ```java
    // Lambda
    Function<String, Integer> strToInt = (s) -> Integer.parseInt(s);
    // 方法引用
    Function<String, Integer> strToIntRef = Integer::parseInt;
    ```

2.  **实例方法引用（特定实例）：`instance::instanceMethodName`**
    ```java
    String str = "hello";
    // Lambda
    Supplier<Integer> len = () -> str.length();
    // 方法引用
    Supplier<Integer> lenRef = str::length;
    ```

3.  **实例方法引用（任意实例）：`ClassName::instanceMethodName`**
    ```java
    // Lambda
    Function<String, String> toUpper = (s) -> s.toUpperCase();
    // 方法引用
    Function<String, String> toUpperRef = String::toUpperCase;
    ```

4.  **构造方法引用：`ClassName::new`**
    ```java
    // Lambda
    Supplier<User> userFactory = () -> new User();
    // 方法引用
    Supplier<User> userFactoryRef = User::new;
    ```

---

## 第二部分：Stream API - 数据处理的“流水线”

如果说 Lambda 是现代化的“零件”，那么 Stream 就是将这些零件组装起来的“自动化流水线”。

### 什么是 Stream？

**Stream（流）** 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种高效、方便的聚合操作（aggregate operations）或大批量数据操作。

**Stream 的核心特性：**

*   **不是数据结构：** 它不存储数据，只是数据源（如集合、数组）的一个视图。
*   **无副作用：** 对 Stream 的任何操作都不会修改其底层的数据源。
*   **惰性求值（Lazy Evaluation）：** 中间操作在终端操作执行之前不会真正执行。就像一个懒汉，不到最后一刻绝不动手。
*   **只能消费一次：** Stream 在终端操作完成后就会被“关闭”，无法重用。就像倒出来的水，泼出去就收不回来了。
*   **支持并行处理：** 可以轻松地切换到并行模式，利用多核 CPU 提升性能。

### Stream 的生命周期：创建 -> 处理 -> 终止

可以把 Stream 的操作想象成一条工厂流水线：

1.  **数据源（Source）：** "原材料"，比如一个 `List` 或 `Array`。
2.  **中间操作（Intermediate Operations）：** "加工站"，可以有多个。每个加工站对流水线上的元素进行处理（如过滤、排序、转换），然后将处理后的元素传递给下一个站。这些操作是**惰性**的。
3.  **终端操作（Terminal Operation）：** "打包出厂"，只能有一个。它会触发前面所有的中间操作，并产生最终结果（如一个新的集合、一个值或无返回值）。

```
List<String> source = ...;

source.stream() // 1. 创建 Stream (数据源)
    .filter(s -> s.startsWith("a")) // 2. 中间操作 (加工站1：过滤)
    .map(String::toUpperCase)     // 2. 中间操作 (加工站2：转换)
    .forEach(System.out::println);  // 3. 终端操作 (打包出厂：打印)
```

### 如何创建 Stream？

创建 Stream 的方式多种多样：

*   **从集合创建：**
    ```java
    List<String> list = Arrays.asList("a", "b", "c");
    Stream<String> stream = list.stream(); // 串行流
    Stream<String> parallelStream = list.parallelStream(); // 并行流
    ```

*   **从数组创建：**
    ```java
    String[] array = new String[]{"a", "b", "c"};
    Stream<String> stream = Arrays.stream(array);
    ```

*   **通过 `Stream.of()`：**
    ```java
    Stream<String> stream = Stream.of("a", "b", "c");
    ```

*   **通过 `Stream.iterate()` 或 `Stream.generate()`（创建无限流）：**
    ```java
    // 0, 2, 4, 6, ...
    Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
    
    // 生成随机数
    Stream<Double> randomStream = Stream.generate(Math::random);
    ```
    > 无限流通常需要搭配 `limit()` 使用，否则会无限执行下去。

---

## 第三部分：Stream 实战演练 - 成为数据处理大师

理论讲完了，是时候亮出真功夫了！让我们通过一系列实战，看看 Stream 如何优雅地解决实际问题。

### 准备工作：我们的“英雄”——`User` 类

我们先定义一个 `User` 类和一些测试数据，这将贯穿我们整个实战环节。

```java
// 为了方便，我们使用 Lombok 简化代码
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
class User {
    private Long id;
    private String name;
    private int age;
    private String city;
    private String department;
}

// 初始化用户列表
List<User> users = Arrays.asList(
    new User(1L, "Alice", 25, "New York", "HR"),
    new User(2L, "Bob", 30, "London", "Tech"),
    new User(3L, "Charlie", 25, "New York", "Tech"),
    new User(4L, "David", 35, "Tokyo", "Finance"),
    new User(5L, "Eve", 30, "London", "HR"),
    new User(6L, "Frank", 40, "Tokyo", "Tech"),
    new User(7L, "Grace", 25, "New York", "Finance"),
    new User(8L, "Bob", 45, "Paris", "Tech") // 重复的名字 Bob
);
```

### 中间操作（Intermediate Operations）：流水线上的加工站

#### `filter()` - 过滤器：留下你想要的

`filter` 接收一个 `Predicate`，用于筛选出满足条件的元素。

**需求：** 找出所有在 "Tech" 部门且年龄大于30岁的员工。

```java
// 传统方式
List<User> result = new ArrayList<>();
for (User user : users) {
    if ("Tech".equals(user.getDepartment()) && user.getAge() > 30) {
        result.add(user);
    }
}

// Stream 方式
List<User> techGurus = users.stream()
        .filter(user -> "Tech".equals(user.getDepartment()))
        .filter(user -> user.getAge() > 30)
        .collect(Collectors.toList());

// techGurus: [Frank(40), Bob(45)]
```

#### `map()` - 映射器：一对一“变身”

`map` 接收一个 `Function`，将流中的每个元素转换为另一种形式。

**需求：** 获取所有员工的姓名列表。

```java
// 传统方式
List<String> names = new ArrayList<>();
for (User user : users) {
    names.add(user.getName());
}

// Stream 方式
List<String> nameList = users.stream()
                             .map(User::getName) // 使用方法引用更简洁
                             .collect(Collectors.toList());

// nameList: ["Alice", "Bob", "Charlie", "David", "Eve", "Frank", "Grace", "Bob"]
```

#### `distinct()` - 去重器：保持唯一

`distinct` 会根据元素的 `equals()` 和 `hashCode()` 方法去除重复的元素。

**需求：** 获取所有不重复的员工姓名。

```java
List<String> distinctNames = users.stream()
        .map(User::getName)
        .distinct()
        .collect(Collectors.toList());

// distinctNames: ["Alice", "Bob", "Charlie", "David", "Eve", "Frank", "Grace"]
```

#### `sorted()` - 排序器：让一切井然有序

`sorted()` 有两种形式：无参（自然排序）和带 `Comparator` 参数（自定义排序）。

**需求：** 按年龄升序排列所有员工，如果年龄相同，则按姓名升序。

```java
List<User> sortedUsers = users.stream()
        .sorted(Comparator.comparing(User::getAge)
                          .thenComparing(User::getName))
        .collect(Collectors.toList());

// sortedUsers 将按年龄和姓名排序
```

#### `limit()` & `skip()` - 截取与跳过：控制流量

*   `limit(n)`：返回一个不超过 `n` 个元素的新流。
*   `skip(n)`：跳过前 `n` 个元素，返回一个新流。

**需求：** 找出年龄最大（排序后）的3名员工。

```java
List<User> top3Oldest = users.stream()
        .sorted(Comparator.comparing(User::getAge).reversed()) // 年龄降序
        .limit(3)
        .collect(Collectors.toList());

// top3Oldest: [Bob(45), Frank(40), David(35)]
```
> `limit` 和 `skip` 经常用于实现分页逻辑。

#### `flatMap()` - 扁平化映射：处理“嵌套”数据

当你的 `map` 操作产生的是一个 Stream（或集合）时，你可能会得到一个“流中流”（`Stream<Stream<T>>`）。`flatMap` 可以将这些内部的流“压平”，合并成一个单一的流（`Stream<T>`）。

**场景：** 假设我们有一个字符串列表，我们想获取所有不重复的字符。

```java
List<String> words = Arrays.asList("Hello", "World");

// 使用 map 会得到 Stream<Stream<String>>
// words.stream().map(word -> Arrays.stream(word.split("")))...

// 使用 flatMap
List<String> uniqueChars = words.stream()
        .map(word -> word.split("")) // 得到 Stream<String[]>
        .flatMap(Arrays::stream)       // 将每个 String[] 变成一个 Stream，然后合并
        .distinct()
        .collect(Collectors.toList());

// uniqueChars: ["H", "e", "l", "o", "W", "r", "d"]
```



| 操作    | 输入      | 转换后结构        | 是否压平 | 结果类型               |
| ------- | --------- | ----------------- | -------- | ---------------------- |
| map     | "a", "bb" | ["A"], ["BB"]     | 否       | Stream<Stream>（嵌套） |
| flatMap | "a", "bb" | ["A"], ["B", "B"] | 是       | Stream（扁平）         |



| 特点     | `map()`         | `flatMap()`                      |
| -------- | --------------- | -------------------------------- |
| 映射关系 | 一对一          | 一对多 或 多对多                 |
| 返回类型 | Stream → Stream | Stream → Stream（扁平后）        |
| 场景     | 简单对象转换    | 嵌套结构的展开、集合的合并处理等 |

### 终端操作（Terminal Operations）：流水线的终点

#### `forEach()` - 遍历：消费每一个元素

这是最简单的终端操作，用于对流中的每个元素执行一个 `Consumer` 动作。

**需求：** 打印出所有员工的姓名。

```java
users.stream()
     .map(User::getName)
     .forEach(System.out::println);
```

#### `collect()` - 收集器：最强大的终结者

`collect` 是一个极其强大的终端操作，它可以将流中的元素转换成各种数据结构，如 `List`, `Set`, `Map` 等。它需要一个 `Collector` 作为参数。`java.util.stream.Collectors` 提供了大量现成的收集器。

**1. 收集到 List/Set/Map**

```java
// 收集到 List (最常用)
List<User> userList = users.stream().collect(Collectors.toList());

// 收集到 Set (自动去重)
Set<String> citySet = users.stream().map(User::getCity).collect(Collectors.toSet());

// 收集到 Map (Key: id, Value: User)
// 如果 key 重复，会抛出 IllegalStateException，需要提供合并函数
Map<Long, User> userMap = users.stream()
        .collect(Collectors.toMap(User::getId, user -> user, (oldValue, newValue) -> newValue));
```

**2. 分组（`groupingBy`）**

这是 `collect` 的明星功能，类似于 SQL 中的 `GROUP BY`。

**需求：** 按城市对员工进行分组。

```java
Map<String, List<User>> usersByCity = users.stream()
        .collect(Collectors.groupingBy(User::getCity));

// usersByCity 的结果：
// {
//   "New York": [Alice, Charlie, Grace],
//   "London": [Bob, Eve],
//   "Tokyo": [David, Frank],
//   "Paris": [Bob]
// }
```

**多级分组：** 按部门分组，再按城市分组。

```java
Map<String, Map<String, List<User>>> multiGroup = users.stream()
        .collect(Collectors.groupingBy(User::getDepartment,
                                        Collectors.groupingBy(User::getCity)));
```

**3. 数据聚合（`summarizingInt`, `joining` 等）**

**需求：** 统计各部门的平均年龄。

```java
Map<String, Double> avgAgeByDept = users.stream()
        .collect(Collectors.groupingBy(User::getDepartment,
                                        Collectors.averagingInt(User::getAge)));
// avgAgeByDept: {HR=27.5, Tech=36.25, Finance=30.0}
```

**需求：** 将所有员工姓名拼接成一个字符串。

```java
String allNames = users.stream()
                       .map(User::getName)
                       .collect(Collectors.joining(", "));
// allNames: "Alice, Bob, Charlie, David, Eve, Frank, Grace, Bob"
```

#### `reduce()` - 归约：将所有元素“浓缩”成一个

`reduce` 操作可以将流中的所有元素组合起来，得到一个单一的结果。

**需求：** 计算所有员工的总年龄。

```java
// 传统方式
int totalAge = 0;
for (User user : users) {
    totalAge += user.getAge();
}

// Stream reduce 方式
// 0 是初始值，(sum, age) -> sum + age 是累加器
int totalAgeByReduce = users.stream()
                            .map(User::getAge)
                            .reduce(0, (sum, age) -> sum + age);
// 或者使用方法引用
totalAgeByReduce = users.stream()
                        .map(User::getAge)
                        .reduce(0, Integer::sum);
```

#### `count()` - 计数器：数一数有多少

返回流中元素的数量。

**需求：** 计算 "New York" 有多少名员工。

```java
long countInNY = users.stream()
                      .filter(u -> "New York".equals(u.getCity()))
                      .count();
// countInNY: 3
```

#### `anyMatch()`, `allMatch()`, `noneMatch()` - 匹配器：判断题大师

这组操作用于检查流中元素是否满足某个 `Predicate`。

*   `anyMatch()`：是否存在**至少一个**元素满足条件？
*   `allMatch()`：是否**所有**元素都满足条件？
*   `noneMatch()`：是否**没有**元素满足条件？

**需求：**
1. 是否有员工来自 "Tokyo"？
2. 是否所有员工的年龄都大于20岁？
3. 是否没有员工来自 "Moscow"？

```java
boolean hasTokyo = users.stream().anyMatch(u -> "Tokyo".equals(u.getCity())); // true
boolean allOver20 = users.stream().allMatch(u -> u.getAge() > 20);      // true
boolean noMoscow = users.stream().noneMatch(u -> "Moscow".equals(u.getCity())); // true
```

#### `findFirst()` & `findAny()` - 查找器：寻宝游戏

*   `findFirst()`：返回流中的第一个元素。
*   `findAny()`：返回流中的任意一个元素（在并行流中性能更好）。

这两个方法都返回一个 `Optional` 对象，因为流可能是空的，找不到任何元素。

**需求：** 找到第一个在 "Tech" 部门工作的员工。

```java
Optional<User> firstTechEmployee = users.stream()
        .filter(u -> "Tech".equals(u.getDepartment()))
        .findFirst();

firstTechEmployee.ifPresent(user -> System.out.println("First tech employee: " + user));
```

---

## 第四部分：Optional - 告别 `NullPointerException` 的优雅之道

`findFirst` 的返回类型 `Optional` 是什么呢？它是 Java 8 引入的一个非常重要的概念，旨在解决 Java 中最臭名昭著的 `NullPointerException`。

### 为什么需要 Optional？

`Optional<T>` 是一个容器类，代表一个值存在或不存在。它就像一个精美的包装盒，里面**可能**装着一个 `T` 类型的值，也**可能**是空的。这样，你就可以在不打开盒子的前提下，安全地检查和处理里面的东西，而不是直接拿到一个可能是 `null` 的“裸”对象。

### Optional 实战指南

**创建 Optional:**

```java
Optional<User> optUser = Optional.of(new User()); // 如果 user 为 null，会立刻抛出 NPE
Optional<User> optUserNullable = Optional.ofNullable(user); // 允许 user 为 null
Optional<User> emptyOpt = Optional.empty(); // 创建一个空的 Optional
```

**使用 Optional:**

```java
Optional<User> optUser = ...; // 假设我们从某个操作得到了一个 Optional

// 1. 检查是否存在
if (optUser.isPresent()) {
    System.out.println(optUser.get()); // 如果存在，用 get() 获取值 (不推荐直接用)
}

// 2. 如果存在，则执行操作 (推荐)
optUser.ifPresent(user -> System.out.println("User name: " + user.getName()));

// 3. 如果为空，则提供一个默认值 (推荐)
User user = optUser.orElse(new User(0L, "Default", 0, "", ""));

// 4. 如果为空，则通过 Supplier 提供一个默认值 (适用于创建成本高的默认对象)
User user2 = optUser.orElseGet(() -> new User(0L, "Default", 0, "", ""));

// 5. 如果为空，则抛出异常 (推荐)
User user3 = optUser.orElseThrow(() -> new IllegalStateException("User not found"));
```
> **黄金法则：** `Optional` 主要用于方法的**返回值**，提醒调用者这个值可能不存在。**不要**用它作为类的字段或方法的参数。

---

## 第五部分：并行流（Parallel Streams）- 释放多核 CPU 的洪荒之力

### 什么是并行流？

并行流就是将一个大的任务拆分成多个小任务，交由多个线程同时执行，最后将结果汇总。这在处理大规模数据集时，可以极大地提升性能。

将一个串行流转换为并行流非常简单：

```java
// 方法一：直接从集合创建
Stream<User> parallelStream = users.parallelStream();

// 方法二：从现有串行流转换
Stream<User> parallelStream2 = users.stream().parallel();
```

**示例：** 并行计算总年龄

```java
int totalAgeParallel = users.parallelStream()
                            .map(User::getAge)
                            .reduce(0, Integer::sum);
```
> 在底层，并行流使用了 Java 7 引入的 Fork/Join 框架。

### 并行流的注意事项

并行流虽好，但并非“银弹”，滥用反而会导致性能下降。

*   **适用场景：**
    *   **大数据量：** 对于少量数据，线程切换的开销可能比计算本身还大。
    *   **计算密集型任务：** 每个元素的操作比较耗时。
    *   **易于拆分的任务：** 比如 `ArrayList` 很容易拆分，而 `LinkedList` 则不然。
*   **线程安全：** 确保你的 Lambda 表达式是**无状态**的。如果你的操作需要修改共享变量（如 `List.add`），你必须自己处理线程同步，但这会严重影响并行性能，甚至导致结果错误。
*   **装箱/拆箱：** 尽量使用原生类型的 Stream（如 `IntStream`, `LongStream`, `DoubleStream`）来避免自动装箱和拆箱带来的性能损耗。
    ```java
    // 更好
    int totalAge = users.stream().mapToInt(User::getAge).sum();
    ```

---

## 第六部分：高级实战场景 - 真实世界的挑战

理论和基础操作都已经掌握，现在让我们进入更贴近真实项目开发的场景，看看 Stream 如何应对更复杂的挑战。

### 准备工作：扩展我们的 `User` 类

为了模拟更真实的场景，我们给 `User` 类增加 `salary`（薪水）和 `skills`（技能列表）字段。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class User {
    private Long id;
    private String name;
    private int age;
    private double salary;
    private String city;
    private String department;
    private List<String> skills;
}

// 更新我们的用户列表
List<User> users = Arrays.asList(
    new User(1L, "Alice", 25, 60000, "New York", "HR", Arrays.asList("Communication", "Recruiting")),
    new User(2L, "Bob", 30, 90000, "London", "Tech", Arrays.asList("Java", "Spring", "Docker")),
    new User(3L, "Charlie", 25, 85000, "New York", "Tech", Arrays.asList("Python", "Machine Learning")),
    new User(4L, "David", 35, 120000, "Tokyo", "Finance", Arrays.asList("Excel", "SQL")),
    new User(5L, "Eve", 30, 75000, "London", "HR", Arrays.asList("Communication", "Management")),
    new User(6L, "Frank", 40, 150000, "Tokyo", "Tech", Arrays.asList("Java", "Microservices", "K8s")),
    new User(7L, "Grace", 25, 110000, "New York", "Finance", Arrays.asList("SQL", "Tableau")),
    new User(8L, "Heidi", 45, 140000, "Paris", "Tech", Arrays.asList("Java", "GCP", "K8s"))
);
```

### 场景一：复杂分组与多重聚合统计

`Collectors.groupingBy` 可以嵌套一个下游收集器（downstream collector），实现非常强大的聚合功能。

**需求：** 按部门分组，并一次性计算出每个部门的：
1.  员工总数
2.  平均年龄
3.  最高薪水
4.  所有员工姓名的拼接字符串

**解决方案：** 我们可以使用 `Collectors.teeing`（Java 12+）或者自定义收集器来实现，但更通用的方法是使用 `Collectors.groupingBy` 结合 `Collectors.summarizingDouble` 等。这里我们用一种更灵活的方式，将结果聚合到一个自定义对象中。

```java
// 先创建一个 DTO (Data Transfer Object) 来存放聚合结果
@Data
@AllArgsConstructor
class DepartmentStats {
    private Long count;
    private Double averageAge;
    private Double maxSalary;
    private String allNames;
}

Map<String, DepartmentStats> statsByDept = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment, 
        Collectors.collectingAndThen(Collectors.toList(), list -> {
            long count = list.size();
            double avgAge = list.stream().mapToInt(User::getAge).average().orElse(0.0);
            double maxSalary = list.stream().mapToDouble(User::getSalary).max().orElse(0.0);
            String names = list.stream().map(User::getName).collect(Collectors.joining(", "));
            return new DepartmentStats(count, avgAge, maxSalary, names);
        })
    ));

// 打印结果
statsByDept.forEach((dept, stats) -> {
    System.out.println("Department: " + dept);
    System.out.println("  Stats: " + stats);
});
```

### 场景二：分区（Partitioning）

分区是分组的一种特殊情况，它使用 `Predicate` 将流中的元素分为两组：`true` 组和 `false` 组。

**需求：** 将所有员工按薪水是否高于10万（100000）分成两组。

**解决方案：** 使用 `Collectors.partitioningBy`。

```java
Map<Boolean, List<User>> partitionedUsers = users.stream()
    .collect(Collectors.partitioningBy(user -> user.getSalary() > 100000));

List<User> highEarners = partitionedUsers.get(true);
List<User> others = partitionedUsers.get(false);

System.out.println("High Earners: " + highEarners.stream().map(User::getName).collect(Collectors.joining(", ")));
System.out.println("Others: " + others.stream().map(User::getName).collect(Collectors.joining(", ")));
```

### 场景三：扁平化与词频统计

这是 `flatMap` 的经典应用场景。

**需求：** 统计在所有员工中，每种技能（skill）出现的次数，并找出最热门的3种技能。

**解决方案：**
1.  使用 `flatMap` 将所有用户的技能列表“压平”成一个技能流。
2.  使用 `groupingBy` 和 `counting` 来统计词频。
3.  对结果进行排序和截取。

```java
Map<String, Long> skillFrequency = users.stream()
    .flatMap(user -> user.getSkills().stream()) // Stream<List<String>> -> Stream<String>
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

// 找出最热门的3种技能
List<String> top3Skills = skillFrequency.entrySet().stream()
    .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
    .limit(3)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());

System.out.println("Skill Frequency: " + skillFrequency);
System.out.println("Top 3 Skills: " + top3Skills);
// Skill Frequency: {Java=3, Communication=2, K8s=2, ...}
// Top 3 Skills: [Java, Communication, K8s] (顺序可能因计数相同而不同)
```

### 场景四：文件 I/O 与流

使用 `java.nio.file.Files.lines` 可以非常方便地将一个文件转换成字符串流进行处理。

**需求：** 假设有一个日志文件 `app.log`，我们需要找出所有包含 "ERROR" 关键字的行，并统计它们的数量。

**解决方案：**

```java
import java.nio.file.Files;
import java.nio.file.Paths;
import java.io.IOException;

// 伪代码，实际运行时需要一个 app.log 文件
// try-with-resources 确保流被正确关闭
try (Stream<String> lines = Files.lines(Paths.get("app.log"))) {
    long errorCount = lines
        .filter(line -> line.contains("ERROR"))
        .peek(System.out::println) // 打印出找到的错误行
        .count();
    System.out.println("Total error lines: " + errorCount);
} catch (IOException e) {
    e.printStackTrace();
}
```

### 场景五：结合 Optional 的安全查找与转换

在实际开发中，我们经常需要根据一个输入去查找数据，然后对数据进行转换，这个过程可能处处是 `null` 陷阱。

**需求：** 创建一个方法，该方法接收一个用户 ID。
1.  如果能找到对应的用户，则返回其大写的用户名。
2.  如果找不到用户，则返回字符串 `"UNKNOWN_USER"`。

**解决方案：** 这个场景完美展示了 `Optional` 的链式调用能力。

```java
public String findUsernameById(Long id) {
    // 模拟数据库查找，返回 Optional<User>
    Optional<User> userOptional = users.stream()
                                       .filter(u -> u.getId().equals(id))
                                       .findFirst();
    
    return userOptional
        .map(User::getName)       // Optional<User> -> Optional<String>
        .map(String::toUpperCase) // Optional<String> -> Optional<String> (大写转换)
        .orElse("UNKNOWN_USER");  // 如果链中任何一步为空，则返回默认值
}

// 测试
System.out.println(findUsernameById(6L));   // 输出: FRANK
System.out.println(findUsernameById(99L));  // 输出: UNKNOWN_USER
```

---


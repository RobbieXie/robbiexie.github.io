---
title: 编程规范
---

## 目录

+ Git分支管理规范
  + 常用分支说明
  + 操作流程实例

+ 设计规范
  + 前言
  + 阿里设计规约

  + 本组UML图示例

+ 阿里巴巴Java开发规范解读

## Git分支管理规范

### 常用分支说明

#### master分支:

+ 线上版本分支, 对应线上的每一个版本, 用tag标记
+ 保护分支, 不允许在该分支直接提代码
+ 稳定长期存在

#### develop分支:

+ 开发分支, 包含所有最新功能和代码, 所有开发基于此分支进行
+ 保护分支, 不允许直接在该分支提代码
+ 开发新功能时, 基于最新的develop分支切出自己的feature分支进行
+ 稳定长期存在

#### feature & fix分支:

+ 新功能分支,  或修复BUG分支, 通常对应Sprint中的一个特定任务
+ 分支命名用 feature-xxx 或 fix-xxx 开头,且有意义的词汇构成
+ comment 包含Sprint中对应任务id 及此分支功能描述
+ 开发完成后, 提交PR(pull request) 到 origin develop分支,  PR merge到develop分支后并删除此分支

#### release分支:

+ 发布分支, 准备上线版本时 使用的分支. 
+ 准备发布新版本时，从develop 切出一个 release 分支，来做发布前的准备
+ 在修复线上紧急BUG时, 从 master 切出一个 release(hotfix)分支, 来做发布前的准备
+ release分支 测试发现的bug直接在 release分支进行修复, 上线后 合并至 master 分支 和 develop分支
+ 上线发布完成后, 可以选择删除release分支

### 操作流程示例

+ 两个长期分支:  master 和 develop.  日常开发中, 逐渐丰富develop分支功能 

<img src="./git-0.png" alt="image-20220316161954036" style="zoom:42%;" />

+ 多人协作开发时,  提交PR前先在本地 pull并rebase最新的远端develop分支,   git pull origin develop --rebase;

<img src="./git-1-1.png" alt="image-20220318105311716" style="zoom:35%;" />

+ 开发过程中准备一些新功能上线. 首先, 团队一起挑选出要上线的commit点

<img src="./git-1.png" alt="image-20220316160232302" style="zoom:50%;" />

+ 从该commit切出release分支 , 发布到UAT环境 并交给测试和业务同事验证功能, 如符合预期 则使用该版本应用包上线. 上线完成后合并至 master 分支.

<img src="./git-miss.png" alt="image-20220316160610988" style="zoom:50%;" />

+ 若该release分支测试时出了问题, 则直接在该分支修复问题. 知道符合预期后 发布生产, 并将新改动合并至master和develop分支.

<img src="./git-3.png" alt="image-20220316160903903" style="zoom:50%;" />

+ 若从develop分支切release分支时, 有不想跟随本次上线的commit, 使用git cherry-pick精确选择要上线的commit, 从而产出符合预期的release分支.

<img src="./git-4.png" alt="image-20220316161230112" style="zoom:50%;" />

+ 当生产环境出现要紧急修复的BUG时, 从线上对应的master分支commit 切出release分支, 修复bug后上线发布, 完成后合并至master和develop.

<img src="./git-5.png" alt="image-20220316161359901" style="zoom:50%;" />

## 设计规范

### 前言

此处谈不上是规范, 希望大家一起探讨,总结出工作中必要的一些设计及文档产出. 

+ 比较复杂的需求开始前, 往往需要提前进行方案设计和评审, 那我们要评审些什么?  评审产出什么样的结果?
  + 评估需求实现的技术方案&细节&可行性 -> 如何描述技术方案?  
  + 方案带来的改动点 -> 参与方对改动内容达成一致
  + 不同方案的优缺点&成本 -> 拍板确定实施方案
  + 优雅的设计细节 -> 保证复用性和拓展性
  + ...
+ 代码或项目开发完成后,  如何验证符合预期?  -> 遵循文档约定

### 阿里设计规约

![image-20220318111434810](./design-1.png)

### 本组UML图示例

我们组开发的一个电话系统

#### 1. 一通电话生命周期的状态图:

![image-20220318112715504](./design1.png)

#### 2. 通话发起到接听的时序图

![image-20220318112918395](./design2.png)

#### 3. 与协作方外部系统交互的 活动图

<img src="./design3.png" alt="image-20220318113905621" style="zoom:55%;" />

#### 4. UAT测试环境各个服务间的部署图

![image-20220318113430160](./design4.png)

## 阿里巴巴 Java开发手册解读

手册下载链接: https://github.com/alibaba/p3c

### 难点解读:

示例代码源码:  https://github.com/RobbieXie/p3c_guideline_code_demo

##### 命名规约-9: 【强制】POJO 类中的任何布尔类型的变量，都不要加 is 前缀，否则部分框架解析会引起序列化错误。

```java
public class NameGuideline9 {

    public static void main(String[] args) throws JsonProcessingException {
        Record record = Record.builder()
                .isDeleted(true)
                .build();

        ObjectMapper objectMapper = new ObjectMapper();
        Gson gson = new Gson();

        // jackson默认输出 {"deleted":true}
        System.out.println(objectMapper.writeValueAsString(record));
        // gson默认输出 {"isDeleted":true}
        System.out.println(gson.toJson(record));
    }

    @Data
    @Builder
    public static class Record {
        private boolean isDeleted;
    }
}
```

##### 集合规约-2: 【强制】判断所有集合内部的元素是否为空，使用 isEmpty() 方法，而不是 size() == 0 的方式。

```java
// 在某些集合中，前者的时间复杂度为 O(1)，而且可读性更好。
// 比如 ConcurrentLinkedQueue

    // size()方法
    public int size() {
        int count = 0;
        for (Node<E> p = first(); p != null; p = succ(p))
            if (p.item != null)
                // Collection.size() spec says to max out
                if (++count == Integer.MAX_VALUE)
                    break;
        return count;
    }
    
    // isEmpty()方法
    public boolean isEmpty() {
        return peekFirst() == null;
    }
```

##### 集合规约-3&4: 【强制】在使用 java.util.stream.Collectors 类的 toMap() 方法转为 Map 集合时，一定要使用参数类型 为 BinaryOperator，参数名为 mergeFunction 的方法，否则当出现相同 key 时会抛出 IllegalStateException 异常。【强制】在使用 java.util.stream.Collectors 类的 toMap() 方法转为 Map 集合时，一定要注意当 value 为 null 时会抛 NPE 异常。

```java
public class CollectionGuideline4 {

    public static void main(String[] args) {
        ArrayList<User> users = Lists.newArrayList(
                new User("xietiandi", "male"),
                new User("xietiandi", "male"),
                new User("xtd", null)
        );

        // 因为有重复key value， 会抛出IllegalStateException异常
        users.stream().collect(Collectors.toMap(User::getName, User::getSex));

        // 因为有value为null的元素， 会抛出NPE
        users.stream().collect(Collectors.toMap(User::getName, User::getSex, (v1, v2) -> v2));

        // 有重复元素且允许value为null时的正确写法
        users.stream().collect(HashMap::new,
                (hashMap, user) -> hashMap.put(user.getName(), user.getSex()),
                HashMap::putAll).forEach((key, value) -> System.out.println(key + ":" + value));
    }

    @AllArgsConstructor
    @Getter
    public static class User {
        private String name;
        private String sex;
    }
}
```

##### 集合规约-12:  【强制】泛型通配符<? extends T>来接收返回的数据，此写法的泛型集合不能使用 add 方 而<? super T>不能使用 get 方法，两者在接口调用赋值的场景中容易出错。

```java
public class CollectionGuideline12 {

    public static void main(String[] args) {
        List<Animal> animals = new ArrayList<>();
        List<Dog> dogs = new ArrayList<>();
        List<Husky> huskies = Lists.newArrayList(new Husky(), new Husky());
        List<Corgi> corgis = Lists.newArrayList(new Corgi(), new Corgi());

        // addAll 可以将Dog任意子类加入到List<Dog>中
        addAll(dogs, huskies);
        addAll(dogs, corgis);

        // addAll 也可以将Dog任意子类加入到List<Animal>中
        addAll(animals, huskies);
        addAll(animals, corgis);
    }

    /**
     * 将 任何Dog或其子类的List对象 添加到 任何 Dog或其父类的List中
     *
     * @param dest
     * @param src
     */
    public static void addAll(List<? super Dog> dest, List<? extends Dog> src) {
        dest.addAll(src);
    }

    private static class Animal {}
    private static class Dog extends Animal {}
    private static class Husky extends Dog {}
    private static class Corgi extends Dog {}

}
```

##### 并发规约-6: 【强制】必须回收自定义的 ThreadLocal 变量，尤其在线程池场景下，线程经常会被复用，如果不清理 自定义的 ThreadLocal 变量，可能会影响后续业务逻辑和造成内存泄露等问题。尽量在代理中使用 try-finally 块进行回收。

+ 注意设置java启动参数 -Xms15m -Xmx15m 限制堆大小15M;

```java
public class ConcurrentGuideline6 {

    /**
     * supplier supplies 5MB buffer
     */
    private static final ThreadLocal<ByteBuffer> BYTE_BUFFER_THREAD_LOCAL =
            ThreadLocal.withInitial(() -> ByteBuffer.allocate(5 * 1024 * 1024));

    private static final ThreadLocal<ByteBuffer> BYTE_BUFFER_THREAD_LOCAL_2 =
            ThreadLocal.withInitial(() -> ByteBuffer.allocate(5 * 1024 * 1024));

    private static final ThreadLocal<ByteBuffer> BYTE_BUFFER_THREAD_LOCAL_3 =
            ThreadLocal.withInitial(() -> ByteBuffer.allocate(5 * 1024 * 1024));

    public static void main(String[] args) {
        System.out.println("main-thread...");

        /**
         * Runnable1逻辑：
         * 1. 获取5mb的ThreadLocal1对象值
         * 2. 打印日志
         * 3. 线程强制waiting 2秒
         * 4. 手动清除ThreadLocal
         */
        Runnable runnable = () -> {
            BYTE_BUFFER_THREAD_LOCAL.get();
            System.out.println(Thread.currentThread().getName() + "get ByteBuffer 1");
            LockSupport.parkNanos(Duration.ofSeconds(2).toNanos());
            BYTE_BUFFER_THREAD_LOCAL.remove();
        };

        // Runnable2 获取其他ThreadLocal对象：ThreadLocal2
        Runnable runnable2 = () -> {
            BYTE_BUFFER_THREAD_LOCAL_2.get();
            System.out.println(Thread.currentThread().getName() + "get ByteBuffer 2");
            LockSupport.parkNanos(Duration.ofSeconds(2).toNanos());
            BYTE_BUFFER_THREAD_LOCAL_2.remove();
        };

        // Runnable3 获取其他ThreadLocal对象：ThreadLocal3
        Runnable runnable3 = () -> {
            BYTE_BUFFER_THREAD_LOCAL_3.get();
            System.out.println(Thread.currentThread().getName() + "get ByteBuffer 3");
            LockSupport.parkNanos(Duration.ofSeconds(2).toNanos());
            BYTE_BUFFER_THREAD_LOCAL_3.remove();
        };

        // 单线程线程池
        ThreadPoolExecutor threadPoolExecutor =
                new ThreadPoolExecutor(1, 1, 0L, TimeUnit.SECONDS,
                        new LinkedBlockingDeque<>(10), Executors.defaultThreadFactory());

        // 在限制heapSize是15M的情况下，如不remove ThreadLocal, 会OOM
        threadPoolExecutor.execute(runnable);
        threadPoolExecutor.execute(runnable2);
        threadPoolExecutor.execute(runnable3);
    }
}
```


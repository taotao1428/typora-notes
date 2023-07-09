# 跨越DDD从理论到落地的鸿沟



## DDD核心

### 统一语言

统一语言是指每个人对事物的理解是一致，这样方便大家开发与交流。下表是博客系统中的一些概念，统一语言要求对概念理解的一致，并不要求准确性。例如【写博客的人】不一定要用【作者】表示，可以直接使用【用户】表示，只要大家达成一致即可

| 英文     | 中文 | 解释                                                 |
| -------- | ---- | ---------------------------------------------------- |
| Author   | 作者 | 写博客的人                                           |
| Article  | 文章 | 博客中的文章，和博客同义                             |
| Category | 分类 | 分类用来管理文章，一篇文章只能属于一个分类           |
| Tag      | 标签 | 分类的另一种方式，一篇文章可以有0到多个标签          |
| Comment  | 评论 | 可以对文章进行评论，可以对评论进行评论，评论可以盖楼 |



### 领域建模

领域建模是指根据系统的用例（UseCase）或者用户故事（UserStory）提取系统的模型。



> **小明**晚上**10点**，写了一篇**《DDD实战》**，**分类**到技术文章，并打上了DDD**标签**，有人**评论**说：写的很好



| 对象        | 模型 |
| ----------- | ---- |
| 小明        | 作者 |
| 10点        | 时间 |
| 《DDM实战》 | 文章 |
| 分类        | 分类 |
| 标签        | 标签 |
| 评论        | 评论 |



### 领域边界（界限上下文）

在实践过程中，通常我们会为每个子域创建一个领域边界（界限上下文）。统一语言和领域建模都是在特定领域边界内进行。例如下图中，在不同领域边界内，Account的含义是不同，在银行上下文中关注的是钱；文学上下文关注的是文章。

<img src="/其他知识点/DDD/.assert/跨越DDD从理论到落地的鸿沟/image-20221015214101515.png" alt="image-20221015214101515" style="zoom:50%;" />







## 工程落地



### 统一语言

知行合一，保证设计和实现一致。在需求设计时，需要使用统一的语言输出设计文档。在实际开发时，也要使用设计文档一致语言落实到代码中。例如：使用Author表示作者，那在文档中和代码中，都应该使用Author表示作者





### 上下结合

DDD的架构分为应用层和领域层，在工程落地中，应该循序渐进，将应用层的部分逻辑沉淀到领域层。领域能力特点：

1. 内聚性，里面应该仅包含领域内部的逻辑；
2. 复用性，里面应该包含通用的逻辑，而不是某种常见特定下的逻辑，并且逻辑的粒度要适当小一些；
3. 可理解性（业务语义表达），即从方法名称就能看到方法的功能。



<img src="/其他知识点/DDD/.assert/跨越DDD从理论到落地的鸿沟/image-20221016151514441.png" alt="image-20221016151514441" style="zoom:50%;" />





在DDD中，应用层的代码主要是做任务编排。**应该根据行为的层次，将所有行为结构化成一个树形结构，不应该将所有的行为平铺在一个层次**。如下图，一个用户case可以拆分成不同的Phase，Phase可以继续拆分成Step，如果Step逻辑仍比较复杂，可以继续拆分。对于应用层的重复逻辑可以考虑下沉到领域层。



<img src="/其他知识点/DDD/.assert/跨越DDD从理论到落地的鸿沟/image-20221016161110077.png" alt="image-20221016161110077" style="zoom:50%;" />





#### 领域能力内聚的好处

1. 增强代码可维护性。逻辑下移到领域层后，将不会分散的领域层的不同地方，不会出现霰弹式修改。
2. 增强代码可理解性。当领域内的概念放置到领域层，会更加好理解



例子：有一个客户管理系统，有一个新的业务规则，从3月份起，注册资本在100万以上的公司认为是大客户。

如果判断逻辑直接加到业务代码中，那代码中将会出现很多下面的代码片段。

<img src="/其他知识点/DDD/.assert/跨越DDD从理论到落地的鸿沟/image-20221016163007641.png" alt="image-20221016163007641" style="zoom:50%;" />

因为大客户其实是属于客户领域内的知识，如果将判断逻辑下移到领域模型，领域模型只需要增加一个方法isBigClient()，应用层调用该方法即可判断用户是否为大客户，会让业务代码更直观。

<img src="/其他知识点/DDD/.assert/跨越DDD从理论到落地的鸿沟/image-20221016163538908.png" alt="image-20221016163538908" style="zoom:50%;" />

<img src="/其他知识点/DDD/.assert/跨越DDD从理论到落地的鸿沟/image-20221016163555755.png" alt="image-20221016163555755" style="zoom:50%;" />





#### 哪些业务逻辑属于领域能力

实体应该是贫血模式或者充血模型，不应该是失血模型或者胀血模型



领域能力的种类

1. 只有getter和setter能力。
2. 原子操作。原子操作是包含少量业务逻辑的查询、修改和检验等类型的方法。
3. 组合业务逻辑。就是能独立完成某一个功能的所有操作，包括调用外部依赖的方法。
4. 实体间操作，持久化。是指在一个实体内，可以对其他实体进行操作和持久化。这一部分能力应该放在服务中，不能放在实体中。



```java

public static class Customer {
  private long id;
  private long registeredCapital;
  private String email;

  private List<Customer> friends;

  private HealthCodeGateway healthCodeGateway;
  private PayGateway payGateway;
  private ExpressGateway expressGateway;
  private EmailGateway emailGateway;
  private CustomerGateway customerGateway;

  // Getter/Setter
  public long getRegisteredCapital() {
    return registeredCapital;
  }

  // 原子操作（不涉及外部调用，推荐放在实体中）
  public boolean isBigClient() {
    return registeredCapital >= 10000000;
  }

  // 原子操作（涉及外部调用，推荐放在领域服务中）
  public void checkHealthCode() {
    String healthCodeColor = healthCodeGateway.getHealthCodeColor(id);

    if ("green".equals(healthCodeColor) || ("yellow".equals(healthCodeColor) && isBigClient())) {
      return;
    }

    throw new BizException("health code is invalid");
  }

	// 组合业务逻辑，涉及多个操作（推荐放在领域服务中）
  public void buy(List<Goods> goodsList) {
    int totalMoney = 0;
    for (Goods goods : goodsList) {
      totalMoney += goods.getPrice();
    }
		// 大客户打折
    if (isBigClient()) {
      totalMoney *= 0.8;
    }

    // 支付订单
    payGateway.pay(id, totalMoney);
    // 创建快递
    String expressId = expressGateway.create(id, goodsList);
    // 发送邮件通知客户
    emailGateway.send(email, "Id of express is " + expressId);
  }

  // 实体之间相互操作和持久化（这种方法不应该出现实体中，应该放在领域服务中）
  public void addFriend(Customer friend) {
    if (friends.contains(friend)) {
      return;
    }

    friends.add(friend);
    friend.friends.add(this);

    customerGateway.save(this);
    customerGateway.save(friend);
  }
}
```





<img src="/其他知识点/DDD/.assert/跨越DDD从理论到落地的鸿沟/image-20221016171225192.png" alt="image-20221016171225192" style="zoom:50%;" />






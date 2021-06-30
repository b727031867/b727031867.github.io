---
title: Spring事务
date: 2021-06-29 12:41:10
tags:
- Spring
categories:
- Spring
---
# Spring事务

### 特性：ACID

- 原子性：事务执行的基本单位，保证一个或多个操作**要么全部成功，要么全部失败**，没有中间状态，失败则回滚至初始状态。
- 一致性：**指系统从一个正确的状态,迁移到另一个正确的状态**。这是通过AID来保证的，即事务同时具备原子性、隔离性、持久性，才能保证事务的一致性。
- 隔离性：不同的事务在提交以前应当对其他事务不可见，但是根据实际情况会分为以下四种隔离级别：
    - 脏读：事务A读取了事务B的中间值，导致数据运算结果不准确。
    - 不可重复读（重点在于修改）：事务A多次读取结果不同，因为事务B在事务A读取的间隙修改了数据。
    - 幻读（重点在于新增与删除）：与不可重复读类似，事务A查询期间事务B插入了数据，事务A再次查询发现多了几条数据（两次数据查询都在同一个事务中）
    - 修改丢失：事务A与事务B同时修改同一数据，导致后完成的事务覆盖之前事务的结果。
- 持久性：事务执行后涉及的数据将存入存储系统，不会回滚。

### 隔离级别

1. **TransactionDefinition.ISOLATION_DEFAULT**：采用数据库默认的隔离级别。MySQL为可重复读，Oracle为采用可提交读。此为默认设置
2. **TransactionDefinition.ISOLATION_READ_UNCOMMITTED**：读未提交数据，可以看见其他事务处理中的数据，隔离级别最低
3. **TransactionDefinition.ISOLATION_READ_COMMITTED**：可以读已提交数据（可提交读），能够在A事务中读取B事务的修改与插入的数据。
4. **TransactionDefinition.ISOLATION_REPEATABLE_READ**：可重复读，能够在A事务中读取B事务的插入的新数据，无法读取B事务中对旧数据的更新。
5. **TransactionDefinition.ISOLATION_SERIALIZABLE**：串行化执行，隔离级别最高

- 不同隔离级别对应的并发问题：（表格中，“是”说明存在该类问题）
    - 一类更新：回滚丢失（Lost update）：对同一数据，事务A与事务B分别执行，事务B提交完毕，事务A回滚，结果回滚撤销了事务B的提交（就像事务B没有执行），**所有隔离级别都不允许回滚丢失**。
    - 二类更新：多次更新丢失（Second lost update）：事务A与事务B对同一数据进行修改，事务A修改后提交，但事务B的修改基于修改前的值，导致事务B修改后覆盖了事务A提交的值。

| 隔离级别 | 一类更新 | 二类更新 | 脏读 | 不可重复读 | 幻读 |
| :------: | :------: | :------: | :--: | :--------: | :--: |
| 读未提交 |    否    |    是    |  是  |     是     |  是  |
| 读已提交 |    否    |    是    |  否  |     是     |  是  |
| 可重复读 |    否    |    否    |  否  |     否     |  是  |
|  串行化  |    否    |    否    |  否  |     否     |  否  |

### 传播特性（7种）

- 支持当前事务：
    1. PROPAGATION_REQUIRED：当前存在事务则沿用，否则开启新事务
    2. PROPAGATION_SUPPORTS：事务不做强制要求（有就用事务，没有就不用）
    3. PROPAGATION MANDATORY：有事务则加入事务，没有事务则抛异常
- 不支持当前事务：
    1. PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
    2. PROPAGATION_NOT_SUPPORTED：非事务运行，若存在事务，则把当前事务挂起。
    3. PROPAGATION_NEVER：用非事务模式运行，存在事务则抛异常
- 其他:
    1. ROPAGATION_NESTED：存在事务，则开启一个子事务（子事务回滚，不影响外部事务，而外部事务回滚会导致子事务回滚），当前没有事务则等同于PROPAGATION_REQUIRED

### 事务的运行状态

**TransactionStatus接口**的相关方法与属性，除此之外可以设置事务超时时间以及是否是只读事务（增加性能）

- **isNewTransaction**(); // 是否是新的事物
- **hasSavepoint**(); // 是否有恢复点
- **isRollbackOnly**(); // 是否为只回滚
- **boolean**  **isCompleted**()：是否完成

### 编程风格

- 声明式事务：通过Spring AOP 实现
  - 
- 编程式事务

### 事务不生效常见原因

1. 当前对象未被SpringIoC容器管理

   解决方法：注入当前对象到Spring的IoC容器中

2. 数据库存储引擎不支持事务，例如MySQL的MyISAM存储引擎

   解决方法：若业务允许，使用InnoDB存储引擎，其他数据库类似，选择支持事务的存储引擎

3. 当前Bean被过早的初始化，导致未执行postProcessAfterInitialization方法生成代理对象，因此事务无法生效

   例如：若实现BeanPostProcessor和PriorityOrdered接口，则在依赖此BeanPostProcessor的`@Configuration`类中，其他依赖的Bean都会被过早初始化

   解决办法：

    1. BeanPostProcessor单独在某个`@Configuration`中使用
    2. 使用static 方法注册后置处理器（BeanPostProcessor）

4. 方法非Public，`@Transactional` 只能用于 public 的方法上

   解决方法：若要代理非Public方法开启事务，需开启 `AspectJ` 代理模式

5. 自身调用

   ```java
   @Service
   public class ServiceImpl implements Service {
   
       @Transactional
       public void test1() {
           test2();
       }
   
       @Transactional(propagation = Propagation.REQUIRES_NEW)
       public void test2() {
           // 数据库操作
       }
       
       public void test3() {
           test4()
       }
       
       @Transactional
       public void test4() {
           // 数据库操作
       }
   
   }
   ```

   上述代码中，调用test1()事务不会生效，调用test3()事务也不生效，由于这些调用是在同一个类内部进行调用，并未使用代理对象进行方法调用，因此事务不会生效

   解决办法：

    1. 在当前类中注入这个类本身，然后再用注入的对象调用方法
    2. 配置<aop:aspectj-autoproxy expose-proxy=“true”> ，然后通过代理对象调用方法 ((Service)AopContext.currentProxy()).test1()，此时可以触发传播事务PROPAGATION_REQUIRES_NEW，即test1() 和 test3()方法会在两个不同的事务中运行，test3()异常后回滚不会影响test1()的事务

6. 事务管理器未配置PlatformTransactionManager 未被注入到当前Ioc容器中

   ```java
   @Bean
   public PlatformTransactionManager transactionManager(DataSource dataSource) {
       return new DataSourceTransactionManager(dataSource);
   }
   ```

   解决方法：根据需求注入需要的事务管理器

7. 事务传播特性设置错误

   若设置NOT_SUPPORTED传播级别，则若存在其他事务，则会被挂起，当前方法中执行的各种数据操作都不会被事务管理，方法执行完毕后再恢复事务。

   解决方法：正确设置传播特性

8. 异常未正确抛出

   解决方法：检查代码，捕获异常后，需要判断是否需要进行事务的回滚，若需要回滚，则该异常要抛出

9. 事务回滚异常设置错误

   解决方法：@Transactional(rollbackFor = Exception.class)，设置后，包括非RuntimeException异常也会回滚

![](https://img.springlearn.cn/blog/learn_1629295333000.png)

## 一、前言

本系列文章主要的目的是提高大家对代码的单测意识, 其中文章主要会分享单测过程中,常见的测试场景及这
些场景的解决方案和处理思路。 为了能使大家更好的了解单元测试,作为程序员首先从源码入手,分享JUnit
的运行原理。在先了解了JUnit的原理后,再来回顾我们的问题场景, 就自然而然的从根源深处解决大家的测
单痛点以及大家对单测框架不熟悉的情况。


**首先小编会提出几个问题,大家带着问题来思考。收获肯能会更大一些。**

## 二、源码分析适配不通

### 2.1 ① 谁在调用JUnit

*基于SpringBoot 2.1.x版本分析,当点击了执行单例,发生了什么事情?*

你是否也会有这样的疑问呢？其实这个问题很简单我们就能知道,你知道怎么做吗？学习任何源码都可以这样操作。
跟小编这样任意点击一个单侧方法,执行debug。然后打开堆栈信息,看下调用关系。就可以了。
如下图一样。

![](https://img.springlearn.cn/blog/learn_1617790044000.png)

可以看到idea会将单侧的类和方法传递给JUnit。最终由

![](https://img.springlearn.cn/blog/learn_1630067874000.png)

可以看到最终是由 AllDefaultPossibilitiesBuilder 来进行了承接 。所以到这里我们就找到了入口。后续所有的能力,都要从JUnit中去寻找了。

``` 
 @Override
    public Runner getRunner() {
        if (runner == null) {
            synchronized (runnerLock) {
                if (runner == null) {
                    runner = new AllDefaultPossibilitiesBuilder(canUseSuiteMethod).safeRunnerForClass(fTestClass);
                }
            }
        }
        return runner;
    }
```

现在你知道谁在调用JUnit了吗? 知道后我们就继续看Junit的源码吧。这里可以自己思考下idea是如何做到呢？

小编说下小编的思考: idea是用Java来实现的,那么他调用JUnit,其实也就像mysql驱动一样,源码只引用了接口，具体
谁实现JUnit的接口,就看你引入的jar包了。这项技术名叫: jndi(Java Naming and Directory Interface）。
感性去自己在去百度，小编只做到抛砖引玉的作用。天下代码一大抄，看你会抄不会抄，抄来抄去有提高，看你会抄不会抄。

### 2.2 ② 如何知道是否依赖Spring容器

你是否在写SpringBoot单元测试时候,不知道如何来启动SpringBootTest容器呢? 在SpringBoot2.1.x版本前SpringBoot
还没有那么智能。

``` 
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {Application.class}) // 指定启动类
public class BaseApplicationTest {
}
```

如果不这样写 `@RunWith(SpringRunner.class)`, 默认使用 BlockJUnit4ClassRunner 来进行运行。即不依赖容器。 假如说如果需要容器怎么办呢 ? 基于SpringBoot 2.1.x版本分析


- SpringRunner告诉JUnit要使用Spring容器
- SpringBootTest告诉JUnit容器的引导类是这个

JUnit是如何实现的呢?

![](https://img.springlearn.cn/blog/learn_1617791013000.png)

前面启动类中我们使用的注解是 @RunWith 和 @SpringBootTest 那么哪里来解析这个的呢?

![](https://img.springlearn.cn/blog/learn_1617791209000.png)

由此 JUnit 知道要使用 SpringRunner 进行引导。

由上图我们知道 SpringRunner 实例化的入参就是当前的测试类。那么后续所有的奥妙就在这里了。 我们跟进构造往下追究。

![](https://img.springlearn.cn/blog/learn_1617795279000.png)

BootstrapUtils#resolveTestContextBootstrapper 拿到SpringBoot的测试引导类 SpringBootTestContextBootstrapper

![](https://img.springlearn.cn/blog/learn_1617795346000.png)


拿到SpringBoot容器的启动 Main 函数。

到此已经拿到了所有的SpringBoot容器启动参数了。这部分的调用链特别的深，建议根据上面流程图加上自己打开源码跟着来思考。
其中大概思路:
1. 通过当前执行debug的类，去解析找到SpringBootTest注解，然后找到SpringBoot引导类.
2. 将点击debug的方法，转换成 TestContext.

### 2.3 ③ JUnit单测类属性注入

通过前面的阅读我们已经能拿到了所有的容器启动参数。那么我们可以思考下。我们自己的 单测类其实并没有交给容器来管理,那么我们的单测类中的属性都是什么时候注入的呢?

答案就在 TestExecutionListener. JUnit提供了一系列的Listener。功能类似Spring中的`BeanPostProcessor`.
就是要对 TestContext进行二次加工的切入点。

通过前面我们就能把Spring容器的上下文获取到了，然后又能获取当前单侧类的所有信息。就看看是否依赖容器，如果依赖容器，就向当前的测试类执行，注入。下面图中可以看到这样的代码实现。大家认真看。

``` 
public interface TestExecutionListener {

    default void beforeTestClass(TestContext testContext) throws Exception {
    }

    default void prepareTestInstance(TestContext testContext) throws Exception {
    }

    default void beforeTestMethod(TestContext testContext) throws Exception {
    }

    default void beforeTestExecution(TestContext testContext) throws Exception {
    }

    default void afterTestExecution(TestContext testContext) throws Exception {
    }

    default void afterTestMethod(TestContext testContext) throws Exception {
    }

    default void afterTestClass(TestContext testContext) throws Exception {
    }

}
```
![](https://img.springlearn.cn/blog/learn_1617795655000.png)

通过名字我们发现了貌似一个可以进行依赖注入的类。没错就是在这里,在单侧方法执行前。通过

``` 
public class DependencyInjectionTestExecutionListener extends AbstractTestExecutionListener {
    @Override
    public void beforeTestMethod(TestContext testContext) throws Exception {
        if (Boolean.TRUE.equals(testContext.getAttribute(REINJECT_DEPENDENCIES_ATTRIBUTE))) {
            if (logger.isDebugEnabled()) {
                logger.debug("Reinjecting dependencies for test context [" + testContext + "].");
            }
            injectDependencies(testContext);
        }
    }

    protected void injectDependencies(TestContext testContext) throws Exception {
        Object bean = testContext.getTestInstance();
        Class<?> clazz = testContext.getTestClass();
        AutowireCapableBeanFactory beanFactory = testContext.getApplicationContext().getAutowireCapableBeanFactory();
        beanFactory.autowireBeanProperties(bean, AutowireCapableBeanFactory.AUTOWIRE_NO, false);
        beanFactory.initializeBean(bean, clazz.getName() + AutowireCapableBeanFactory.ORIGINAL_INSTANCE_SUFFIX);
        testContext.removeAttribute(REINJECT_DEPENDENCIES_ATTRIBUTE);
    }
}   
```

### 2.4 ④ 事务回滚原理

在前文单测类注入中我们知道.JUnit提供了一些监听器,允许 当单测方法执行时候去对单测上下文进行调整。所以呢事务回滚也是基于 这里的特性完成的。基于SpringBoot 2.1.x版本分析

![](https://img.springlearn.cn/blog/learn_1617795655000.png)

**源码分析**

Spring中为了适配不同的数据库,提供了事务平台的概念。 PlatformTransactionManager 只要实现了该接口 就允许对事务进行控制。具体事务的控制是通过工具类来处理的。 TransactionContextHolder 可以获取当前线程 执行的事务上下文。JUnit通过该工具拿到事务的上下文,然后对此做相应的修改。具体的 修改逻辑见下文注释。两句话解释清楚。

`TransactionalTestExecutionListener`

``` 
    // 单测方法执行前,移除容器原来的事务管理器,然后开启一个新的事务
    @Override
    public void beforeTestMethod(final TestContext testContext) throws Exception {
        Method testMethod = testContext.getTestMethod();
        Class<?> testClass = testContext.getTestClass();
        Assert.notNull(testMethod, "Test method of supplied TestContext must not be null");

        TransactionContext txContext = TransactionContextHolder.removeCurrentTransactionContext();
        Assert.state(txContext == null, "Cannot start new transaction without ending existing transaction");

        PlatformTransactionManager tm = null;
        TransactionAttribute transactionAttribute = this.attributeSource.getTransactionAttribute(testMethod, testClass);

        if (transactionAttribute != null) {
            transactionAttribute = TestContextTransactionUtils.createDelegatingTransactionAttribute(testContext,
                transactionAttribute);

            if (logger.isDebugEnabled()) {
                logger.debug("Explicit transaction definition [" + transactionAttribute +
                        "] found for test context " + testContext);
            }

            if (transactionAttribute.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NOT_SUPPORTED) {
                return;
            }

            tm = getTransactionManager(testContext, transactionAttribute.getQualifier());
            Assert.state(tm != null,
                    () -> "Failed to retrieve PlatformTransactionManager for @Transactional test: " + testContext);
        }

        if (tm != null) {
            txContext = new TransactionContext(testContext, tm, transactionAttribute, isRollback(testContext));
            runBeforeTransactionMethods(testContext);
            txContext.startTransaction();
            TransactionContextHolder.setCurrentTransactionContext(txContext);
        }
    }
    
    // 单测方法执行结束后,结束事务然后回滚或提交
    @Override
    public void afterTestMethod(TestContext testContext) throws Exception {
        Method testMethod = testContext.getTestMethod();
        Assert.notNull(testMethod, "The test method of the supplied TestContext must not be null");

        TransactionContext txContext = TransactionContextHolder.removeCurrentTransactionContext();
        // If there was (or perhaps still is) a transaction...
        if (txContext != null) {
            TransactionStatus transactionStatus = txContext.getTransactionStatus();
            try {
                // If the transaction is still active...
                if (transactionStatus != null && !transactionStatus.isCompleted()) {
                    txContext.endTransaction();
                }
            }
            finally {
                runAfterTransactionMethods(testContext);
            }
        }
    }
```


## 三、总结

还是那句话,没有人能真正把源码每个部分都给你讲清楚，讲明白。一定要跟着小编一起去思考。
当阅读的源码多了，思考的多了。就自然看透了。所以你有没有领略到这句话呢？

**天下代码一大抄，抄来抄去有提高，看你会抄不会抄。**

在学习一个源码的过程中，要多思考，思考为什么这么做。主要学习其设计模式。然后吸收为我所用。

### 3.1 知识点总结

1. idea调用JUnit。可以给到我们当前类信息。我们可以在代码中来解析这个类的所有信息
   你也可以通过反射得到所有你想要的信息，甚至可以通过约定的设计来达到更智能的操作。比如
   SpringBoot2.2.x之后的版本,你直接使用`@SpringBootTest` 一个注解，不用指定引导类,就能启动一个依赖
   容器的单侧类了。那么他就是通过约定的方式,去指定的目录解析你的引导类,如果解析到就能启动了。

2. JUnit的源码跟Spring有很多的相似点，比如如何使用他的扩展点，就像前文小编说的。JUnit
   的 `TestExecutionListener` 的功能就类Spring中的 `BeanPostProcessor`。

3. 容器注入其实就是对 `TestExecutionListener`的一个实践，是一个比较好学习的例子。当掌握了
   知识点1和2。知识点3就可以做更多的事情。

4. 事务回滚考虑对Spring源码的掌握能力，同时也是对`TestExecutionListener`的一个另外的实践。

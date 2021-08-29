
![](https://img.springlearn.cn/blog/learn_1629295333000.png)

## 一、前言

前面概念篇介绍完,就到了我们的技能篇。本篇文章会介绍些Java领域常见的
单元测试框架,及使用的方法。小编一口气介绍了4款单元测试的框架
干货内容有点多,建议收藏后慢慢观看。

## 二、技术选型

### 2.1 JUnit

JUnit目标是为JVM上的开发人员端测试创建最新的基础。这包括关注Java 8及更高版本，以及启用许多不同的测试样式。

强制使用 Junit3 以上版本, 目前最新的版本是 Junit5, 常用的是 JUnit4,建议使用JUnit4 或者使用JUnit5。

这里有一个小坑。如果SpringBoot2.1.x版本依赖的Junit4。SpringBoot应用要通过 @RunWith + @SpringBootTest。 在SpringBoot后续的版本依赖JUnit5,直接使用@SpringBootTest即可。

```
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

### 2.2 Mockito

Mockito 是一个非常不错的模拟框架。它使您可以使用干净简单的API编写漂亮的测试。Mockito不会给您带来麻烦，因为这些测试的可读性很强，并且会产生清晰的验证错误。

mockito-core只包含mockito类，而mockito-all包含mockito类以及一些依赖项，其中一个是hamcrest。

实际上mockito-all已停产according to the mockito website

``` 
<!-- https://mvnrepository.com/artifact/org.mockito/mockito-core -->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.8.0</version>
    <scope>test</scope>
</dependency>
```


``` 
    // 根据这个原理,我们可以mock所有未实现的功能,比如三方的接口
    @Test
    public void test(){
        List mockList = Mockito.mock(List.class);
        Mockito.doReturn(12).when(mockList).get(0);
        // 12
        System.out.println(mockList.get(0));
        Assert.assertSame(12,mockList.get(0));
    }
```

### 2.3 JMockData

JMockData 是一款国人开发用来生成模拟数据的工具,对象太复杂,模拟数据复制太难? 一行代码搞定。


``` 
   <dependency>
       <groupId>com.github.jsonzou</groupId>
       <artifactId>jmockdata</artifactId>
       <version>4.3.0</version>
       <scope>test</scope>
   </dependency>
```

## 三、框架API

### 3.1 JUnit API

![](https://img.springlearn.cn/blog/learn_1629380317000.png)

#### 3.1.1 常用注解

##### 3.1.1.1 @Before & @After


``` 
    @Before
    public void before() {
        System.out.println("before");
    }

    @After
    public void after() {
        System.out.println("after");
    }
```

##### 3.1.1.2 @BeforeClass & @AfterClass

区别与上一个,不管单测类中有几个单测方法,都只会执行一次

要用静态修饰

``` 
    @BeforeClass
    public static void beforeClass() {
        System.out.println("beforeClass");
    }

    @AfterClass
    public static void afterClass() {
        System.out.println("afterClass");
    }
```

``` 
public class JUnitTest {

    @BeforeClass
    public static void beforeClass() {
        System.out.println("beforeClass");
    }

    @Before
    public void before() {
        System.out.println("before");
    }

    @Test
    public void testOne() {
        System.out.println("testOne");
    }

    @Test
    public void testTwo() {
        System.out.println("testTwo");
    }

    @AfterClass
    public static void afterClass() {
        System.out.println("afterClass");
    }

    @After
    public void after() {
        System.out.println("after");
    }
}

```

![](https://img.springlearn.cn/blog/learn_1629380444000.png)

##### 3.1.1.3 @Timed

被修饰的方法会加上一个时间限制,如果超过了指定的时间范围,就算单侧代码执行成功 了也被认为是失败。(注意该方法依赖于SpringBoot容器)

``` 
    @Test
    @Timed(millis = 2000)
    public void testTimeout() {
        System.out.println("testOne");
    }
```

##### 3.1.1.4 @Repeat

指定当前单测方法被执行的次数,如果被该注解修饰 将会被重复执行。(注意该方法依赖于SpringBoot容器)

``` 
    @Test
    @Repeat(3)
    public void testOne() {
        System.out.println("testOne");
    }
```

#### 3.1.2 断言API

断言的好处在于程序帮忙判断单测结果。不需要人工在接入验证数据。JUnit的口号就是

`keep the bar green to keep the code clean。`

一个不用观察输出就知道代码有没有问题的高效单元测试工具。

``` 
import org.hamcrest.Matchers;
import org.hamcrest.core.AllOf;
import org.hamcrest.core.AnyOf;
```

##### 3.1.2.1 Matchers 匹配

``` 
        // 是否相等
        Assert.assertThat(2, Matchers.is(2));
        // 2 小于等于2
        Assert.assertThat(2,Matchers.lessThanOrEqualTo(2));
        Map<String,String> map = new HashMap<>();
        map.put("name","jay");
        // map 中是否包含key为name的元素
        Assert.assertThat(map,Matchers.hasKey("name"));
        // map 中是否包含value为jay的元素
        Assert.assertThat(map,Matchers.hasValue("jay"));
        // map 中是否包含name等于jay的元素
        Assert.assertThat(map,Matchers.hasEntry("name","jay"));
```


##### 3.1.2.2 AllOf 匹配

全部满足

``` 
   // 2 小于4同时也小于3
   Assert.assertThat(2, AllOf.allOf(Matchers.lessThan(4), Matchers.lessThan(3)));
```

##### 3.1.2.3 AllOf AnyOf

任意满足

``` 
   // 2 大于1小于3
   Assert.assertThat(2, AnyOf.anyOf(Matchers.greaterThan(1), Matchers.lessThan(3)));
```

#### 3.1.3 结果验证

##### 3.1.3.1 空值验证

``` 
    @Test
    public void test() {
        Object o = new Object();
        // 非空验证
        Assert.assertNotNull(o);
        // 空值验证
        Assert.assertNull(null);
    }    
```

##### 3.1.3.2 逻辑验证

``` 
    import static org.hamcrest.MatcherAssert.*;
    import static org.hamcrest.CoreMatchers.*;
    public calss Test{
        @Test
        public void test() {
            //测试变量是否大于指定值
            ArrivalNoticeOrderDO ao = new ArrivalNoticeOrderDO();
            ao.setId(12L);
            //测试所有条件必须成立
            assertThat(ao.getId(), allOf(is(12L)));
            //测试只要有一个条件成立
            assertThat(ao.getId(), anyOf(is(50), is(12L)));
            //测试变量值等于指定值
            assertThat(ao.getId(), is(12L));
        }
    }
```

##### 3.1.3.3 异常验证

``` 
    /**
     * 预期异常
     */
    @Test(expected = NullPointerException.class)
    public void testError(){
        Object o = null;
        System.out.println(o.toString());
    }
```


#### 3.1.4 Idea快速创建

建议使用 Idea 自动创建, 不要手动创建。

![](https://ddd.springlearn.cn/assets/images/junit-fce53aac66d9c35a4871c3494fff9548.gif)

## 3.2 MockData API

`JMockData` 是一款国人开发用来生成模拟数据的工具

### 3.2.1 基础类型模拟

| 描述         | 类型                                                         |
| ------------ | ------------------------------------------------------------ |
| 基础类型     | `byte` `boolean` `char` `short` `int` `long` `float` `double` |
| 包装类型包装 | `Byte` `Boolean` `Character` `Short` `Integer` `Long` `Float` `Double` |
| 常用类型     | `BigDecimal` `BigInteger` `Date` `LocalDateTime` `LocalDate` `LocalTime` `java.sql.Timestamp` `String` `Enum` |
| 多维数组     | 以上所有类型的多维数组 如：`int[]` `int[][]` `int[][][]` .... etc.() |

``` 
//基本类型模拟
int intNum = JMockData.mock(int.class);
int[] intArray = JMockData.mock(int[].class);
Integer integer = JMockData.mock(Integer.class);
Integer[] integerArray = JMockData.mock(Integer[].class);
//常用类型模拟
BigDecimal bigDecimal = JMockData.mock(BigDecimal.class);
BigInteger bigInteger = JMockData.mock(BigInteger.class);
Date date = JMockData.mock(Date.class);
String str = JMockData.mock(String.class);
```
### 3.2.2 JAVA对象模拟

模拟bean，被模拟的数据最好是plain bean，通过反射给属性赋值。

``` 

public class User {

    private String name;

    private int age;

    private long cardId;
    
}  
```

``` 
    @Test
    public void test() {
        User mock = JMockData.mock(User.class);
        // User{name='jrq2b', age=9338, cardId=2850}
        System.out.println(mock);
    }  
```


### 3.2.3 集合容器对象模拟

``` 
@Test
//******注意TypeReference要加{}才能模拟******
public void testTypeRefrence() {
  //模拟基础类型，不建议使用这种方式，参考基础类型章节直接模拟。
  Integer integerNum = JMockData.mock(new TypeReference<Integer>(){});
  Integer[] integerArray = JMockData.mock(new TypeReference<Integer[]>(){});
  //模拟集合
  List<Integer> integerList = JMockData.mock(new TypeReference<List<Integer>>(){});
  //模拟数组集合
  List<Integer[]> integerArrayList = JMockData.mock(new TypeReference<List<Integer[]>>(){});
  //模拟集合数组
  List<Integer>[] integerListArray = JMockData.mock(new TypeReference<List<Integer>[]>(){});
  //模拟集合实体
  List<BasicBean> basicBeanList = JMockData.mock(new TypeReference<List<BasicBean>>(){});
  //各种组合忽略。。。。map同理。下面模拟一个不知道什么类型的map
  Map<List<Map<Integer, String[][]>>, Map<Set<String>, Double[]>> some = JMockData.mock(new TypeReference<Map<List<Map<Integer, String[][]>>, Map<Set<String>, Double[]>>>(){});
}
```


### 3.2.4 模拟范围限定

前面说了可以模拟各种数据,不同类型的数据都允许指定一个范围。 如下


``` 
System.out.println(
JMockData.mock(Date.class,MockConfig.newInstance()
.dateRange("2018-11-20", "2018-11-30")));
```

全局配置

``` 
        MockConfig mockConfig = new MockConfig()
                // 全局配置
                .globalConfig()
                .setEnabledStatic(false)
                .setEnabledPrivate(false)
                .setEnabledPublic(false)
                .setEnabledProtected(false)
                .sizeRange(1, 1)
                .charSeed((char) 97, (char) 98)
                .byteRange((byte) 0, Byte.MAX_VALUE)
                .shortRange((short) 0, Short.MAX_VALUE)
                // 某些字段（名等于integerNum的字段、包含float的字段、double开头的字段）配置
                .subConfig("integerNum", "*float*", "double*")
                .intRange(10, 11)
                .floatRange(1.22f, 1.50f)
                .doubleRange(1.50, 1.99)
                .longRange(12, 13)
                .dateRange("2018-11-20", "2018-11-30")
                .stringSeed("SAVED", "REJECT", "APPROVED")
                .sizeRange(1, 1)
                // 全局配置
                .globalConfig()
                // 排除所有包含list/set/map字符的字段。表达式不区分大小写。
                .excludes("*List*", "*Set*", "*Map*");
```

## 3.3 Mockito API

![](https://img.springlearn.cn/blog/learn_1629381317000.png)

### 3.3.1 Mockito加载方式

#### 3.3.1.1 不依赖Spring容器

如果你的单测不依赖容器,那么使用这种方式是比较方便和简介的。但是如果 依赖容器,我们是到JUnit的原理是只要发现有一个Runner就会返回,如果这里指定了 MockitoJUnitRunner那么SpringRunner就不会被使用。

``` 
   @RunWith(MockitoJUnitRunner.class)
   public class ExampleTest {
   
       @Mock
       private List list;
   
       @Test
       public void shouldDoSomething() {
           list.add(100);
       }
   }
```

#### 3.3.1.2 依赖容器

方式2是依赖于Spring容器的,所以要求我们在单测方法执行前来通知Mockito来处理 他的逻辑,处理他说使用的注解。JUnit4的@Before注解就是做好的加载时机,因为我们 可以这样写。

``` 

   /**
     * 将单测类中依赖Mockito的属性,进行处理。
     * 帮我们实现 Mockito.mock()
     */
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }
```


### 3.3.2 Mockito必知概念

#### 3.3.2.1 完全模拟 Mock

什么是完全模拟,使用的注解就是@Mock。被Mock的对象,所有的方法都不会被 真正的执行。

#### 3.3.2.2 部分模拟 Spy

部分模拟,使用的注解就是@Spy(间谍一样)。被声明的方法走Mock,没有声明的方法 还是由实例进行执行和反馈。

### 3.3.3 代码实例

这里的例子我们为了启动快速,不依赖Spring容器。直接new出来对象。 另外多说一句,其实就算依赖Spring容器,当@Before方法执行前所有的示例其实也都是已经注入好的了。

```
public class MockitoEmp {
        public String getName() {
            return "真实的MockitoTest";
        }

        public Integer getAge() {
            return 23;
        }
    }
```

#### 3.3.3.1 @Mock

``` 
MockitoEmp mock = Mockito.mock(MockitoEmp.class);
```

![](https://img.springlearn.cn/blog/learn_1629381540000.png)

``` 

public class MockitoTest {

    // 整个对象都是Mock的
    @Mock
    private MockitoEmp mock = new MockitoEmp();

    /**
     * 将单测类中依赖Mockito的属性,进行处理。
     * 帮我们实现 Mockito.mock()
     */
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void testMock() {
        Mockito.doReturn("Mock数据").when(mock).getName();
        //等价于Mockito.when(mock.getName()).thenReturn("Mock数据");
        // Mock数据
        Assert.assertSame("Mock数据", mock.getName());
        // getAge() 方法没有用Mockito声明动作, 应该是多少呢?
        Assert.assertSame(0, mock.getAge());
        // 0
        System.out.println(mock.getAge());
    }
}

```

#### 3.3.3.2 @Spy

``` 
MockitoEmp spy = Mockito.spy(MockitoEmp.class);
```

![](https://img.springlearn.cn/blog/learn_1629381617000.png)


``` 

public class MockitoTest {

    // 整个对象都是Mock的
    @Mock
    private MockitoEmp mock = new MockitoEmp();

    /**
     * 将单测类中依赖Mockito的属性,进行处理。
     * 帮我们实现 Mockito.mock()
     */
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void testSpy() {
        Mockito.doReturn("Mock数据").when(spy).getName();
        // Mock数据
        Assert.assertSame("Mock数据", spy.getName());
        // getAge() 方法没有用Mockito声明动作, 应该是多少呢?
        Assert.assertSame(23, spy.getAge());
        // 23
        System.out.println(spy.getAge());
    }
}

```

## 3.4 SpringBoot Testing


![](https://img.springlearn.cn/blog/learn_1618140868000.png)

前面我们对Mockito的用法有了一个了解,这里告诉大家一个好消息,SpringBoot已经帮我们继承了 这些框架,而且提供了更加简单好用的API。

### 3.4.1 Mockito加载方式


前面我们说了两种加载方式 MockitoJUnitRunner 和 MockitoAnnotations.initMocks(this); 这些在SpringBoot中都不需要了。

所以这一段就是废话, 不用在看了。但是相信你已经看完了。

### 3.4.2 Mockito必知概念

这些概念,参考Mockito章节,概念统统保留。

##### 3.4.2.1 完全模拟 MockBean

只需要将@Mock 换成 @MockBean即可

##### 3.4.2.2 部分模拟 SpyBean

只需要将@Spy 换成 @MockBean即可。主要这里有一个小坑。 如果是Feign接口,使用@SpyBean会报错。提示final class不能被代理。

原因是SpringBoot依赖的Mockito版本太古老了,是2.23.4。从Mockito2.7.6 开始已经解决了这个问题, 我们可以通过引入下面依赖解决。

``` 
 <dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-inline</artifactId>
    <version>3.3.3</version>
</dependency>

```

解决方案就是帮我们新增了一个配置,启动Mockit的插件来生成代理。 大概原理就是及不实用JDK代理,也不是Cglib代理。 DefaultMockitoPlugins & InlineByteBuddyMockMaker


![](https://img.springlearn.cn/blog/learn_1617877205000.png)


### 3.4.3 代码实例

#### 3.4.3.1 @MockBean 完全模拟

没有被声明的方法返回值,对象类型返回null,基本类型是返回默认类型。

``` 



public class TradeShopIntegrationImplTest extends BaseApplicationTest {

    @Autowired
    private TradeShopIntegration shopBrandIntegration;

    @MockBean
    private BrandServiceApi brandService;
    
    @MockBean
    private GoodsStockApi goodsStockApi;
    
    @Test
    public void testGetAllBrands() {
        Mockito.doReturn(JsonResult.failure("fail")).when(goodsStockApi).getSkuList(Mockito.any());
        // 底层调用的是goodsStockApi.getSkuList()
        List<GoodsBaseMsgDTO> goodsBaseMsgDTOS = shopBrandIntegration.queryAllSku();
        // 因为前面声明了返回fail。所以这里没有数据返回。
        JsonConsoleUtils.println(goodsBaseMsgDTOS);
        // 这里因为使用的是Mock完全模拟,所以尽管前面没有声明返回值,就默认返回null
        List<OutBrandDTO> allBrands = shopBrandIntegration.getAllBrands();
        JsonConsoleUtils.println(allBrands);
    }
    
}    
```

#### 3.4.3.2 @SpyBean 部分模拟

没有被声明的方法返回值,走原来逻辑。

``` 


public class TradeShopIntegrationImplTest extends BaseApplicationTest {

    @Autowired
    private TradeShopIntegration shopBrandIntegration;

    @MockBean
    private BrandServiceApi brandService;
    
    @MockBean
    private GoodsStockApi goodsStockApi;
    
    @Test
    public void testGetAllBrands() {
        Mockito.doReturn(JsonResult.failure("fail")).when(goodsStockApi).getSkuList(Mockito.any());
        // 底层调用的是goodsStockApi.getSkuList()
        List<GoodsBaseMsgDTO> goodsBaseMsgDTOS = shopBrandIntegration.queryAllSku();
        // 因为前面声明了返回fail。所以这里没有数据返回。
        JsonConsoleUtils.println(goodsBaseMsgDTOS);
        // 这里跟上面的区别就是,如果没有声明返回值,就走原来的方法。
        List<OutBrandDTO> allBrands = shopBrandIntegration.getAllBrands();
        JsonConsoleUtils.println(allBrands);
    }
    
}    

```


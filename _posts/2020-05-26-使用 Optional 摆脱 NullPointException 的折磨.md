**背景**

在Java中，如果你尝试对null做函数调用，就会引发NullPointerException（NPE），NPE是Java程序开发中的最典型的异常，对于Java开发者来说，无论你是初出茅庐的新人和还工作多年的老司机，NPE经常让他们翻车。为了避免NPE，他们会加很多if判断语句，使得代码的可读性变得很差。

从软件设计的角度来看，null本身是没有意义的语义，这是一种对缺失变量值的错误的建模。

从Java类型系统的角度看，null可以被赋值给任何类型的变量，并且不断被传递，知道最后谁也不知道它是从哪里引入的。

**Optional的引入**

Java设计者从Haskell和Scala中获取灵感，在Java 8中引入了一个新的类 `java.util.Optional`。如果一个接口返回Optional，可以表示一个人可能有车也可能没有车，这个比简单的返回Car要更明确，阅读代码的人不需要提前准备业务知识。

Optional的目的就在于此：通过类型系统让你的领域模型中隐藏的知识显式地体现在你的代码中。

**Optional的使用**

![img](https://s1.ax1x.com/2020/05/27/tAWqSS.png)

上面这张表里列举了Optional的基础API，我这里列举了一些使用的tips：

- 你可以用ofNullable将一个可能为null的对象封装为Optional对象，然后获取值的时候使用orElse方法提供默认值；可以使用empty方法创建一个空的Optional对象；of方法一般不用，不过如果你知道某个值不可能为null，则可以用Optional封装该值，这样它一旦为null就会抛出异常。

  ![img](https://s1.ax1x.com/2020/05/27/tAfFlF.png)

- 从某个对象中获取值是最常见的一种场景，这时候为了避免这个对象为null导致NPE，一般是使用if-then-else结构检查，如果使用Optional的话，则可以使用map方法来获取它封装的对象中某个字段的值。

  ![img](https://s1.ax1x.com/2020/05/27/tAfllD.png)

- 如果需要连续、层层递进的从某个对象链的末端获取字段的值，则不能全部使用map方法，需要先使用flatMap，最后再使用map方法；Optional中的map、flatMap和filter方法，在概念是与Stream中对应的方法都很类似，区别就在于Optional中的元素至多有一个，算是Stream的一种特殊情况——一种特殊的集合。

  ![img](https://s1.ax1x.com/2020/05/27/tAfYTI.png)

- 不要使用ifPresent和get方法，它们本质上和不适用Optional对象之前的模式相同，都是臃肿的if-then-else判断语句；

- 由于Optional无法序列化，所以在领域模型中，无法将某个字段定义为Optional的，原因是：Optional的设计初衷仅仅是要支持能返回Optional对象的语法，如果我们希望在域模型中引入Optional，则可以用下面这种替代的方法：

  ![img](https://s1.ax1x.com/2020/05/27/tAfh1U.png)

- 不要使用基础类型的Optional对象，原因是：基础类型的Optional对象不支持map、flatMap和filter方法，而这些方法是Optional中非常强大的方法。



**实战案例**

**
**

**使用工具类方法改良可能抛出异常的API** 

Java方法处理异常结果的方式有两种：返回null（或错误码）；抛出异常，例如：Integer.parseInt(String)这个方法——如果无法解析到对应的整型，该方法就抛出一个NumberFormationException，这种情况下我们一般会使用try/catch语句处理异常情况。

一般我们建议将try/catch块单独提取到一个方法中，在这里使用Optional设计这个方法，代码如下。在开发中，可以尝试构建一个OptionalUtility工具类，将这些复杂的try/catch逻辑封装起来。

![img](https://s1.ax1x.com/2020/05/27/tAfLh6.png)



**综合案例**

现在有个方法，是尝试从一个属性映射中获取某个关键词对应的值，例子代码如下：

![img](https://s1.ax1x.com/2020/05/27/tAfvcD.png)

使用Optional的写法后，代码如下所示：

![img](https://s1.ax1x.com/2020/05/27/tAhPAI.png)

如果需要访问的属性值不存在，Properites.getProperty(String)方法的返回值就是一个null，使用noNullable工厂方法就可以将该值转换为Optional对象；接下来，可以使用flatMap将一个Optional<String>转换为Optional<Integer>对象；最后使用filter过滤掉负数，然后就可以使用orElse获取属性值，如果拿不到则返回默认值0。

**总结**

使用Optional的思路和Stream相同，都是链式思路，跟数据库查询似的，表达力很强，而且省去了哪些复杂的try/catch和if-then-else方法。在后面的开发中，可以使用Optional设计API，这样可以设计出更安全的接口和方法。

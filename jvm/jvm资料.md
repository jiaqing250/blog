# 1、jvm 堆内存分配
![1667964235041.jpg](https://cdn.nlark.com/yuque/0/2022/jpeg/2923644/1667964238464-f1127330-2967-411b-8987-cd04f201cea5.jpeg#averageHue=%23f9fcf9&clientId=u35b0ce74-fcd9-4&errorMessage=unknown%20error&from=paste&height=245&id=u54714754&originHeight=367&originWidth=1174&originalType=binary&ratio=1&rotation=0&showTitle=false&size=143639&status=error&style=none&taskId=ua7c983c5-9b6a-44c8-ac51-f14fe0836d2&title=&width=782.6666666666666)
# 2、字符串拼接操作


●常量与常量的拼接结果在常量池，原理是编译期优化<br />●常量池中不会存在相同内容的变量<br />●只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder<br />●如果拼接的结果调用intern()方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址

 
```java
public void test6(){
    String s0 = "beijing";
    String s1 = "bei";
    String s2 = "jing";
    String s3 = s1 + s2;
    System.out.println(s0 == s3); // false s3指向对象实例，s0指向字符串常量池中的"beijing"
    String s7 = "shanxi";
    final String s4 = "shan";
    final String s5 = "xi";
    String s6 = s4 + s5;
    System.out.println(s6 == s7); // true s4和s5是final修饰的，编译期就能确定s6的值了
}

```
●不使用final修饰，即为变量。如s3行的s1和s2，会通过new StringBuilder进行拼接<br />●使用final修饰，即为常量。会在编译器进行代码优化。在实际开发中，能够使用final的，尽量使用
# 3、jvm全景图
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2923644/1667975675579-92fc4760-e8f0-4852-947b-87723a58988a.png#averageHue=%23eee8e7&clientId=u35b0ce74-fcd9-4&errorMessage=unknown%20error&from=paste&id=u685c62a8&originHeight=1632&originWidth=2940&originalType=url&ratio=1&rotation=0&showTitle=false&size=1051492&status=error&style=none&taskId=u0e020396-f52d-424a-abb2-4875000ec18&title=)<br />![](https://cdn.nlark.com/yuque/0/2022/png/2923644/1668052614028-392d1a76-e356-4a7c-a89e-6ea5d519ab26.png#averageHue=%23f1eaac&clientId=u4e493a83-6050-4&from=paste&id=u5e840792&originHeight=6311&originWidth=5045&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ub0d20b6a-6b85-47ae-91e5-ae6a6f503a9&title=)<br />[点击查看【processon】](https://www.processon.com/embed/5ea7a1b9e401fd21c196eb17)<br />[点击查看【processon】](https://www.processon.com/embed/625cafa8e401fd32a55cafff)<br />原链接：<br />[JVM内存模型完整版 流程图模板_ProcessOn思维导图、流程图](https://www.processon.com/view/5ea7a1b9e401fd21c196eb17?fromnew=1)

# 
# 5、所谓的jvm性能优化实际上就是减少全部 gc处理的时间(stw Stop-The-World)

# 6、jvm GcRoot方式
[一文带你弄懂 JVM 三色标记算法！ - 陈树义 - 博客园](https://www.cnblogs.com/chanshuyi/p/head-first-of-triple-color-marking-algorithm.html)<br />![](https://cdn.nlark.com/yuque/0/2022/png/2923644/1667981085600-9d00c62a-2191-463c-b916-28c44b43985b.png#averageHue=%23fbf3ee&clientId=u35b0ce74-fcd9-4&errorMessage=unknown%20error&from=paste&id=u9df42365&originHeight=1460&originWidth=2428&originalType=url&ratio=1&rotation=0&showTitle=false&status=error&style=none&taskId=ud47f1c63-ab92-4e85-b967-370d1843c54&title=)

# 7、jvm slot(槽)
先知道栈桢(stackFrame)的结构<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/2923644/1667890598422-0e256953-ec37-4c02-982f-18c71ed80957.png#averageHue=%23f6f2ee&clientId=u97126ff1-c00b-4&from=paste&height=609&id=u63bf39dc&originHeight=914&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=319220&status=done&style=none&taskId=u0a7b2a08-7a21-4c41-972c-63ff03092cd&title=&width=1280)<br />槽的介绍：<br />著作权归https://pdai.tech所有。 链接：[https://pdai.tech/md/java/jvm/java-jvm-struct.html](https://pdai.tech/md/java/jvm/java-jvm-struct.html)

- 局部变量表最基本的存储单元是 Slot（变量槽）
- 在局部变量表中，32 位以内的类型只占用一个 Slot(包括returnAddress类型)，64 位的类型（long和double）占用两个连续的 Slot
- byte、short、char 在存储前被转换为int，boolean也被转换为int，0 表示 false，非 0 表示 true
- long 和 double 则占据两个 Slot
- JVM 会为局部变量表中的每一个 Slot 都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值，索引值的范围从 0 开始到局部变量表最大的 Slot 数量
- 当一个实例方法被调用的时候，它的方法参数和方法体内部定义的局部变量将会**按照顺序被复制**到局部变量表中的每一个 Slot 上
- **如果需要访问局部变量表中一个 64bit 的局部变量值时，只需要使用前一个索引即可**。（比如：访问 long 或 double 类型变量，不允许采用任何方式单独访问其中的某一个 Slot）
- 如果当前帧是由构造方法或实例方法创建的，那么该对象引用** this 将会存放在 index 为 0 的 Slot **处，其余的参数按照参数表顺序继续排列（这里就引出一个问题：静态方法中为什么不可以引用 this，就是因为this 变量不存在于当前方法的局部变量表中）
- **栈帧中的局部变量表中的槽位是可以重用的**，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变量就很有可能会复用过期局部变量的槽位，从而**达到节省资源的目的**。（下图中，this、a、b、c 理论上应该有 4 个变量，c 复用了 b 的槽）

工具：

1. idea
2. jclasslib (插件)

代码<br />![原代码](https://cdn.nlark.com/yuque/0/2022/png/2923644/1667890966604-4bfdcfd8-ddc4-4c7b-963e-a49bb82a88c3.png#averageHue=%23302e2c&clientId=u97126ff1-c00b-4&from=paste&height=115&id=u9678482a&originHeight=172&originWidth=336&originalType=binary&ratio=1&rotation=0&showTitle=true&size=8802&status=done&style=none&taskId=ud2d67168-648d-4a9d-b525-03ed55d8f7a&title=%E5%8E%9F%E4%BB%A3%E7%A0%81&width=224 "原代码")![jclasslib插件查询出来的槽的数量](https://cdn.nlark.com/yuque/0/2022/png/2923644/1667890990660-d00b2292-84bf-4bfc-bcce-e5eef8128331.png#averageHue=%233b3f43&clientId=u97126ff1-c00b-4&from=paste&height=164&id=u4a7579b7&originHeight=328&originWidth=865&originalType=binary&ratio=1&rotation=0&showTitle=true&size=22295&status=done&style=none&taskId=u73e954ac-adec-4d66-ade6-d454f05a41f&title=jclasslib%E6%8F%92%E4%BB%B6%E6%9F%A5%E8%AF%A2%E5%87%BA%E6%9D%A5%E7%9A%84%E6%A7%BD%E7%9A%84%E6%95%B0%E9%87%8F&width=433 "jclasslib插件查询出来的槽的数量")![对应的字节码](https://cdn.nlark.com/yuque/0/2022/png/2923644/1667891228832-f8f10eab-c751-4e95-8d0e-651e57457257.png#averageHue=%233e4346&clientId=u97126ff1-c00b-4&from=paste&height=236&id=ua505cc7e&originHeight=314&originWidth=874&originalType=binary&ratio=1&rotation=0&showTitle=true&size=48115&status=done&style=none&taskId=ua6308601-cf69-43a3-934f-6c70386d0a3&title=%E5%AF%B9%E5%BA%94%E7%9A%84%E5%AD%97%E8%8A%82%E7%A0%81&width=656 "对应的字节码")![jclasslib插件上也有对应的信息显示](https://cdn.nlark.com/yuque/0/2022/png/2923644/1667891748228-15235aa0-d6aa-4ed5-bf9c-ff6206004aa8.png#averageHue=%233d4245&clientId=u97126ff1-c00b-4&from=paste&height=184&id=u5ee00a94&originHeight=276&originWidth=886&originalType=binary&ratio=1&rotation=0&showTitle=true&size=22976&status=done&style=none&taskId=ub4305311-7470-4687-b91b-1c3f691a2c5&title=jclasslib%E6%8F%92%E4%BB%B6%E4%B8%8A%E4%B9%9F%E6%9C%89%E5%AF%B9%E5%BA%94%E7%9A%84%E4%BF%A1%E6%81%AF%E6%98%BE%E7%A4%BA&width=590.6666666666666 "jclasslib插件上也有对应的信息显示")
# 8、设置堆内存大小和 OOM
Java 堆用于存储 Java 对象实例，那么堆的大小在 JVM 启动的时候就确定了，我们可以通过 -Xmx 和 -Xms 来设定

- -Xms 用来表示堆的起始内存，等价于 -XX:InitialHeapSize
- -Xmx 用来表示堆的最大内存，等价于 -XX:MaxHeapSize

如果堆的内存大小超过 -Xmx 设定的最大内存， 就会抛出 OutOfMemoryError 异常。<br />我们通常会将 -Xmx 和 -Xms 两个参数配置为相同的值，其目的是为了能够在垃圾回收机制清理完堆区后不再需要重新分隔计算堆的大小，从而提高性能

- 默认情况下，初始堆内存大小为：电脑内存大小/64
- 默认情况下，最大堆内存大小为：电脑内存大小/4

可以通过代码获取到我们的设置值，当然也可以模拟 OOM：
```java
// 著作权归https://pdai.tech所有。
// 链接：https://pdai.tech/md/java/jvm/java-jvm-struct.html

public static void main(String[] args) {

    //返回 JVM 堆大小
    long initalMemory = Runtime.getRuntime().totalMemory() / 1024 /1024;
    //返回 JVM 堆的最大内存
    long maxMemory = Runtime.getRuntime().maxMemory() / 1024 /1024;

    System.out.println("-Xms : "+initalMemory + "M");
    System.out.println("-Xmx : "+maxMemory + "M");

    System.out.println("系统内存大小：" + initalMemory * 64 / 1024 + "G");
    System.out.println("系统内存大小：" + maxMemory * 4 / 1024 + "G");
}
```
参考：<br />[JVM 基础 - JVM 内存结构](https://pdai.tech/md/java/jvm/java-jvm-struct.html)
# 9、jvm
![](https://cdn.nlark.com/yuque/0/2020/png/544173/1579623348007-aba3f934-9874-4f06-8577-99e3cbf612a1.png?x-oss-process=image%2Fresize%2Cw_681%2Climit_0#averageHue=%23fdfdfd&from=url&id=rP8YO&originHeight=382&originWidth=681&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
# 10、对象内存分配
![](https://cdn.nlark.com/yuque/0/2022/png/21911222/1664784643117-4710fee9-6f6b-430b-958b-cbc879dfc233.png#averageHue=%23faf8f7&from=url&id=BQy10&originHeight=866&originWidth=2130&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
# 11、java中的string创建
![](https://cdn.nlark.com/yuque/0/2022/jpeg/2923644/1664176196096-e17d2639-b41c-4d05-9021-bc329321ac15.jpeg#averageHue=%23facd83&clientId=u86ab9ff0-3248-4&from=paste&id=u7110a0c5&originHeight=562&originWidth=708&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uf9777766-2ca7-4777-a7bf-0a6930df6c3&title=)
# 12、对于引用计算器和gcRoot比较清晰的描述
[JVM 垃圾回收](https://www.yuque.com/yinjianwei/vyrvkf/gq1850?view=doc_embed)<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/223463/1567396117464-fc76d59e-0991-4c05-989d-f277960c6ad9.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_17%2Ctext_5q635bu65Y2rIC0gSmF2YSDlvIDlj5HnrJTorrA%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_592%2Climit_0#averageHue=%238aa975&from=url&id=RKV4f&originHeight=438&originWidth=592&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/223463/1567397302878-a9a105ea-1751-4210-9ae1-2b2a012375d2.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_17%2Ctext_5q635bu65Y2rIC0gSmF2YSDlvIDlj5HnrJTorrA%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_592%2Climit_0#averageHue=%2381a86f&from=url&id=sGkzE&originHeight=438&originWidth=592&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
# 13、JVM 类加载机制
先说一下场景：<br />1、可以用来进行加密jar包，然后自定义classloader进行解密运行程序。<br />2、在线程池中的时候可以用来隔离资源。<br />_类加载器为什么要使用双亲委派？_

1. 安全性
2. 防止类重复加载

# 
# 14、JMM 如何解决可见性、原子性、有序性的问题
Java 中已经提供了一系列处理并发编程的关键字，比如 volatile、synchronized、final 这些就是 Java 内存模型封装了底层的实现后提供给开发人员使用的关键字。

**所以 Java 内存模型除了定义了一套规范，还提供了一系列并发编程的关键字。**

volatile、synchronized、final 这些关键字分别可以解决哪些线程的安全性问题呢？

- volatile 可以解决可见性、有序性的问题
- synchronized 可以解决可见性、原子性、有序性的问题（synchronized 是全能型关键字）
- final 可以解决可见性问题    

下面我们通过使用 volatile 和 synchronized 关键字来解决可见性、原子性、有序性的问题，对比[线程的安全性问题](https://www.yuque.com/yinjianwei/vyrvkf/ygfq5z)<br />[Java 内存模型（JMM）](https://www.yuque.com/yinjianwei/vyrvkf/hi3xiq?view=doc_embed)<br />[synchronized 关键字](https://www.yuque.com/yinjianwei/vyrvkf/og00ra?view=doc_embed)<br />[死磕Synchronized底层实现--概论 · Issue #12 · farmerjohngit/myblog](https://github.com/farmerjohngit/myblog/issues/12)<br />[高性能队列——Disruptor](https://tech.meituan.com/2016/11/18/disruptor.html)

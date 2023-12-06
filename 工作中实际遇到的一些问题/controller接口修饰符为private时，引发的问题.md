
在正常情况下，private修饰符的接口 注入的类都是为null。理论上通过springutil.getbean获取到的类都是注入好的。所以理想情况下应该是有值的 -- 通过实验证明，这理论是没问题的<br />

![这个private修饰的接口没有出现上面的情况](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690249142286-c9b5a61a-6b79-446c-8477-7f23f638ff26.png#averageHue=%23627d46&clientId=u62b3ac76-31a1-4&from=paste&height=345&id=ube55e85b&originHeight=517&originWidth=1344&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=94851&status=done&style=none&taskId=u296a6baa-805a-49c0-8135-42b643ba097&title=%E8%BF%99%E4%B8%AAprivate%E4%BF%AE%E9%A5%B0%E7%9A%84%E6%8E%A5%E5%8F%A3%E6%B2%A1%E6%9C%89%E5%87%BA%E7%8E%B0%E4%B8%8A%E9%9D%A2%E7%9A%84%E6%83%85%E5%86%B5&width=896 "这个private修饰的接口没有出现上面的情况")

研究办法：通过两个不同接口的栈堆进行分析 -- 这里涉及到公司信息，删除了
    

通过差异代码发现有aop介入导致的问题<br />![必须有aop能够捕捉到的方法，才会被aop代理](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690251970639-f811ef45-6c18-4cb5-9f12-3e62a3e91183.png#averageHue=%232f2d2b&clientId=u62b3ac76-31a1-4&from=paste&height=561&id=u1667b6ce&originHeight=841&originWidth=1170&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=127750&status=done&style=none&taskId=u97aa893f-3154-4803-a0ba-6912de5f623&title=%E5%BF%85%E9%A1%BB%E6%9C%89aop%E8%83%BD%E5%A4%9F%E6%8D%95%E6%8D%89%E5%88%B0%E7%9A%84%E6%96%B9%E6%B3%95%EF%BC%8C%E6%89%8D%E4%BC%9A%E8%A2%ABaop%E4%BB%A3%E7%90%86&width=780 "必须有aop能够捕捉到的方法，才会被aop代理")<br />结论：

1. 在controller中如果没有使用aop切面的时候，使用public和private都是一样的，@resource都是生效的。 -- controller类使用实例化的方式进行的
2. 在controller中如果有使用aop切面的时候，使用public和private是不一样的，@resource不生效。 -- controller类使用代理类的方式进行的

怎么看是否通过代理类进行的，可以通过观察发现<br />![this @符合后面是一个正常的数字，表示是通过实例化进行的](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690249142286-c9b5a61a-6b79-446c-8477-7f23f638ff26.png#averageHue=%23627d46&from=url&id=ieQEa&originHeight=517&originWidth=1344&originalType=binary&ratio=1.5&rotation=0&showTitle=true&status=done&style=none&title=this%20%40%E7%AC%A6%E5%90%88%E5%90%8E%E9%9D%A2%E6%98%AF%E4%B8%80%E4%B8%AA%E6%AD%A3%E5%B8%B8%E7%9A%84%E6%95%B0%E5%AD%97%EF%BC%8C%E8%A1%A8%E7%A4%BA%E6%98%AF%E9%80%9A%E8%BF%87%E5%AE%9E%E4%BE%8B%E5%8C%96%E8%BF%9B%E8%A1%8C%E7%9A%84 "this @符合后面是一个正常的数字，表示是通过实例化进行的")<br />![this 后面有$$cglib表示使用的是cglib的方式进行代理的](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690248703519-d076e493-c9fb-4271-b5ba-ee180a984771.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0#averageHue=%236b8279&from=url&id=PrYw0&originHeight=499&originWidth=1125&originalType=binary&ratio=1.5&rotation=0&showTitle=true&status=done&style=none&title=this%20%E5%90%8E%E9%9D%A2%E6%9C%89%24%24cglib%E8%A1%A8%E7%A4%BA%E4%BD%BF%E7%94%A8%E7%9A%84%E6%98%AFcglib%E7%9A%84%E6%96%B9%E5%BC%8F%E8%BF%9B%E8%A1%8C%E4%BB%A3%E7%90%86%E7%9A%84 "this 后面有$$cglib表示使用的是cglib的方式进行代理的")<br />然后通过观察，我又发现了一个问题
```java
package com.example.demo.controller;


import com.example.demo.entity.Stu;
import com.example.demo.service.StuService;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
// 有aop切面这个类
@RestController
@RequestMapping("/stu")
public class StuController {

    public StuController() {
        System.out.println("StuController");
    }

    @Resource
    private StuService stuService;
// 如果只是调用这个方法，因为是private，所以是代理不了的，应该走正常实例化的
    @PostMapping
    private void save(@RequestBody Stu stu){
        // 
        Stu stuOne= stuService.getById(stu.getId());
        System.out.println(stuOne);
        stuService.saveOrUpdate(stu);
    }
    // 如果调用这个方法，可以进行代理，应该走cglib代理的
    @PostMapping("test")
    public void test(@RequestBody Stu stu){
        Stu stuOne= stuService.getById(stu.getId());
        System.out.println(stuOne);
        stuService.saveOrUpdate(stu);
    }

}


```
根据spring中一个类，理论上是单例的。所以最开始实例化的springbean将被后面的aop cglib代理类替换掉，导致调用private方法时，也是使用的cglib代理类。<br />![在后面的实验中，private接口反而为cglib代理的](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690255929866-026e6187-65ae-453e-9548-3dc4d5a2b0c0.png#averageHue=%23778361&clientId=ueacd82d6-c665-4&from=paste&height=479&id=u56c03648&originHeight=718&originWidth=1748&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=216139&status=done&style=none&taskId=ua08e20b9-1a05-414a-8302-081c3e34aae&title=%E5%9C%A8%E5%90%8E%E9%9D%A2%E7%9A%84%E5%AE%9E%E9%AA%8C%E4%B8%AD%EF%BC%8Cprivate%E6%8E%A5%E5%8F%A3%E5%8F%8D%E8%80%8C%E4%B8%BAcglib%E4%BB%A3%E7%90%86%E7%9A%84&width=1165.3333333333333 "在后面的实验中，private接口反而为cglib代理的")<br />![public接口反而是正常的](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690256081876-692d940d-b0f0-43fe-b89e-da1f22b601e7.png#averageHue=%23607948&clientId=ueacd82d6-c665-4&from=paste&height=401&id=ubb53101f&originHeight=601&originWidth=1776&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=141515&status=done&style=none&taskId=uef086a94-f8e0-42fe-b2d0-0131ceeffa7&title=public%E6%8E%A5%E5%8F%A3%E5%8F%8D%E8%80%8C%E6%98%AF%E6%AD%A3%E5%B8%B8%E7%9A%84&width=1184 "public接口反而是正常的")

![然后中间的栈堆发现了cglib的存在，猜测：原本是cglib代理的类，然后通过aop后就变成了实例化的类？](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690256257196-a0858650-7845-430c-b559-ca43169dd819.png#averageHue=%235e8449&clientId=ueacd82d6-c665-4&from=paste&height=589&id=ube3d4ea3&originHeight=883&originWidth=1838&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=241975&status=done&style=none&taskId=ude679a2d-d3b9-4595-88b9-0ffc6802c67&title=%E7%84%B6%E5%90%8E%E4%B8%AD%E9%97%B4%E7%9A%84%E6%A0%88%E5%A0%86%E5%8F%91%E7%8E%B0%E4%BA%86cglib%E7%9A%84%E5%AD%98%E5%9C%A8%EF%BC%8C%E7%8C%9C%E6%B5%8B%EF%BC%9A%E5%8E%9F%E6%9C%AC%E6%98%AFcglib%E4%BB%A3%E7%90%86%E7%9A%84%E7%B1%BB%EF%BC%8C%E7%84%B6%E5%90%8E%E9%80%9A%E8%BF%87aop%E5%90%8E%E5%B0%B1%E5%8F%98%E6%88%90%E4%BA%86%E5%AE%9E%E4%BE%8B%E5%8C%96%E7%9A%84%E7%B1%BB%EF%BC%9F&width=1225.3333333333333 "然后中间的栈堆发现了cglib的存在，猜测：原本是cglib代理的类，然后通过aop后就变成了实例化的类？")<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690256411526-1d58732d-3bc2-4be5-b207-2814b8a77752.png#averageHue=%239ba28d&clientId=ueacd82d6-c665-4&from=paste&height=229&id=ue9372089&originHeight=343&originWidth=1823&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=81434&status=done&style=none&taskId=u85e35804-649c-4db3-9525-2c1642bb948&title=&width=1215.3333333333333)<br />![的确是spring代理的bean](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690257138276-b0d507e5-dda3-4b28-a91e-0509ce8385f4.png#averageHue=%233c4044&clientId=ueacd82d6-c665-4&from=paste&height=597&id=u8e172dff&originHeight=895&originWidth=1558&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=39149&status=done&style=none&taskId=ufc9a9d00-72db-4ac3-b38a-c77f46b876f&title=%E7%9A%84%E7%A1%AE%E6%98%AFspring%E4%BB%A3%E7%90%86%E7%9A%84bean&width=1038.6666666666667 "的确是spring代理的bean")

> `CglibAopProxy.processReturnType()`是Spring框架中CGLIB动态代理相关的方法之一。该方法是用来处理被代理方法的返回类型的转换。在Spring AOP中，动态代理会拦截被代理对象的方法调用，并在方法调用前后添加一些额外的逻辑（如事务管理、日志记录等）。
> 
> 当被代理方法执行完毕后，CGLIB动态代理会通过`processReturnType()`方法来处理方法的返回值。该方法的主要作用是将原始的返回值转换为代理对象期望的返回值类型，以确保在返回时返回的是代理对象，而不是原始对象。这样做的目的是保证后续对返回值的方法调用也能被动态代理拦截并进行处理。
> 
> 在Spring AOP中，如果一个方法的返回值类型是一个被代理的类型（即在AOP切面中被代理的类），Spring会使用`processReturnType()`方法来将原始返回值转换为代理对象。
> 
> 请注意，具体的方法实现可能会根据Spring版本和具体的代码变化而有所不同。如果你想深入了解`CglibAopProxy.processReturnType()`的实现细节，可以查看Spring框架的源代码。


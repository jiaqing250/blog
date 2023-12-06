[controller接口修饰符为private时，引发的问题](./controller接口修饰符为private时，引发的问题.md)
```java
@Service("stuService")
public class StuServiceImpl extends ServiceImpl<StuDao, Stu> implements StuService {

    @Resource
    public StuDao stuDao; //这里的修饰符为public
}
```
通过之前的实验证明，如果springbean采用cglib方式进行代理，那么可能会出现依赖注入的类为null。<br />![在直接调用的过程中发现，注入的类为null](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690360730662-20117339-9a3b-4f2f-8ac5-1e568a1e7ab9.png#averageHue=%23393e41&clientId=u37643411-5056-4&from=paste&height=379&id=ucac4f720&originHeight=568&originWidth=1558&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=31839&status=done&style=none&taskId=ub6471b8f-ecd4-4866-bb42-a13e643cf0d&title=%E5%9C%A8%E7%9B%B4%E6%8E%A5%E8%B0%83%E7%94%A8%E7%9A%84%E8%BF%87%E7%A8%8B%E4%B8%AD%E5%8F%91%E7%8E%B0%EF%BC%8C%E6%B3%A8%E5%85%A5%E7%9A%84%E7%B1%BB%E4%B8%BAnull&width=1038.6666666666667 "在直接调用的过程中发现，注入的类为null")<br />那么就能解决之前的几个疑惑

1. service层为什么要采用接口实现的方式来写
   1. 如果没有使用接口，直接写实现类。那么必定由cglib进行代理，在注入该bean后无法直接获取该类的公开的属性（此时一定为null）
   2. 在团队开发过程中，由于开发人员的质量参差不齐，一定会有人遇到这样的问题。为了避免出现这样的情况，通过实现接口的方式屏蔽掉直接调用公开的属性。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690361308878-346130c0-857d-443d-8b52-a7aa35e4fb11.png#averageHue=%235e7874&clientId=u37643411-5056-4&from=paste&height=416&id=udd7b68ac&originHeight=624&originWidth=1325&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=65023&status=done&style=none&taskId=ufd4acf79-04d9-4ee3-af28-ffdf7240271&title=&width=883.3333333333334)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690361337096-49cd23af-a796-40ac-9232-69839395bf74.png#averageHue=%23383d41&clientId=u37643411-5056-4&from=paste&height=275&id=u5a0e5bba&originHeight=412&originWidth=930&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=21311&status=done&style=none&taskId=u9ae4ac14-cf44-4a0b-9b14-d2206cde51f&title=&width=620)<br />![stuService.stuDao=null 验证成功](https://cdn.nlark.com/yuque/0/2023/png/2923644/1690361408480-adae8ea6-7475-443a-b362-bb23fefed7f8.png#averageHue=%2375894e&clientId=u37643411-5056-4&from=paste&height=569&id=uc5dea317&originHeight=853&originWidth=1395&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=182792&status=done&style=none&taskId=u96d7dcab-da21-4717-823b-10644b3985f&title=stuService.stuDao%3Dnull%20%E9%AA%8C%E8%AF%81%E6%88%90%E5%8A%9F&width=930 "stuService.stuDao=null 验证成功")

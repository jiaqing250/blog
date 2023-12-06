### 1、dbunit 字段不能为空的问题
[dbUnit / Bugs / #363 Regression: cannot insert an empty string](https://sourceforge.net/p/dbunit/bugs/363/)<br />![解决办法](https://cdn.nlark.com/yuque/0/2023/png/2923644/1673579676674-f3bebe2c-03b2-4429-bcef-da1afa9434c8.png#averageHue=%23302d2c&clientId=u1c194fb6-1750-4&from=paste&height=267&id=udbe53a57&originHeight=401&originWidth=1693&originalType=binary&ratio=1&rotation=0&showTitle=true&size=72180&status=done&style=none&taskId=u9d723994-835f-4f60-be68-0da2f56d28d&title=%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95&width=1128.6666666666667 "解决办法")

1. 为啥要这样做？在实际的数据表中有text类型的字段，值是可以为空字符串或者null的

### 2、spock commit 后格式错乱的问题，留一行即可解决
![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1673589382416-0297bf83-4c9d-49ee-9d57-59ff6d98d82b.png#averageHue=%232e2d2c&clientId=u029a40c7-7c11-4&from=paste&height=427&id=u3063004a&originHeight=641&originWidth=1599&originalType=binary&ratio=1&rotation=0&showTitle=false&size=128285&status=done&style=none&taskId=u767444fb-7ca9-4e74-8234-1b3ac8a38ff&title=&width=1066)
### 3、记录一下testme 模板
这个testme是idea的一个插件，需要先安装
```groovy

#parse("TestMe macros bak.groovy")
#set($hasMocks=$MockitoMockBuilder.hasMockable($TESTED_CLASS.fields))

#if($PACKAGE_NAME)
package ${PACKAGE_NAME}
#end
import spock.lang.*
import org.mapstruct.factory.Mappers
import cn.xxx.supper.MapperUtil
import cn.xxx.supper.MyBaseSpec


#parse("File Header.java")
class ${CLASS_NAME}  extends MyBaseSpec {


    private $TESTED_CLASS.canonicalName $StringUtils.deCapitalizeFirstLetter($TESTED_CLASS.name) 
    def setup() {
        $StringUtils.deCapitalizeFirstLetter($TESTED_CLASS.name) = new $TESTED_CLASS.canonicalName ();
        #grSetUpRenderBaseMapper($StringUtils.deCapitalizeFirstLetter($TESTED_CLASS.name),$TESTED_CLASS.fields)
    }
#foreach($method in $TESTED_CLASS.methods)
#if($TestSubjectUtils.shouldBeTested($method))
    #set( $method_name= $StringUtils.camelCaseToWords($method.name))
    /// region $method_name 方法测试区域
   
    @Unroll
    def "单元测试: $method_name"() {
#if($MockitoMockBuilder.shouldStub($method,$TESTED_CLASS.fields))
        given:
        // todo init param 

#end
        when:
        #grRenderMethodCall($method,$TESTED_CLASS.name)

        then:
            pass
        // todo check return value
    }
    ///endregion
#end
#end
}

```
```velocity
#parse("TestMe common macros.java")
################## Global vars ###############
#set($grReplacementTypesStatic = {
"java.util.Collection": "[<val>]", "java.util.Deque": "new LinkedList([<val>])", "java.util.List": "[<val>]", "java.util.Map": "[<val>:<val>]", "java.util.NavigableMap": "new java.util.TreeMap([<val>:<val>])", "java.util.NavigableSet": "new java.util.TreeSet([<val>])", "java.util.Queue": "new java.util.LinkedList<types>([<val>])", "java.util.RandomAccess": "new java.util.Vector([<val>])", "java.util.Set": "[<val>] as java.util.Set<types>", "java.util.SortedSet": "[<val>] as java.util.SortedSet<types>", "java.util.LinkedList": "new java.util.LinkedList<types>([<val>])", "java.util.ArrayList": "[<val>]", "java.util.HashMap": "[<val>:<val>]", "java.util.TreeMap": "new java.util.TreeMap<types>([<val>:<val>])", "java.util.LinkedList": "new java.util.LinkedList<types>([<val>])", "java.util.Vector": "new java.util.Vector([<val>])", "java.util.HashSet": "[<val>] as java.util.HashSet", "java.util.Stack": "new java.util.Stack<types>(){{push(<VAL>)}}", "java.util.LinkedHashMap": "[<val>:<val>]", "java.util.TreeSet": "[<val>]
as java.util.TreeSet" }) #set($grReplacementTypes = $grReplacementTypesStatic.clone())
#set($grReplacementTypesForReturn = $grReplacementTypesStatic.clone()) #foreach($javaFutureType in
$TestSubjectUtils.javaFutureTypes)
#evaluate(${grReplacementTypes.put($javaFutureType,"java.util.concurrent.CompletableFuture.completedFuture(<val>)")})
#end #foreach($javaFutureType in $TestSubjectUtils.javaFutureTypes)
#evaluate(${grReplacementTypesForReturn.put($javaFutureType,"<val>")})
#end
#set($grDefaultTypeValues = {
"byte": "(byte)0",
"short": "(short)0",
"int": "0",
"long": "0l",
"float": "0f",
"double": "0d",
"char": "(char)'a'",
"boolean": "true",
"java.lang.Byte": """00110"" as Byte",
"java.lang.Short": "(short)0",
"java.lang.Integer": "0",
"java.lang.Long": "1l",
"java.lang.Float": "1.1f",
"java.lang.Double": "0d",
"java.lang.Character": "'a' as Character",
"java.lang.Boolean": "Boolean.TRUE",
"java.math.BigDecimal": "0 as java.math.BigDecimal",
"java.math.BigInteger": "0g",
"java.util.Date": "new java.util.GregorianCalendar($YEAR, java.util.Calendar.$MONTH_NAME_EN.toUpperCase(), $DAY_NUMERIC, $HOUR_NUMERIC, $MINUTE_NUMERIC).getTime()",
"java.time.LocalDate": "java.time.LocalDate.of($YEAR, java.time.Month.$MONTH_NAME_EN.toUpperCase(), $DAY_NUMERIC)",
"java.time.LocalDateTime": "java.time.LocalDateTime.of($YEAR, java.time.Month.$MONTH_NAME_EN.toUpperCase(), $DAY_NUMERIC, $HOUR_NUMERIC, $MINUTE_NUMERIC, $SECOND_NUMERIC)",
"java.time.LocalTime": "java.time.LocalTime.of($HOUR_NUMERIC, $MINUTE_NUMERIC, $SECOND_NUMERIC)",
"java.time.Instant": "java.time.LocalDateTime.of($YEAR, java.time.Month.$MONTH_NAME_EN.toUpperCase(), $DAY_NUMERIC, $HOUR_NUMERIC, $MINUTE_NUMERIC, $SECOND_NUMERIC).toInstant(java.time.ZoneOffset.UTC)",
"java.io.File": "new File(getClass().getResource(""/$PACKAGE_NAME.replace('.','/')/PleaseReplaceMeWithTestFile.txt"").getFile())",
"java.lang.Class": "Class.forName(""$TESTED_CLASS.canonicalName"")"
})
##
##
################## Macros #####################
#macro(grRenderTestSubjectInit $testedClass $hasTestableInstanceMethod $hasMocks)
#if($hasMocks)
@InjectMocks
$testedClass.canonicalName $StringUtils.deCapitalizeFirstLetter($testedClass.name)
#elseif($hasTestableInstanceMethod)
$testedClass.canonicalName $StringUtils.deCapitalizeFirstLetter($testedClass.name)= $TestBuilder.renderInitType($testedClass,"$testedClass.name",$grReplacementTypes,$grDefaultTypeValues)
#end
#end
##
#macro(grRenderMockedFields $testedClassFields)
#foreach($field in $testedClassFields)
#if($MockitoMockBuilder.isMockable($field))
@Mock
$field.type.canonicalName $field.name
#elseif($MockitoMockBuilder.isMockExpected($field))
$MockitoMockBuilder.getImmockabiliyReason("//",$field)
#end
#end
#end
##
## 
#macro(grRenderBaseMapper $testedClassFields)
#foreach($field in $testedClassFields)
#if($field.name.indexOf("Mapper")>-1)
$field.type.canonicalName $field.name = MapperUtil.getMapper($field.type.canonicalName)
#elseif($field.name.indexOf("Converter")>-1)
$field.type.canonicalName $field.name = Mappers.getMapper($field.type.canonicalName)
#elseif($MockitoMockBuilder.isMockExpected($field))
$MockitoMockBuilder.getImmockabiliyReason("//",$field)
#end
#end
#end
##
## 
#macro(grSetUpRenderBaseMapper $testClassName,$testedClassFields)
#foreach($field in $testedClassFields)
#if($field.name  == "baseMapper")
$testClassName . $field.name = MapperUtil.getMapper($StringUtils.capitalizeFirstLetter($StringUtils.removeSuffix($testClassName,"ServiceImpl").concat("Mapper")))
#elseif($field.name.indexOf("Mapper")>-1)
$testClassName . $field.name = MapperUtil.getMapper($field.type.canonicalName)
#elseif($field.name.indexOf("Converter")>-1)
$testClassName . $field.name = Mappers.getMapper($field.type.canonicalName)
#elseif($MockitoMockBuilder.isMockExpected($field))
$MockitoMockBuilder.getImmockabiliyReason("//",$field)
#end
#end
#end
##
#macro(grRenderAssert $method)
result#if($TestSubjectUtils.isJavaFuture($method.returnType)).get()#end == $TestBuilder.renderReturnParam($method,$method.returnType,"replaceMeWithExpectedResult",$grReplacementTypesForReturn,$grDefaultTypeValues)
#end
##
#macro(grRenderMethodCall $method,$testedClassName)
#renderJavaReturnVar($method.returnType)#if($method.static)$testedClassName#{else}$StringUtils.deCapitalizeFirstLetter($testedClassName)#end.${method.name}($TestBuilder.renderMethodParams($method,$grReplacementTypes,$grDefaultTypeValues))
#end
##
#macro(grRenderParameterizedMethodCall $method,$testedClassName $methodClassParamsStr)
#if($method.static)$testedClassName#{else}$StringUtils.deCapitalizeFirstLetter($testedClassName)#end.${method.name}($methodClassParamsStr)##
#end
##
#macro(grRenderMockStubs $method $testedClassFields)
#foreach($field in $testedClassFields)
#if($MockitoMockBuilder.isMockable($field))
#foreach($fieldMethod in $field.type.methods)
#if($fieldMethod.returnType && $fieldMethod.returnType.name !="void" && $TestSubjectUtils.isMethodCalled($fieldMethod,$method))
when($field.name.${fieldMethod.name}($MockitoMockBuilder.buildMockArgsMatchers(${fieldMethod.methodParams},"Java"))).thenReturn($TestBuilder.renderReturnParam($method,$fieldMethod.returnType,"${fieldMethod.name}Response",$grReplacementTypes,$grDefaultTypeValues))
#end
#end
#end
#end
#end
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1673338157030-e2e411c4-1c75-4a08-9c6c-c0ac33a8d92f.png#averageHue=%23494f50&clientId=u0a0c90e4-346d-4&from=paste&height=673&id=u3743f39d&originHeight=1009&originWidth=1472&originalType=binary&ratio=1&rotation=0&showTitle=false&size=152358&status=done&style=none&taskId=u45eccf30-55eb-4236-b042-5e4fca3008d&title=&width=981.3333333333334)
### 4、按照目前的idea plugin 开发一套适合spock格式的dbunit插件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1673252241469-be29b906-4909-4f77-8670-b48003eadf36.png#averageHue=%237f977a&clientId=u476a83e3-3760-4&from=paste&height=471&id=u0d03af11&originHeight=706&originWidth=1560&originalType=binary&ratio=1&rotation=0&showTitle=false&size=139606&status=done&style=none&taskId=u3d1a1251-c284-4657-b34c-2b36a4d9e50&title=&width=1040)
:::tips
skill (skill_id: "4",skill_name: "家居安装维修",parent_id: "0",sort: "1",level: "1",create_time: "1614061455573",update_time: "1644839217688")
:::
[dbunit-extractor.jar.zip](https://www.yuque.com/attachments/yuque/0/2023/zip/2923644/1687747817542-9b37d365-a484-40ce-ac9d-a51bd5764212.zip)<br />使用的时候把zip去掉，再导入到idea中

### 5、mybatis 一级缓存导致的脏读:
场景：在使用spock时发现的一个问题
```java
public class BusinessService{
    public void setSelfOwned(BusinessSelfOwnedDTO dto) {
        Business business = businessMapper.selectById(dto.getBusinessId());
        ApiException.throwError(null == business, ApiErrorCode.BUSINESS_NOT_FOUND);
        business.setIsSelfOwned(dto.getIsSelfOwned());
        //        businessMapper.updateById(business);
    }
}


    @DbUnit(
            content = {
                business(
                        "business_id": "1",
                        "business_name": "开发测试",
                        "mobile": "mymobile",
                        "password": "123",
                        "province_id": "310000",
                        "province": "上海市",
                        "city_id": "310100",
                        "city": "上海市",
                        "area_id": "310118",
                        "area": "青浦区",
                        "address": "城区",
                        "account_amount": "9748.8",
                        "income_amount": "0",
                        "income_amount_total": "10",
                        "code": "B2205051603150192",
                        "rate": "0.1",
                        "verification_type": "20",
                        "contacts": "Jeffery",
                        "contact_phone": "18888888888",
                        "is_enable": "1",
                        "type": "20",
                        "token": "000",
                        "is_self_owned": "3",
                        "self_owned_rate": "0.08",
                        "expire_time": "1674885382680"
                )
            }
    )
    @Unroll
    def "test setSelfOwned"() {
        given:
        def ownedDTO = new BusinessSelfOwnedDTO(businessId: businessId, isSelfOwned: isSelfOwned)
        when:
        businessService.setSelfOwned(ownedDTO)
        then:
        def business = businessService.businessMapper.selectById(businessId)
        business.getIsSelfOwned() == isSelfOwned
        println(business)
        where:
        businessId | isSelfOwned
        1 		   | 1
        1          | 0
        1          | 0
    }
```
发现无论怎么都是 pass
#### 解决方案：
配置 localCacheScope
```java
public class DataSourceHolder {
    //    public static SqlSessionFactory builder = new MybatisSqlSessionFactoryBuilder().build(DataSourceHolder.class.getClassLoader().getResourceAsStream("mybatis-config.xml"));
    public static SqlSessionFactory builder;
    public static Configuration configuration;

    public static DataSource dataSource = SpecUtils.inMemoryDataSource();


    static {
        //这种方式主要是为了兼容dbUnit
        MybatisSqlSessionFactoryBean factory = new MybatisSqlSessionFactoryBean();
        try {
            factory.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
            factory.setDataSource(SpecUtils.inMemoryDataSource());
            builder = factory.getObject();

            MybatisConfiguration configurationT = (MybatisConfiguration) builder.getConfiguration();
            //解决一级缓存导致的脏读
            //场景: 下面方法中没有update，在重复单元测试时发现查询出来的数据竟然是缓存的数据
            /*public void setSelfOwned(BusinessSelfOwnedDTO dto) {
                    Business business = businessMapper.selectById(dto.getBusinessId());
                    ApiException.throwError(null == business, ApiErrorCode.BUSINESS_NOT_FOUND);
                    business.setIsSelfOwned(dto.getIsSelfOwned());
                    //businessMapper.updateById(business);
                }
             */
            configurationT.setLocalCacheScope(LocalCacheScope.STATEMENT);
            GlobalConfig globalConfig = configurationT.getGlobalConfig();
            globalConfig.setMetaObjectHandler(new MybatisPlusHandler());
            configuration = builder.getConfiguration();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        //这里是原mybatis的方式
        //       SqlSessionFactory builder = new SqlSessionFactoryBuilder().build(SkillServiceImplSpec.class.getClassLoader().getResourceAsStream("mybatisTestConfiguration/mybatis-config.xml"));
        //这里是mybatisPlus 封装后的
        // 作用一:处理entity类和mysql中字段不一致的问题，java中是驼峰式，java是下划线
        // 作用二:处理mapper 继承 baseMapper 原mapper没有baseMapper中封装的statement
        // 缺点一:因为是非spring模式，在启动后所有的插件都会失效
        //解决非spring模式下导致metaObjectHandler缺失的问题,主要就是新增或者更新时，自动填充字段的问题
        /*MybatisConfiguration configurationT = (MybatisConfiguration) DataSourceHolder.builder.getConfiguration();
        GlobalConfig globalConfig = configurationT.getGlobalConfig();
        globalConfig.setMetaObjectHandler(new MybatisPlusHandler());
        configuration = DataSourceHolder.builder.getConfiguration();*/

    }
    public static void initTable() {
        String sql = FileUtil.readString("classpath:test/db/init_table.sql", StandardCharsets.UTF_8);
        new Sql(dataSource).execute(sql);
    }
}

```
[MyBatis一级缓存_xinyuan_java的博客-CSDN博客_mybatis localcachescope](https://blog.csdn.net/xinyuan_java/article/details/112718469)<br />![一二级缓存的关系](https://cdn.nlark.com/yuque/0/2022/png/2923644/1672365737154-58adf63f-1565-40e9-9c0e-39abb1557f38.png#averageHue=%23ecebd4&clientId=u3de4ca8b-6429-4&from=paste&id=ud861f7fb&originHeight=611&originWidth=640&originalType=url&ratio=1&rotation=0&showTitle=true&size=263395&status=done&style=none&taskId=uc3847f13-3871-465a-8069-ee513940b42&title=%E4%B8%80%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E7%9A%84%E5%85%B3%E7%B3%BB "一二级缓存的关系")<br />spock
### 6、spock单元测试框架的使用
```groovy
@SpringBootTest
@ContextConfiguration(classes = UserServiceImpl.class)
@WebAppConfiguration
class UserInfoSpec extends Specification {
        @Autowired
        UserServiceImpl userService
        @SpringBean
        UserMapper userMapper = Mock()
        def setup() {}
        @Unroll
        def "testSelectUserById"() {
            given:
            def user = new User(userId: userId, userName: userName, passWord: passWord);
            userMapper.seleteUserById(userId) >> user
            expect:
                userService.selectUserById(userId) == user
            where:
                userId   | userName | passWord
            1234567L | xiaoming | 123456
            12309756 | xiaohong | 4321134
        }
    }



作者：hancaicai
链接：https://ld246.com/article/1573907697146
        来源：链滴
协议：CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/
```
![包名规范路径](https://cdn.nlark.com/yuque/0/2020/png/418304/1606815821083-ce5b862b-9ecb-40b4-8322-b6a827b87885.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_6ZmG6b6f%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10#averageHue=%23fcfbfa&from=url&id=tRveZ&originHeight=555&originWidth=702&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E5%8C%85%E5%90%8D%E8%A7%84%E8%8C%83%E8%B7%AF%E5%BE%84 "包名规范路径")<br />参考：<br />[定位](https://www.yuque.com/lugew/spock/fqgfso?view=doc_embed)
```groovy
@Unroll("非法用户名：#exampleUsername")
    def "用户名需是2-16位数字、字母和中文混合"() {
        when: "初始化用户名"
        username = new Username(exampleUsername)
        then: "不符合规则时抛出异常"
        thrown(IllegalArgumentException)
        where:
        exampleUsername << [
                "1",
                "12345678901234561",
                "杜",
                "国破山河在凑足十七个个字才行啊是不",
                "有符号 。 所以不行",
                "英文.符号所以不行",
                "有空格 不行",
                "",
        ]

    }
```
项目参考:<br />[GitHub - imcamilo/kotlin-spring-project: Kotlin + MyBatis + Spock + Spring Boot + Docker](https://github.com/imcamilo/kotlin-spring-project)

![覆盖率](https://cdn.nlark.com/yuque/0/2022/png/2923644/1671070196810-9a212f4a-c872-4682-b3b4-8ff5bdbc07b6.png#averageHue=%2373895f&clientId=u1d6806ac-3ec6-4&from=paste&id=u52ed8e73&originHeight=387&originWidth=969&originalType=url&ratio=1&rotation=0&showTitle=true&size=54571&status=done&style=none&taskId=u28bcb79b-6fe0-40a8-a6e7-e7ac3770e49&title=%E8%A6%86%E7%9B%96%E7%8E%87 "覆盖率")<br />[IDEA集成jacoco - 紫陌花间客 - 博客园](https://www.cnblogs.com/richered/p/11749299.html)

![关于代码覆盖率，可以使用idea自带的coverage，勾选Tracing，可以看到分支是否完全覆盖。](https://cdn.nlark.com/yuque/0/2021/png/832571/1622478874175-0df3db19-5d6c-46d0-af83-516b5900c557.png#averageHue=%23252a33&from=url&id=PGDLI&originHeight=387&originWidth=897&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E5%85%B3%E4%BA%8E%E4%BB%A3%E7%A0%81%E8%A6%86%E7%9B%96%E7%8E%87%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8idea%E8%87%AA%E5%B8%A6%E7%9A%84coverage%EF%BC%8C%E5%8B%BE%E9%80%89Tracing%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E5%88%86%E6%94%AF%E6%98%AF%E5%90%A6%E5%AE%8C%E5%85%A8%E8%A6%86%E7%9B%96%E3%80%82 "关于代码覆盖率，可以使用idea自带的coverage，勾选Tracing，可以看到分支是否完全覆盖。")

[Spock单元测试框架介绍以及在美团优选的实践](https://tech.meituan.com/2021/08/06/spock-practice-in-meituan.html)<br />[Spock系列 - 老K的Java博客](https://javakk.com/category/spock)<br />github上的spock实战项目<br />[GitHub - lucas-myx/spock_example: Spock是国外一款功能强大的测试框架，但是官方的文档和代码示例不太适合我们实际的工程项目，无法解决我们项目中的复杂业务场景，需要找到一套适合自己项目的成熟解决方案，所以觉得有必要把我们项目中使用Spock的经验分享出来，帮助大家解决实际问题或带来一些启发，如果你在使用过程中遇到问题可以在公众号[Java老K]或我的博客 //javakk.com 交流沟通](https://github.com/lucas-myx/spock_example)
```groovy
 @Unroll
    def "test login with "(){
        when:
        userService.login(name, passwd)

        then:
        userDao.findByName("k") >> null
        userDao.findByName("k1") >> new User("k1", 12, "p")

        then:
        def e = thrown(RuntimeException)
        e.message == errMsg

        where:
        name        |   passwd  |   errMsg
        "k"         |   "k"     |   "${name}不存在"
        "k1"        |   "p1"     |   "${name}密码输入错误"

    }
```
#### 使用mybatisplus参考文章：
[MyBatis Plus_Kramer_149的博客-CSDN博客](https://blog.csdn.net/m0_46199937/article/details/121245533)<br />记载一下成功后的案例：
```groovy
package cn.xxx.manage.service.impl

import cn.xxx.common.handle.component.MybatisPlusHandler
import cn.xxx.common.mapper.GroupServiceMenuMapper
import cn.xxx.common.mapper.SkillMapper
import cn.xxx.common.mapper.WorkerSkillRelationMapper
import cn.xxx.manage.converter.SkillConverterImpl
import cn.xxx.manage.dto.skill.SkillSaveDto
import com.alibaba.testable.core.annotation.MockDiagnose
import com.alibaba.testable.core.model.LogLevel
import com.baomidou.mybatisplus.core.MybatisConfiguration
import com.baomidou.mybatisplus.core.MybatisSqlSessionFactoryBuilder
import org.apache.ibatis.session.SqlSessionFactory
import spock.lang.Specification
import spock.lang.Unroll

/*// 注意这里要使用 unittest 的配置文件了，里面使用H2数据库
@ActiveProfiles("unittest")
// 关于@Rollback(false)注解：正常情况下，SpringBoot的测试用例每执行完一个就会回滚一次，
// 比如说再执行完create entity test之后会立马将刚刚insert进去的数据清除，之后执行“get entity test”的时候应该是测试用例不通过的。
// 加了这个注解后，就可以指定某个测试方法，或者某个测试类的执行结果不进行回滚。
@Rollback
@MybatisPlusTest*/

//@ActiveProfiles("unittest")
//@MybatisPlusTest
class SkillServiceImplSpec extends Specification {

    private SkillServiceImpl skillService;


    void setup() {
        //这里是原mybatis的方式
        //       SqlSessionFactory builder = new SqlSessionFactoryBuilder().build(SkillServiceImplSpec.class.getClassLoader().getResourceAsStream("mybatisTestConfiguration/SkillMapperTestConfiguration.xml"));
        //这里是mybatisPlus 封装后的
        // 作用一:处理entity类和mysql中字段不一致的问题，java中是驼峰式，java是下划线
        // 作用二:处理mapper 继承 baseMapper 原mapper没有baseMapper中封装的statement
        // 缺点一:因为是非spring模式，在启动后所有的插件都会失效
        SqlSessionFactory builder = new MybatisSqlSessionFactoryBuilder().build(SkillServiceImplSpec.class.getClassLoader().getResourceAsStream("mybatisTestConfiguration/SkillMapperTestConfiguration.xml"));
        //解决非spring模式下导致metaObjectHandler缺失的问题,主要就是新增或者更新时，自动填充字段的问题
        def configuration = (MybatisConfiguration) builder.getConfiguration()
        configuration.getGlobalConfig().metaObjectHandler = new MybatisPlusHandler();
//        //you can use builder.openSession(false) to not commit to database

        SkillMapper mapper = builder.getConfiguration().getMapper(SkillMapper.class, builder.openSession(true));
        skillService = new SkillServiceImpl();
        skillService.baseMapper = mapper
        skillService.skillMapper = mapper
        skillService.skillConverter = new SkillConverterImpl();
        skillService.groupServiceMenuMapper = Stub(GroupServiceMenuMapper)
        skillService.workerSkillRelationMapper = Stub(WorkerSkillRelationMapper)
    }

    @MockDiagnose(LogLevel.VERBOSE)
    public static class Mock {

    }

    @Unroll
    def "test save"() {
        given: "修改3级"
        def dto = new SkillSaveDto(skillId: 3, skillName: "新三级名称", parentId: 2, level: 3)
        when: "不存在服务菜单数据,不存在服务数据"
        skillService.save(dto)
        then:
        println "执行成功"

    }


}

```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- Mybatis config sample -->
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>


  <environments default = "default">
    <environment id="default">
      <transactionManager type="JDBC"/>
      <dataSource type="UNPOOLED">
        <property name = "driver" value = "com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://"/>
        <property name="username" value=""/>
        <property name="password" value=""/>
      </dataSource>
    </environment>
  </environments>



  <mappers>
    <mapper resource="mapper/SkillMapper.xml"/>
  </mappers>
</configuration>

```
dbunit 参考：<br />[https://github.com/janbols/spock-dbunit](https://github.com/janbols/spock-dbunit)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/2923644/1671432204998-4f6ad0f8-a81e-4fb2-930c-a690c7fe7c38.png#averageHue=%23f5f4f3&clientId=u1ff29365-4506-4&from=paste&height=204&id=u4e9dc3e5&originHeight=306&originWidth=931&originalType=binary&ratio=1&rotation=0&showTitle=false&size=55857&status=done&style=none&taskId=ucf4e6adf-7ff2-48bb-a3ad-46ffe8b2125&title=&width=620.6666666666666)<br />驼峰转下划线:<br />[在线工具-驼峰转下划线,下划线转驼峰-BeJSON.com](https://www.bejson.com/convert/camel_underscore/)<br />[有赞单元测试实践](https://tech.youzan.com/youzan-test-practice/)

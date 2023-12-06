```java

#parse("TestMe common macros.java")
################## Global vars ###############
#set($grReplacementTypesStatic = {
    "java.util.Collection": "[]", "java.util.Deque": "new LinkedList([])", "java.util.List": "[]", "java.util.Map": "[]", "java.util.NavigableMap": "new java.util.TreeMap([<val>:<val>])", "java.util.NavigableSet": "new java.util.TreeSet([<val>])", "java.util.Queue": "new java.util.LinkedList<types>([<val>])", "java.util.RandomAccess": "new java.util.Vector([<val>])", "java.util.Set": "[<val>] as java.util.Set<types>", "java.util.SortedSet": "[<val>] as java.util.SortedSet<types>", "java.util.LinkedList": "new java.util.LinkedList<types>([<val>])", "java.util.ArrayList": "[<val>]", "java.util.HashMap": "[<val>:<val>]", "java.util.TreeMap": "new java.util.TreeMap<types>([<val>:<val>])", "java.util.LinkedList": "new java.util.LinkedList<types>([<val>])", "java.util.Vector": "new java.util.Vector([<val>])", "java.util.HashSet": "[<val>] as java.util.HashSet", "java.util.Stack": "new java.util.Stack<types>(){{push(<VAL>)}}", "java.util.LinkedHashMap": "[<val>:<val>]", "java.util.TreeSet": "[<val>]
    as java.util.TreeSet" }) #set($grReplacementTypes = $grReplacementTypesStatic.clone())
#set($grReplacementTypesForReturn = $grReplacementTypesStatic.clone()) #foreach($javaFutureType in
    $TestSubjectUtils.javaFutureTypes)
    #evaluate(${grReplacementTypes.put($javaFutureType,"java.util.concurrent.CompletableFuture.completedFuture(<val>)")})
#end #foreach($javaFutureType in $TestSubjectUtils.javaFutureTypes)
    #evaluate(${grReplacementTypesForReturn.put($javaFutureType,"<val>")})
#end
#set ($CLASS_NAME = $CLASS_NAME.replace("Test", "Spec"))
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
```java

#parse("TestMe macros bak.groovy")
#set($hasMocks=$MockitoMockBuilder.hasMockable($TESTED_CLASS.fields))

#if($PACKAGE_NAME)
package ${PACKAGE_NAME}
#end

import spock.lang.*
import org.mapstruct.factory.Mappers
import org.dbunit.supper.MapperUtil
import org.dbunit.supper.MyBaseSpec




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

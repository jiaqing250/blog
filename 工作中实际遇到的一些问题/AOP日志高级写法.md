场景：在日程开发过程中，通常需要记录某接口操作的日志，但是需要的参数需要通过查询数据库后才有，有的数据需要当前接口操作完成之后才有的，或者更加复杂的自定义规则。那么普通方式的aop切面日志就不太够用了<br />先看使用后的案例
```java
@RestController
@RequestMapping("/job")
@Api(tags = "职位模块")
@OperationLog(logClass = JobLogService.class) //标记使用的logService类
public class JobController {
    @OperationLogHandler(isAfterMethod = true) //标记需要保存日志的方法 isAfterMethod 表示需要方法执行后再保存日志
    @PostMapping("/operate/status")
    @ApiOperation("职位状态操作（审核职位/分配负责人/拒绝/停止招聘）")
    public void operateStatus(@RequestBody @Valid JobOperateDTO jobOperateDto) {
        jobService.operate(userDto, jobOperateDto);
    }
}
//日志服务类
@Service
public class JobLogService {
     @Resource
    private JobService jobService;
    @Resource
    private SysUserMapper sysUserMapper;
    //对应的方法  需要方法名称 参数完全一致  返回类型为自定义的类(Content)
    public Content operateStatus(JobOperateDTO jobOperateDto) {
        Optional<Status> statusOptional = Arrays.stream(Status.values()).filter(status -> status.getItem().equals(jobOperateDto.getStatus())).findFirst();
        if (statusOptional.isPresent()) {
            JobModelVO jobModelVO = jobService.selectJobModelVOById(jobOperateDto.getJobId());
            Content content = new Content();
            switch (statusOptional.get()) {
                case AUDITING:
                    content.setBehavior(BehaviorEnum.JOB_AUDIT);
                    content.setContent(StrUtil.format("【通过】【{companyName}】发布的【{jobName}】", BeanUtil.beanToMap(jobModelVO)));
                    break;
                case ALLOCATED:
                    content.setBehavior(BehaviorEnum.JOB_ALLOCATED);
                    Map<String, Object> map = BeanUtil.beanToMap(jobModelVO);
                    map.put("sysUserNameMobile", "无");
                    if (jobOperateDto.getSysUserIdList() != null && !jobOperateDto.getSysUserIdList().isEmpty()) {
                        List<SysUserEntity> sysUserEntityList = sysUserMapper.selectBatchIds(jobOperateDto.getSysUserIdList());
                        if (CollUtil.isNotEmpty(sysUserEntityList)) {
                            String sysUserNameMobile = sysUserEntityList.stream().map(x -> StrUtil.format("{}/{}", x.getRealName(), x.getMobile())).collect(Collectors.joining(","));
                            map.put("sysUserNameMobile", sysUserNameMobile);
                        }
                    }
                    content.setContent(StrUtil.format("【分配】【{companyName}】发布的【{jobName}】给【{sysUserNameMobile}】", map));
                    break;
                case REJECTED:
                    content.setBehavior(BehaviorEnum.JOB_AUDIT);
                    content.setContent(StrUtil.format("【不通过】【{companyName}】发布的【{jobName}】【{sysUserFailRemark}】", BeanUtil.beanToMap(jobModelVO)));
                    break;
                case STOP:
                    content.setBehavior(BehaviorEnum.JOB_STOP);
                    content.setContent(StrUtil.format("【停止招聘】【{companyName}】发布的【{jobName}】", BeanUtil.beanToMap(jobModelVO)));
                    break;
                case RESTORE_HIRING:
                    content.setBehavior(BehaviorEnum.JOB_RESTORE_HIRING);
                    content.setContent(StrUtil.format("【{companyName}】【{jobName}】恢复招聘", BeanUtil.beanToMap(jobModelVO)));
                    break;
                default:
                    break;
            }
            return content;
        }
        return null;
    }
}
```
需要使用的类以及注解
```java
/**
 * 系统日志注解 作用在类上
 * 作用在复杂情况下的日志场景下<br/>
 * 1、新建一个spring bean  命名为 xxxLogService<br/>
 * 2、选择要记录日志的一个方法(比如void save(Dto dto))，复制到 xxxLogService 类，返回值必须为Content 或者 List< Content ><br/>
 * 3、在方法 save 上加上注解@OperationLogHandler
 *
 * @author
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
public @interface OperationLog {
    /**
     * 日志代理类 必填
     */
    Class logClass() default void.class;
}
```
```java


import cn.xx.common.constant.BehaviorEnum;

import java.lang.annotation.*;

/**
 * 日志记录核心注解，作用在具体的方法上
 * 复杂情况下搭配注解 @OperationLog 在类上一起使用
 *
 * @author
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Documented
public @interface OperationLogHandler {


    /**
     * 操作 日志枚举类  简单使用时必填
     */
    BehaviorEnum behavior() default BehaviorEnum.NULL;

    /**
     * 操作对象 一般值参数对象 简单使用时必填
     */
    String object() default "";

    /**
     * 操作详情 在简单的情况下使用，比如参数已经有日志需要的数据了
     */
    String content() default "";

    /**
     * 默认为false
     * false 注解在Controller,需要自己重写操作日志逻辑,返回Content
     * true 注解在操作记录实现类,针对已经写过操作记录的方法,直接修改操作记录返回值为Content类型,切面直接获取返回值。
     */
    boolean isReturn() default false;

    /**
     * 是否在方法执行完后调用
     *
     * @return boolean
     */
    boolean isAfterMethod() default false;

}

```
```java


import cn.xx.common.annotation.OperationLog;
import cn.xx.common.annotation.OperationLogHandler;
import cn.xx.common.constant.Constant;
import cn.xx.common.convert.UserOperationLogConverter;
import cn.xx.common.dto.log.Content;
import cn.xx.common.dto.log.OperationLogEntity;
import cn.xx.common.dto.user.UserDto;
import cn.xx.common.entity.SysUserEntity;
import cn.xx.common.entity.UserOperationLog;
import cn.xx.common.mapper.UserOperationLogMapper;
import cn.xx.common.utils.ApplicationUtils;
import cn.xx.common.utils.SpringContextUtils;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.extra.servlet.ServletUtil;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.PropertyFilter;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.context.annotation.Scope;
import org.springframework.core.DefaultParameterNameDiscoverer;
import org.springframework.core.annotation.Order;
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.Expression;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.OutputStream;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Objects;

/**
 * 日志切面
 */
@Slf4j
@Aspect
@Component
@Order(2)
@Scope("prototype")
public class OperationLogAspect {

    /**
     * 用于SpEL表达式解析.
     */
    private SpelExpressionParser spelExpressionParser = new SpelExpressionParser();
    /**
     * 用于获取方法参数定义名字.
     */
    private DefaultParameterNameDiscoverer nameDiscoverer = new DefaultParameterNameDiscoverer();

    @Pointcut("@within(operationLogHandler) || @annotation(operationLogHandler)")
    public void point(OperationLogHandler operationLogHandler) {
    }

    private List<OperationLogEntity> logEntityList;

    @SuppressWarnings("unchecked")
    @Before(value = "point(operationLogHandler)", argNames = "joinPoint,operationLogHandler")
    public void beforeMethod(JoinPoint joinPoint, OperationLogHandler operationLogHandler) {
        OperationLog sysLog = joinPoint.getTarget().getClass().getDeclaredAnnotation(OperationLog.class);
        if (operationLogHandler == null) {
            return;
        }
        logEntityList = new ArrayList<>();
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        OperationLogEntity operationLogEntity = getLog(joinPoint);

        try {
            if (operationLogHandler.isReturn()) {
                logEntityList.add(operationLogEntity);
            } else if (!operationLogHandler.isAfterMethod()) {
                if (StrUtil.isNotEmpty(operationLogHandler.object()) && StrUtil.isNotEmpty(operationLogHandler.content())) {
                    String object = operationLogHandler.object();
                    String content = operationLogHandler.content();
                    if (object.contains("#")) {
                        //获取方法参数值
                        Object[] args = joinPoint.getArgs();
                        object = getValBySpEL(object, methodSignature, args);
                    }
                    if (content.contains("#")) {
                        //获取方法参数值
                        Object[] args = joinPoint.getArgs();
                        content = getValBySpEL(content, methodSignature, args);
                    }
                    operationLogEntity.setContent(content);
                    operationLogEntity.setObject(object);
                    operationLogEntity.setBehavior(operationLogHandler.behavior().getTitle());
                    logEntityList.add(operationLogEntity);
                } else {
                    Method method = methodSignature.getMethod();
                    Class proxyClass = sysLog.logClass();
                    Method logMethod = proxyClass.getDeclaredMethod(method.getName(), method.getParameterTypes());
                    Object object = SpringContextUtils.getBean(proxyClass);
                    Object logInvoke = logMethod.invoke(object, joinPoint.getArgs());
                    logEntityList.addAll(transferLogEntity(logInvoke, operationLogEntity, operationLogHandler));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            log.error("日志记录异常:{}", e.getMessage());
        }
    }

    @AfterReturning(value = "point(operationLogHandler)", argNames = "joinPoint,operationLogHandler,val", returning = "val")
    public void afterMethodReturning(JoinPoint joinPoint, OperationLogHandler operationLogHandler, Object val) {
        if (operationLogHandler == null) {
            return;
        }
        try {
            if (operationLogHandler.isReturn()) {
                OperationLogEntity operationLogEntity = logEntityList.get(0);
                logEntityList = new ArrayList<>();
                logEntityList.addAll(transferLogEntity(val, operationLogEntity, operationLogHandler));
            } else if (operationLogHandler.isAfterMethod()) {
                OperationLog sysLog = joinPoint.getTarget().getClass().getDeclaredAnnotation(OperationLog.class);
                logEntityList = new ArrayList<>();
                MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
                OperationLogEntity operationLogEntity = getLog(joinPoint);

                if (StrUtil.isNotEmpty(operationLogHandler.object()) && StrUtil.isNotEmpty(operationLogHandler.content())) {
                    String object = operationLogHandler.object();
                    String content = operationLogHandler.content();
                    if (object.contains("#")) {
                        //获取方法参数值
                        Object[] args = joinPoint.getArgs();
                        object = getValBySpEL(object, methodSignature, args);
                    }
                    if (content.contains("#")) {
                        //获取方法参数值
                        Object[] args = joinPoint.getArgs();
                        content = getValBySpEL(content, methodSignature, args);
                    }
                    operationLogEntity.setContent(content);
                    operationLogEntity.setObject(object);
                    operationLogEntity.setBehavior(operationLogHandler.behavior().getTitle());
                    logEntityList.add(operationLogEntity);
                } else {
                    Method method = methodSignature.getMethod();
                    Class proxyClass = sysLog.logClass();
                    Method logMethod = proxyClass.getDeclaredMethod(method.getName(), method.getParameterTypes());
                    Object object = SpringContextUtils.getBean(proxyClass);
                    Object logInvoke = logMethod.invoke(object, joinPoint.getArgs());
                    logEntityList.addAll(transferLogEntity(logInvoke, operationLogEntity, operationLogHandler));
                }
            }

            for (OperationLogEntity operationLogEntity : logEntityList) {
                //封禁状态不记录日志
                if (operationLogEntity.isBan()) {
                    break;
                }
                UserOperationLog userOperationLog = SpringContextUtils.getBean(UserOperationLogConverter.class).toEntity(operationLogEntity);
                if (userOperationLog != null) {
                    if (operationLogEntity.getTargetUser() != null) {
                        UserDto targetUser = operationLogEntity.getTargetUser();
                        userOperationLog.setTargetUserId(Integer.parseInt(targetUser.getUserId().toString()));
                        userOperationLog.setTargetUserName(targetUser.getUserName());
                        userOperationLog.setTargetUserMobile(targetUser.getUserMobile());
                        userOperationLog.setTargetUserType(targetUser.getUserType());
                    }
                    SpringContextUtils.getBean(UserOperationLogMapper.class).insert(userOperationLog);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            log.error("日志记录异常:{}", e.getMessage());
        }
    }

    @SuppressWarnings("unchecked")
    private List<OperationLogEntity> transferLogEntity(Object obj, OperationLogEntity operationLogEntity, OperationLogHandler operationLogHandler) {
        List<OperationLogEntity> operationLogEntityList = new ArrayList<>();
        if (obj instanceof Content) {
            Content content = (Content) obj;
            operationLogEntity.setContent(content.getContent());
            operationLogEntity.setObject(content.getObject());
            operationLogEntity.setTargetUser(content.getTargetUser());
            if (content.getBehavior() == null) {
                operationLogEntity.setBehavior(operationLogHandler.behavior().getTitle());
            } else {
                operationLogEntity.setBehavior(content.getBehavior().getTitle());
            }
            operationLogEntityList.add(operationLogEntity);
        } else if (obj instanceof Collection) {
            List<Content> contentList = (List) obj;
            for (Content content : contentList) {
                OperationLogEntity operationLog = ObjectUtil.clone(operationLogEntity);
                operationLog.setContent(content.getContent());
                operationLog.setObject(content.getObject());
                operationLog.setTargetUser(content.getTargetUser());
                if (content.getBehavior() == null) {
                    operationLog.setBehavior(operationLogHandler.behavior().getTitle());
                } else {
                    operationLog.setBehavior(content.getBehavior().getTitle());
                }
                operationLogEntityList.add(operationLog);
            }
        }
        return operationLogEntityList;
    }

    /**
     * 构建日志对象
     */
    private OperationLogEntity getLog(JoinPoint joinPoint) {
        OperationLogEntity operationLogEntity = new OperationLogEntity();
        Object[] args = joinPoint.getArgs();
        List<Object> objectList = new ArrayList<>(args.length);
        for (Object arg : args) {
            boolean parse = parse(arg);
            if (parse) {
                objectList.add(arg);
            }
        }
        operationLogEntity.setArgs(JSON.toJSONString(objectList, new JsonFilter()));
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        operationLogEntity.setClassName(methodSignature.getDeclaringTypeName());
        operationLogEntity.setMethodName(methodSignature.getName());
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        if (attributes != null) {
            HttpServletRequest request = attributes.getRequest();
            String ipAddress = ServletUtil.getClientIP(request);
            operationLogEntity.setCreatorIp(ipAddress);
        }
        UserDto userInfo = ApplicationUtils.getUserInfo();
        if (userInfo != null) {
            operationLogEntity.setCreatorId(Integer.parseInt(String.valueOf(userInfo.getUserId())));
            operationLogEntity.setCreatorName(userInfo.getUserName());
            operationLogEntity.setCreatorMobile(userInfo.getUserMobile());
            operationLogEntity.setCreatorType(userInfo.getUserType());
            if (userInfo.getUserInfo() != null && userInfo.getUserInfo() instanceof SysUserEntity) {
                SysUserEntity sysUserEntity = (SysUserEntity) userInfo.getUserInfo();
                if (Objects.equals(sysUserEntity.getIsBan(), Constant.YES)) {
                    operationLogEntity.setBan(true);
                }
                operationLogEntity.setCreatorName(sysUserEntity.getRealName());
            }
        }
        return operationLogEntity;
    }

    class JsonFilter implements PropertyFilter {

        /**
         * @param object the owner of the property
         * @param name   the name of the property
         * @param value  the value of the property
         * @return true if the property will be included, false if to be filtered out
         */
        @Override
        public boolean apply(Object object, String name, Object value) {
            return parse(object) && parse(value);
        }
    }

    /**
     * 数据是否可以序列化
     *
     * @param object
     * @return true
     */
    private boolean parse(Object object) {
        if (object instanceof HttpServletRequest) {
            return false;
        } else if (object instanceof HttpServletResponse) {
            return false;
        } else if (object instanceof MultipartFile) {
            return false;
        } else if (object instanceof OutputStream) {
            return false;
        }
        return true;
    }

    /**
     * 解析spEL表达式
     */
    private String getValBySpEL(String spEL, MethodSignature methodSignature, Object[] args) {
        //获取方法形参名数组
        String[] paramNames = nameDiscoverer.getParameterNames(methodSignature.getMethod());
        if (paramNames != null && paramNames.length > 0) {
            Expression expression = spelExpressionParser.parseExpression(spEL);
            // spring的表达式上下文对象
            EvaluationContext context = new StandardEvaluationContext();
            // 给上下文赋值
            for (int i = 0; i < args.length; i++) {
                context.setVariable(paramNames[i], args[i]);
            }
            Object value = expression.getValue(context);
            return value != null ? value.toString() : null;
        }
        return null;
    }
}

```
OperationLogAspect类没有采用单例模式，原因：使用了局部变量临时保存数据，如果使用单例模式会有问题
```java


import lombok.AllArgsConstructor;
import lombok.Getter;

@AllArgsConstructor
@Getter
public enum BehaviorEnum {
    /**
     * 日志操作类型枚举类
     */
    LOGOUT("登出"),
    LOGIN("登录"),
    NULL(""),
    ;

    private String title;
}
```
```java

import cn.xx.common.constant.BehaviorEnum;
import cn.xx.common.dto.user.UserDto;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Content {
	
    //可能需要用到的
    private String object;
    //内容
    private String content;
	//枚举类 
    private BehaviorEnum behavior;

    @ApiModelProperty(value = "目标用户")
    private UserDto targetUser;


}

```
尽管涉及到的组件较多，但是一旦理解，就会明白这种方式非常便捷，可以处理更复杂的日志情况。<br />这套AOP日志的源头是悟空CRM，我对原始AOP日志逻辑进行了优化，而非简单搬运。

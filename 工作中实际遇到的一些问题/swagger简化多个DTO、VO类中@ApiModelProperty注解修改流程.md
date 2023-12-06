场景：在实际项目开发过程中，常常需要使用整数字段来表示某种含义。如果这个字段在多个接口中都被使用，那么需要在多个DTO类中添加@ApiModelProperty注解，以显示该字段的含义。然而，在项目迭代过程中，这样的修改需要进行多次，非常繁琐。为了简化这一流程，可以通过双亲委派机制来重写@ApiModelProperty注解，以添加自定义属性。<br />例子:
```java
@Data
public class CompanyInfoDTO {

    @ApiModelProperty(value = "人员规模人员规模（0~20人=10，20~50人=20，51~100人=30，101~300人=40，301~500人=50，501~1000人=60，1000人以上=70）")
    private Integer staffSize;
}
```
这种dto、vo类可能会有很多个，在后期维护过程中通常需要频繁修改注释。为了方便维护，我们选择覆盖@ApiModelProperty注解
```java
package io.swagger.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Adds and manipulates data of a model property.
 */
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiModelProperty {
    /**
     * A brief description of this property.
     */
    String value() default "";

    /**
     * Allows overriding the name of the property.
     *
     * @return the overridden property name
     */
    String name() default "";

    /**
     * Limits the acceptable values for this parameter.
     * <p>
     * There are three ways to describe the allowable values:
     * <ol>
     * <li>To set a list of values, provide a comma-separated list.
     * For example: {@code first, second, third}.</li>
     * <li>To set a range of values, start the value with "range", and surrounding by square
     * brackets include the minimum and maximum values, or round brackets for exclusive minimum and maximum values.
     * For example: {@code range[1, 5]}, {@code range(1, 5)}, {@code range[1, 5)}.</li>
     * <li>To set a minimum/maximum value, use the same format for range but use "infinity"
     * or "-infinity" as the second value. For example, {@code range[1, infinity]} means the
     * minimum allowable value of this parameter is 1.</li>
     * </ol>
     */
    String allowableValues() default "";

    /**
     * Allows for filtering a property from the API documentation. See io.swagger.core.filter.SwaggerSpecFilter.
     */
    String access() default "";

    /**
     * Currently not in use.
     */
    String notes() default "";

    /**
     * The data type of the parameter.
     * <p>
     * This can be the class name or a primitive. The value will override the data type as read from the class
     * property.
     */
    String dataType() default "";

    /**
     * Specifies if the parameter is required or not.
     */
    boolean required() default false;

    /**
     * Allows explicitly ordering the property in the model.
     */
    int position() default 0;

    /**
     * Allows a model property to be hidden in the Swagger model definition.
     */
    boolean hidden() default false;

    /**
     * A sample value for the property.
     */
    String example() default "";

    /**
     * Allows a model property to be designated as read only.
     *
     * @deprecated As of 1.5.19, replaced by {@link #accessMode()}
     *
     */
    @Deprecated
    boolean readOnly() default false;

    /**
     * Allows to specify the access mode of a model property (AccessMode.READ_ONLY, READ_WRITE)
     *
     * @since 1.5.19
     */
    AccessMode accessMode() default AccessMode.AUTO;


    /**
     * Specifies a reference to the corresponding type definition, overrides any other metadata specified
     */

    String reference() default "";

    /**
     * Allows passing an empty value
     *
     * @since 1.5.11
     */
boolean allowEmptyValue() default false;

/**
* @return an optional array of extensions
*/
Extension[] extensions() default @Extension(properties = @ExtensionProperty(name = "", value = ""));

enum AccessMode {
AUTO,
READ_ONLY,
READ_WRITE;
}

/**
* 动态注解参数
*/
Class<? extends Enum> enumDynamic() default Enum.class;

}
```
```java

public class UserCompany{
    @AllArgsConstructor
    @Getter
    enum StaffSize {
        SIZE_0_TO_20("0~20人", 10),
        SIZE_20_TO_50("20~50人", 20),
        SIZE_51_TO_100("51~100人", 30),
        SIZE_101_TO_300("101~300人", 40),
        SIZE_301_TO_500("301~500人", 50),
        SIZE_501_TO_1000("501~1000人", 60),
        SIZE_ABOVE_1000("1000人以上", 70);

        public final String name;
        private final Integer item;

    }
}
```
```java



import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import com.fasterxml.classmate.ResolvedType;
import io.swagger.annotations.ApiModelProperty;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.util.ReflectionUtils;
import springfox.bean.validators.configuration.BeanValidatorPluginsConfiguration;
import springfox.documentation.schema.Annotations;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spi.schema.ModelPropertyBuilderPlugin;
import springfox.documentation.spi.schema.contexts.ModelPropertyContext;
import springfox.documentation.swagger.schema.ApiModelProperties;

import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Configuration
@Slf4j
@Import(BeanValidatorPluginsConfiguration.class)
public class SwaggerConfig implements ModelPropertyBuilderPlugin {


    @Override
    public void apply(ModelPropertyContext context) {
        try {
            //为枚举字段设置注释
            Optional<ApiModelProperty> annotation = Optional.empty();
            if (context.getAnnotatedElement().isPresent()) {
                annotation = Optional.ofNullable(annotation.orElseGet(
                        ApiModelProperties.findApiModePropertyAnnotation(context.getAnnotatedElement().get())::get));
            }
            if (context.getBeanPropertyDefinition().isPresent()) {
                annotation = Optional.ofNullable(annotation.orElseGet(Annotations.findPropertyAnnotation(
                        context.getBeanPropertyDefinition().get(),
                        ApiModelProperty.class)::get));
            }

            String enumPackage = (annotation.get()).notes();
            Class rawPrimaryType = (annotation.get()).enumDynamic();
            if (StrUtil.isNotBlank(enumPackage)) {
                try {
                    rawPrimaryType = Class.forName(enumPackage);
                } catch (ClassNotFoundException e) {
                    return;
                }
            }

            if (ObjectUtil.isNull(rawPrimaryType)) {
                return;
            }

            Field[] f = rawPrimaryType.getDeclaredFields();
            List<Field> fieldList = new ArrayList<>();
            for (Field field : f) {
                if (
                        !field.isEnumConstant() &&
                                (field.getType().equals(String.class) || field.getType().equals(Integer.class))
                ) {
                    fieldList.add(field);
                }

            }


            Object[] subItemRecords = null;

            if (Enum.class.isAssignableFrom(rawPrimaryType)) {
                subItemRecords = rawPrimaryType.getEnumConstants();
            }
            if (null == subItemRecords) {
                return;
            }

            StringBuilder joinText = new StringBuilder();
            for (Object subItemRecord : subItemRecords) {
                Field indexField = ReflectionUtils.findField(subItemRecord.getClass(), fieldList.get(0).getName());
                ReflectionUtils.makeAccessible(indexField);
                String name = ReflectionUtils.getField(indexField, subItemRecord).toString();

                indexField = ReflectionUtils.findField(subItemRecord.getClass(), fieldList.get(1).getName());
                ReflectionUtils.makeAccessible(indexField);
                String item = ReflectionUtils.getField(indexField, subItemRecord).toString();
                joinText.append(name).append("=").append(item).append(" ，");
            }
            if ("，".equals(joinText.substring(joinText.length() - 1))) {
                joinText = new StringBuilder(joinText.substring(0, joinText.length() - 1));
            }
            joinText = new StringBuilder(annotation.get().value() + " (" + joinText + "）");

            final Class fieldType = context.getBeanPropertyDefinition().get().getField().getRawType();
            final ResolvedType resolvedType = context.getResolver().resolve(fieldType);
            context.getBuilder().description(joinText.toString()).type(resolvedType);

        } catch (Exception ignored) {
        }


    }


    @Override
    public boolean supports(DocumentationType documentationType) {
        return true;
    }
}
```
这样在后期维护过程中只需要维护枚举类即可，效果和上面的注释一样

```java
@Data
public class CompanyInfoDTO {

    @ApiModelProperty(value = "人员规模", enumDynamic = UserCompany.StaffSize.class)
    private Integer staffSize;

}
```

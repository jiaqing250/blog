mybatisplus未提供自定义的机制，需要我们自己想办法进行转换，目前在当前项目中需要将 mysql(datetime)->java(Optional<Date>) 以下是具体代码
```java
 

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.builder.GeneratorBuilder;
import com.baomidou.mybatisplus.generator.config.converts.MySqlTypeConvert;
import com.baomidou.mybatisplus.generator.config.querys.MySqlQuery;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.IColumnType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.google.common.collect.Lists;

import java.sql.SQLException;
import java.util.*;

public class GeneratorApplication {
    //作者
    private static final String author = "";
    //数据库URL
    private static final String url = "jdbc:p6spy:mysql://xxx:3306/xxx?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&nullCatalogMeansCurrent=true";
    // 数据库用户名
    private static final String username = "";
    //数据库密码
    private static final String passwd = "";
    //指定数据库表的前缀。指定后，在生成文件时，模板会自动截取掉前缀字符，如表名为sys_user，指定前缀为sys_,生成实体是自动识别生成为user
    //多个前缀可以用逗号隔开，例如 sys_,bs_，根据项目需要配置
    private static final String tablePrefix = "xx_";
    //设置生成实体时的公共父类，例如 com.baomidou.global.BaseEntity  ，根据项目需要配置
    private static final String superEntityPackageString = "io.xxx.common.entity.BaseHpEntity";
    //设置生成Controller时的公共父类，例如 com.baomidou.global.BaseController   ，根据项目需要配置
    private static final String superControllerPackageString = "";

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入" + tip + "：");
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    /**
     * 执行 run
     */
    public static void main(String[] args) throws SQLException {

        //数据库配置
        DataSourceConfig dataSourceConfig = new DataSourceConfig
                // 驱动连接的URL、数据库连接用户名、数据库连接密码
                .Builder(url, username, passwd)
                // 类型转换,数据库=》JAVA类型  mysql: MySqlTypeConvert() sqlserver:SqlServerTypeConvert() oracle:OracleTypeConvert()
                .typeConvert(new MyTypeConvert())
                // 关键字处理 ,这里选取了mysql5.7文档中的关键字和保留字（含移除） 说明：官方文档中没有关于sqlserver，oracle数据库的配置
                //.keyWordsHandler(new MySqlKeyWordsHandler())
                // 数据库信息查询类,默认由 dbType 类型决定选择对应数据库内置实现：mysql:MySqlQuery(),sqlserver :SqlServerQuery(),Oracle:OracleQuery()
                .dbQuery(new MySqlQuery())
                // 数据库 schema name
                .schema("mybatis-plus")
                .build();

        //全局策略配置
        GlobalConfig globalConfig = GeneratorBuilder.globalConfigBuilder()
                // 覆盖已生成文件
                .fileOverride()
                // 指定输出目录
                .outputDir(System.getProperty("user.dir") + "/common/src/main/java/io/xx/common")
                //禁止打开生成目录
                .disableOpenDir()
                // 生成swagger注解
                .enableSwagger()
                // 作者名
//                .author(author)
                // 时间策略
                .dateType(DateType.ONLY_DATE)
                // 注释日期格式
                .commentDate("yyyy-MM-dd")
                .build();

        Map<OutputFile, String> pathInfo = new HashMap<>();
 
        pathInfo.put(OutputFile.mapperXml, System.getProperty("user.dir") + "/common/src/main/resources/mapper");
        pathInfo.put(OutputFile.entity, System.getProperty("user.dir") + "/common/src/main/java/io/xx/common/entity");
        pathInfo.put(OutputFile.mapper, System.getProperty("user.dir") + "/common/src/main/java/io/xx/common/mapper");
        //包配置
        PackageConfig packageConfig = new PackageConfig.Builder()
                // 父包名。如果为空，将下面子包名必须写全部， 否则就只需写子包名
                .parent("io.xx.common")
                // 父包模块名
//                .moduleName(scanner("子包名(例如包名为：com.mes.system,子包名为：system)"))
                //其他entity ，service，controller 使用默认值即可，如果需要单独指定，在此处配置就行
                .mapper("mapper")
                .entity("entity")
                .pathInfo(pathInfo)
                .build();

        TemplateConfig templateConfig = new TemplateConfig.Builder()
                .disable(TemplateType.ENTITY)
                .entity("/templates/entity.java")
                .service("")
                .serviceImpl("")
                .mapper("/templates/mapper.java")
                .mapperXml("/templates/mapper.xml")
                .controller("")
                .build();


        //策略配置
        StrategyConfig strategyConfig = new StrategyConfig.Builder()
                //表名匹配，按指定的表名生成对应的文件
                .addInclude(scanner("数据库表名，多个表名用英文逗号分割").split(","))
                //指定数据库表的前缀。指定后，在生成文件时，模板会自动截取掉前缀字符，如表名为sys_user，指定前缀为sys_,生成实体是自动识别生成为user
                .addTablePrefix(tablePrefix)

                /**实体模型配置*/
                .entityBuilder()
                //数据库表映射到实体的命名策略,NamingStrategy.underline_to_camel 指下划线转驼峰，NamingStrategy.no_change 无改变
                .naming(NamingStrategy.underline_to_camel)
                //开启实体Lombok注解模式
                .enableLombok()
                .idType(IdType.ASSIGN_ID)
                //开启 Boolean 类型字段移除 is 前缀
                .enableRemoveIsPrefix()
                //开启生成实体时生成字段注解
                .enableTableFieldAnnotation()
                .addIgnoreColumns("create_data")
                .addIgnoreColumns("update_data")
                //设置Entity父类
                .superClass(superEntityPackageString)
//                /**service接口模型配置*/
//                .serviceBuilder()
//                //格式化 Service文件名称
//                .formatServiceFileName("%sService")
//
//                /**controller模型配置*/
//                .controllerBuilder()
//                //设置Controller父类
//                .superClass(superControllerPackageString)
//                //开启驼峰转连字符
//                .enableHyphenStyle()
//                //开启生成@RestController 控制器
//                .enableRestStyle()

                /**mapper配置*/
                .mapperBuilder()
                //开启 @Mapper 注解
                .enableMapperAnnotation()
                .build();

        // 配置模板
        //TemplateConfig templateConfig = new TemplateConfig.Builder().disable().build();//激活所有默认模板

        //添加以上配置到AutoGenerator中
        AutoGenerator autoGenerator = new AutoGenerator(dataSourceConfig); // 数据源配置
        autoGenerator.global(globalConfig); // 全局策略配置
        autoGenerator.packageInfo(packageConfig);    // 包配置
        autoGenerator.strategy(strategyConfig);

        autoGenerator.template(templateConfig); // 配置模板
        // 生成代码
        autoGenerator.execute();

    }

	//主要看这个类
    public static class MyTypeConvert implements ITypeConvert {

        /**
         * 执行类型转换
         *
         * @param globalConfig 全局配置
         * @param fieldType    字段类型
         * @return ignore
         */
        public IColumnType processTypeConvert( GlobalConfig globalConfig,  String fieldType) {
            List<String> values = Lists.newArrayList("date", "time", "year");
            String dateType = fieldType.replaceAll("\\(\\d+\\)", "");
            for (String value : values) {
                if(dateType.contains(value)){
                    return new IColumnType() {
                        @Override
                        public String getType() {
                            return "Optional<Date>";
                        }

                        @Override
                        public String getPkg() {
                            return "java.util.Optional";
                        }
                    };
                }
            }

            return new MySqlTypeConvert().processTypeConvert(globalConfig,fieldType);
        }
    }


}

```
当然，最后虽然我写出来了，但是没有使用Optional<Date>，而是继续使用Date
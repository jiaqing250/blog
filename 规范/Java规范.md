# 1、java规范问题，之前没想明白的
1. 在实体类中统一使用包装类,在方法中则使用基本类
2. 前后端规范

参考链接:<br />[微服务后端接口开发及返回值规范札记_后端接口返回数据类型规范_zhg_vincent的博客-CSDN博客](https://blog.csdn.net/Vincent2014Linux/article/details/103241610)

3. controller中的方法名应该要和接口地址是一样的
4. 使用mybatis-plus后sql到处飞导致出现了很多新问题，其中一个问题就是无法统一管理，在兼容性面前无法实现性能高的情况。
5. 使用feign且访问方式为非get方式时,参数传递使用map包裹进行传递,没必要为了一个属性创建一个类

[优秀的后端应该有哪些开发习惯？](https://mp.weixin.qq.com/s/gpV75wBsEe78Yab-WVTvuQ)<br />[编码5分钟，命名2小时？史上最全的Java命名规范参考！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486449&idx=1&sn=c3b502529ff991c7180281bcc22877af&chksm=cea2443af9d5cd2c1c87049ed15ccf6f88275419c7dbe542406166a703b27d0f3ecf2af901f8&token=999884676&lang=zh_CN#rd)

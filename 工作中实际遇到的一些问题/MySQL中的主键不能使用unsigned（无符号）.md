
在项目开发过程中，我们从自增长方式转换为使用UUID（纯数字bigint类型）来生成ID。在进行一次旧数据兼容时，我们使用了UUID_SHORT()函数生成ID。在测试环境下生成的ID是18位，但在正式环境下生成的ID是19位。
<br />
由于脚本按照测试环境的配置进行补全，所以在ID的前面添加了一个1。然而，在正式环境运行后，我们没有及时发现这个问题。直到测试功能时才注意到这个问题。因此，在后续的数据表设计中，我们禁止在主键上添加unsigned属性。

<br />
![测试环境下生成的uuid](https://cdn.nlark.com/yuque/0/2023/png/2923644/1689303926383-fab4c540-7399-42b4-926b-6f0ac711854b.png#averageHue=%23fcfcfb&clientId=u83cab37c-80d7-4&from=paste&height=359&id=aLjAj&originHeight=538&originWidth=397&originalType=binary&ratio=1.5&rotation=0&showTitle=true&size=10049&status=done&style=none&taskId=u968e3cd3-deec-4c32-9f8c-7fcad828811&title=%E6%B5%8B%E8%AF%95%E7%8E%AF%E5%A2%83%E4%B8%8B%E7%94%9F%E6%88%90%E7%9A%84uuid&width=264.6666666666667 "测试环境下生成的uuid")

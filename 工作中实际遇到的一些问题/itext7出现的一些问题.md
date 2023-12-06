# 1、Adobe Acrobat Pro DC 文字识别导致出现不同系统显示有bug
经验<br />解决办法：关闭文字识别<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/2923644/1666238158972-a5d5d126-0955-4103-b01d-b363e6224ff0.png#averageHue=%23f6f6f5&clientId=u6c51c84f-11eb-4&from=paste&height=508&id=ua64640de&originHeight=762&originWidth=334&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26041&status=done&style=none&taskId=u8a9f0f7e-e92c-4104-b48a-6ef4a615bdd&title=&width=222.66666666666666)
# 2、iText7使用PdfFont遇特殊字符如‘凉’字报空指针，中文打印乱码问题
场景：名称为  迪丽热巴·迪力木拉提  <br />原因：带有 · 特殊字符<br />报错函数： pdfDoc.close(); 导致空指针异常



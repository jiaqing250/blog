### 1、简历生成的几种方案
 

1. itext 后端手敲代码，一个一个调整
   1. 弊端，调试麻烦
2. 使用itext进行html转pdf

[darren/itextsample](https://gitee.com/ydq/itext-sample)<br />[技术分享 - 智能简历平台 - 掘金](https://juejin.cn/post/7093555559051296798)

### 2、itext7 html转pdf简历
在实际项目中需要把用户数据转成pdf简历，最后选择使用html+itext7转成pdf简历
```java



import cn.hutool.core.util.ArrayUtil;
import cn.hutool.core.util.StrUtil;
import com.itextpdf.html2pdf.ConverterProperties;
import com.itextpdf.html2pdf.HtmlConverter;
import com.itextpdf.html2pdf.attach.impl.OutlineHandler;
import com.itextpdf.kernel.events.PdfDocumentEvent;
import com.itextpdf.kernel.font.PdfFont;
import com.itextpdf.kernel.geom.PageSize;
import com.itextpdf.kernel.pdf.PdfDocument;
import com.itextpdf.kernel.pdf.PdfWriter;
import com.itextpdf.kernel.pdf.WriterProperties;
import com.itextpdf.layout.font.FontProvider;

import java.io.ByteArrayOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;

public class Itext7Generator {

    private static final int BUFFER_SIZE = 4096;
    private static final String TEMPLATE_PATH = "/template/resume.html";//模板路径
    private static final String FONT_PATH = "/fonts/Alibaba-PuHuiTi-Regular.otf"; // 中文字体路径
    private FontProvider fontProvider;

    private volatile static Itext7Generator instance;

    public static void main(String[] args) throws Exception {
        //测试
        String html = html();
        Itext7Generator generator = Itext7Generator.getInstance();
        ByteArrayOutputStream outputStream = generator.generateByteArrayOutputStream(html);
        generator.generateWriterPdf(outputStream, "out.pdf");
    }


    public static Itext7Generator getInstance() {
        if (instance == null) {
            synchronized (Itext7Generator.class) {
                if (instance == null) {
                    instance = new Itext7Generator();
                    try {
                        instance.initFont();
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                }
            }
        }

        return instance;
    }

    private void initFont() throws IOException {
        fontProvider = new FontProvider();
        //设置中文字体文件的路径，可以在classpath目录下
        fontProvider.addFont(FONT_PATH);
    }

    public void generateWriterPdf(ByteArrayOutputStream outputStream, String outputFile) throws IOException {
        outputStream.writeTo(new FileOutputStream(outputFile));
    }

    public String generatePdfToOss(String html) throws Exception {
        ByteArrayOutputStream outputStream = this.generateByteArrayOutputStream(html);
        return this.uploadPdfToOss(outputStream);
    }

    private ByteArrayOutputStream generateByteArrayOutputStream(String html) throws IOException {
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        try (PdfWriter writer = new PdfWriter(outputStream, new WriterProperties().setFullCompressionMode(Boolean.TRUE));
             PdfDocument doc = new PdfDocument(writer)) {
            doc.setDefaultPageSize(PageSize.A4);
            doc.getDefaultPageSize().applyMargins(20, 20, 20, 20, true);
            // 获取字体，提供给水印 和 页码使用
            PdfFont pdfFont = fontProvider.getFontSet()
                    .getFonts()
                    .stream()
                    .findFirst()
                    .map(fontProvider::getPdfFont)
                    .orElse(null);

            doc.addEventHandler(PdfDocumentEvent.END_PAGE, new WaterMarker(pdfFont));
            doc.addEventHandler(PdfDocumentEvent.END_PAGE, new PageMarker(pdfFont));

            ConverterProperties properties = new ConverterProperties();
            properties.setFontProvider(fontProvider);
            //PDF目录
            properties.setOutlineHandler(OutlineHandler.createStandardHandler());
            HtmlConverter.convertToPdf(html, doc, properties);
        }
        return outputStream;
    }

    private String uploadPdfToOss(ByteArrayOutputStream outputStream) throws IOException {
        UserDto userInfo = null;
        UploadFileInputStreamDTO fileInputStreamDTO = new UploadFileInputStreamDTO();
        fileInputStreamDTO.setBytes(ArrayUtil.wrap(outputStream.toByteArray()));
        fileInputStreamDTO.setFileName(StrUtil.format("resume/{}/{}", userInfo.getUserId(), userInfo.getUserName()));
        return SpringContextUtils.getBean(OssFeignService.class).uploadFileInputStream(fileInputStreamDTO).getData().getUrl();
    }

    private static String html() {
        StringBuilder out = new StringBuilder();

        try (InputStream input = Itext7Generator.class.getResourceAsStream(TEMPLATE_PATH)) {
            byte[] b = new byte[BUFFER_SIZE];
            int n;
            while ((n = input.read(b)) != -1) {
                out.append(new String(b, 0, n));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return out.toString();
    }
}
```
```java


import com.itextpdf.kernel.events.Event;
import com.itextpdf.kernel.events.IEventHandler;
import com.itextpdf.kernel.events.PdfDocumentEvent;
import com.itextpdf.kernel.font.PdfFont;
import com.itextpdf.kernel.geom.Rectangle;
import com.itextpdf.kernel.pdf.PdfDocument;
import com.itextpdf.kernel.pdf.PdfPage;
import com.itextpdf.kernel.pdf.canvas.PdfCanvas;
import com.itextpdf.layout.Canvas;
import com.itextpdf.layout.element.Paragraph;
import com.itextpdf.layout.property.TextAlignment;
import com.itextpdf.layout.property.VerticalAlignment;
import lombok.AllArgsConstructor;

@AllArgsConstructor
    public class WaterMarker implements IEventHandler {

        private PdfFont pdfFont;

        @Override
        public void handleEvent(Event event) {

            PdfDocumentEvent docEvent = (PdfDocumentEvent) event;
            PdfDocument pdf = docEvent.getDocument();
            PdfPage page = docEvent.getPage();

            // 判断是否为第一页
            if (pdf.getPageNumber(page) == 1) {
                Rectangle pageSize = page.getPageSize();
                PdfCanvas pdfCanvas = new PdfCanvas(
                    page.getLastContentStream(), page.getResources(), pdf);
                Canvas canvas = new Canvas(pdfCanvas, pdf, pageSize);
                Paragraph waterMarker = new Paragraph("水印水印")
                    .setFont(pdfFont)
                    .setOpacity(.1f)
                    .setFontSize(30);

                canvas.showTextAligned(waterMarker, pageSize.getX(), pageSize.getTop() - 60, pdf.getNumberOfPages(), TextAlignment.LEFT, VerticalAlignment.TOP, .6f);

                canvas.close();
            }
        }
    }
```
```java


import com.itextpdf.kernel.events.Event;
import com.itextpdf.kernel.events.IEventHandler;
import com.itextpdf.kernel.events.PdfDocumentEvent;
import com.itextpdf.kernel.font.PdfFont;
import com.itextpdf.kernel.geom.Rectangle;
import com.itextpdf.kernel.pdf.PdfDocument;
import com.itextpdf.kernel.pdf.PdfPage;
import com.itextpdf.kernel.pdf.canvas.PdfCanvas;
import com.itextpdf.layout.Canvas;
import com.itextpdf.layout.element.Paragraph;
import com.itextpdf.layout.property.TextAlignment;
import lombok.AllArgsConstructor;

@AllArgsConstructor
    public class PageMarker implements IEventHandler {

        private PdfFont pdfFont;

        @Override
        public void handleEvent(Event event) {
            PdfDocumentEvent docEvent = (PdfDocumentEvent) event;
            PdfDocument      pdf      = docEvent.getDocument();
            PdfPage          page     = docEvent.getPage();
            Rectangle        pageSize = page.getPageSize();
            PdfCanvas pdfCanvas = new PdfCanvas(
                page.getLastContentStream(), page.getResources(), pdf);
            Canvas canvas = new Canvas(pdfCanvas, pdf, pageSize);
            float  x      = (pageSize.getLeft() + pageSize.getRight()) / 2;
            float  y      = pageSize.getBottom() + 15;
            Paragraph p = new Paragraph("第" + pdf.getPageNumber(page) + "页")
                .setFontSize(12)
                .setFont(pdfFont);
            canvas.showTextAligned(p, x, y, TextAlignment.CENTER);
            canvas.close();
        }
    }
```

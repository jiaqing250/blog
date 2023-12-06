### COLOR_BGR2HSV 导致图片颜色识别错误 
```javascript

console.time("导入类");
runtime.images.initOpenCvIfNeeded();
importClass(org.opencv.core.MatOfByte);
importClass(org.opencv.core.Scalar);
importClass(org.opencv.core.Point);
importClass(org.opencv.core.CvType);
importClass(java.util.List);
importClass(java.util.ArrayList);
importClass(java.util.LinkedList);
importClass(org.opencv.imgproc.Imgproc);
importClass(org.opencv.imgcodecs.Imgcodecs);
importClass(org.opencv.core.Core);
importClass(org.opencv.core.Mat);
importClass(org.opencv.core.MatOfDMatch);
importClass(org.opencv.core.MatOfInt);
importClass(org.opencv.core.MatOfFloat);
importClass(org.opencv.core.MatOfKeyPoint);
importClass(org.opencv.core.MatOfRect);
importClass(org.opencv.core.Size);
console.timeEnd("导入类");
console.log('Core.VERSION:',Core.VERSION)
getColorBias(images.read('/sdcard/autoJsTools/image.png').getMat())
function getColorBias(image){
     // 转换为HSV色彩空间
        let hsv_image = new Mat();
        Imgproc.cvtColor(image, hsv_image, Imgproc.COLOR_RGB2HSV); //坑 一开始一直是使用BGR2HSV 导致识别错误，在java中是没问题的，但是在autojs中就不行
        console.log(image)
        // 计算色彩直方图
        let hist = new Mat();
        let list= new java.util.ArrayList();
        list.add(hsv_image);
         console.log(list)
        //  Imgproc.calcHist(Collections.singletonList(hsv_image), new MatOfInt(0), new Mat(), hist, new MatOfInt(180), new MatOfFloat(0, 180));
        Imgproc.calcHist(list,new MatOfInt([0]), new Mat(), hist, new MatOfInt([180]),new MatOfFloat([0,180]));
            console.log(hist)
        // // 查找直方图峰值的索引
         
         let result = Core.minMaxLoc(hist);
        let peak_index = result.maxLoc.y;
        // peak_index-=75;
        log(peak_index)
        // // 根据峰值索引判断主要颜色
        let color_bias;
        if (peak_index < 15 || peak_index > 165) {
            color_bias = "red";
        } else if (peak_index < 45) {
            color_bias = "yellow";
        } else if (peak_index < 75) {
            color_bias = "green";
        } else if (peak_index < 105) {
            color_bias = "cyan";
        } else if (peak_index < 135) {
            color_bias = "blue";
        } else if (peak_index < 165) {
            color_bias = "purple";
        } else {
            color_bias = "未知";
        }
        log(color_bias)
        toast(color_bias);
        return color_bias;
}
 
```

# 1、从utbot项目发现的一个生成java文件并使用shell运行编译生成class文件的代码
看到这个功能后就想起了之前在大学的时候，问了老师一个问题，当时在练习 ++i,i++。想用程序生成相关题目，然后生成代码然后直接给出答案，但是这个当时的老师也不会。
```java
package com.alibaba.demo;

import java.io.File;
import java.io.FileWriter;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Arrays;
import java.util.concurrent.TimeUnit;

/**
 * @author 2023-01-17 
 */
public class TestUtil {

    public static void main(String[] args) throws Exception {
        new TestUtil().compileClassFile("Test","public class Test {\n" +
                "    ///region Test suites for executable org.utbot.examples.manual.examples.AssignedArrayExample.foo\n" +
                "    \n" +
                "public static void main(String[] args) throws Exception {"+
                "System.out.printf(\"hello world\");"+
                "}"+
                "    ///region\n" +
                "}");
    }
    public void compileClassFile(String className, String body) throws Exception {
        Path workdir = Files.createTempDirectory(Paths.get("target"), "test");

        File sourceCodeFile = new File(workdir.toAbsolutePath().toFile(), className + ".java");
        FileWriter fileWriter = new FileWriter(sourceCodeFile,true);
        fileWriter.write(body);
        fileWriter.close();
        long timeout = 60L;
        ProcessBuilder processBuilder = new ProcessBuilder()
                .redirectErrorStream(true)
                .directory(workdir.toFile())
                .command(Arrays.asList("javac", "./" + sourceCodeFile.toPath().getFileName()))
                .inheritIO();
        ProcessBuilder processBuilder2 = new ProcessBuilder()
                .redirectErrorStream(true)
                .directory(workdir.toFile())
                .command(Arrays.asList("java",  sourceCodeFile.toPath().getFileName().toString().replace(".java","")))
                .inheritIO();

        Process process = processBuilder.start();

        // ProcessBuilder may hang if output is not read. We can return a Pair instance
        // from the method if we want to handle the output
        /*val processResult = */
        boolean javacFinished = process.waitFor(timeout, TimeUnit.SECONDS);

        if (javacFinished) {
            Process process2 = processBuilder2.start();
            process2.waitFor(timeout, TimeUnit.SECONDS);
            System.out.printf("\nJavac can't complete in %s sec\n",timeout);
        }

    }
}



```
代码来源：[unitbot项目](https://github.com/UnitTestBot/UTBotJava)
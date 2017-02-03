---
layout: post
title:  "Java Log StaticLoggerBinder"
date: 2017-01-04
categories: Java,Log
tags: Java,Middleware,Log
published: true
---
* 目录
{:toc}

### Maven 依赖

```plain
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.12</version>
</dependency>

<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.3</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.3</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.3</version>
</dependency>

<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.2</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

### 绑定结果

```plain
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/Users/daijiajia/.m2/repository/org/slf4j/slf4j-log4j12/1.7.12/slf4j-log4j12-1.7.12.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/Users/daijiajia/.m2/repository/org/apache/logging/log4j/log4j-slf4j-impl/2.3/log4j-slf4j-impl-2.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/Users/daijiajia/.m2/repository/ch/qos/logback/logback-classic/1.1.2/logback-classic-1.1.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
log4j:WARN No appenders could be found for logger (Main).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

### 本地运行classpath

```plain
/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/bin/java -Didea.launcher.port=7537 "-Didea.launcher.bin.path=/Volumes/IntelliJ IDEA CE/IntelliJ IDEA CE.app/Contents/bin" -Dfile.encoding=UTF-8 -classpath "/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/deploy.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/jfxrt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/javaws.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jfxswt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/plugin.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/javafx-mx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/packager.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/lib/tools.jar:/Users/daijiajia/Project/log4j-demo/target/classes:/Users/daijiajia/.m2/repository/log4j/log4j/1.2.17/log4j-1.2.17.jar:/Users/daijiajia/.m2/repository/org/slf4j/slf4j-log4j12/1.7.12/slf4j-log4j12-1.7.12.jar:/Users/daijiajia/.m2/repository/org/slf4j/slf4j-api/1.7.12/slf4j-api-1.7.12.jar:/Users/daijiajia/.m2/repository/org/apache/logging/log4j/log4j-core/2.3/log4j-core-2.3.jar:/Users/daijiajia/.m2/repository/org/apache/logging/log4j/log4j-api/2.3/log4j-api-2.3.jar:/Users/daijiajia/.m2/repository/org/apache/logging/log4j/log4j-slf4j-impl/2.3/log4j-slf4j-impl-2.3.jar:/Users/daijiajia/.m2/repository/ch/qos/logback/logback-core/1.1.2/logback-core-1.1.2.jar:/Users/daijiajia/.m2/repository/ch/qos/logback/logback-classic/1.1.2/logback-classic-1.1.2.jar:/Users/daijiajia/.m2/repository/org/slf4j/jcl-over-slf4j/1.7.10/jcl-over-slf4j-1.7.10.jar:/Volumes/IntelliJ IDEA CE/IntelliJ IDEA CE.app/Contents/lib/idea_rt.jar" com.intellij.rt.execution.application.AppMain Main
```

>在IDEA中运行时其中的classpath定义顺序和Maven中依赖出现的顺序完全一致，可以看看iml文件的内容

### iml文件内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<module org.jetbrains.idea.maven.project.MavenProjectsManager.isMavenModule="true" type="JAVA_MODULE" version="4">
  <component name="NewModuleRootManager" LANGUAGE_LEVEL="JDK_1_5" inherit-compiler-output="false">
    <output url="file://$MODULE_DIR$/target/classes" />
    <output-test url="file://$MODULE_DIR$/target/test-classes" />
    <content url="file://$MODULE_DIR$">
      <sourceFolder url="file://$MODULE_DIR$/src/main/java" isTestSource="false" />
      <sourceFolder url="file://$MODULE_DIR$/src/main/resources" type="java-resource" />
      <sourceFolder url="file://$MODULE_DIR$/src/test/java" isTestSource="true" />
      <excludeFolder url="file://$MODULE_DIR$/target" />
    </content>
    <orderEntry type="inheritedJdk" />
    <orderEntry type="sourceFolder" forTests="false" />
    <orderEntry type="library" name="Maven: log4j:log4j:1.2.17" level="project" />
    <orderEntry type="library" name="Maven: org.slf4j:slf4j-log4j12:1.7.12" level="project" />
    <orderEntry type="library" name="Maven: org.slf4j:slf4j-api:1.7.12" level="project" />
    <orderEntry type="library" name="Maven: org.apache.logging.log4j:log4j-core:2.3" level="project" />
    <orderEntry type="library" name="Maven: org.apache.logging.log4j:log4j-api:2.3" level="project" />
    <orderEntry type="library" name="Maven: org.apache.logging.log4j:log4j-slf4j-impl:2.3" level="project" />
    <orderEntry type="library" name="Maven: ch.qos.logback:logback-core:1.1.2" level="project" />
    <orderEntry type="library" name="Maven: ch.qos.logback:logback-classic:1.1.2" level="project" />
    <orderEntry type="library" scope="TEST" name="Maven: junit:junit:4.12" level="project" />
    <orderEntry type="library" scope="TEST" name="Maven: org.hamcrest:hamcrest-core:1.3" level="project" />
    <orderEntry type="library" name="Maven: org.slf4j:jcl-over-slf4j:1.7.10" level="project" />
  </component>
</module>
```

### jetty服务器lib加载顺序

```java
/**
 *  Look for jars in WEB-INF/lib
 *
 * @param context
 * @return
 * @throws Exception
 */
protected List<Resource> findWebInfLibJars(WebAppContext context)
throws Exception
{
    Resource web_inf = context.getWebInf();
    if (web_inf==null || !web_inf.exists())
        return null;

    List<Resource> jarResources = new ArrayList<Resource>();
    Resource web_inf_lib = web_inf.addPath("/lib");
    if (web_inf_lib.exists() && web_inf_lib.isDirectory())
    {
        String[] files=web_inf_lib.list();
        for (int f=0;files!=null && f<files.length;f++)
        {
            try
            {
                Resource file = web_inf_lib.addPath(files[f]);
                String fnlc = file.getName().toLowerCase(Locale.ENGLISH);
                int dot = fnlc.lastIndexOf('.');
                String extension = (dot < 0 ? null : fnlc.substring(dot));
                if (extension != null && (extension.equals(".jar") || extension.equals(".zip")))
                {
                    jarResources.add(file);
                }
            }
            catch (Exception ex)
            {
                LOG.warn(Log.EXCEPTION,ex);
            }
        }
    }
    return jarResources;
}
```

>此处不保证加载的顺序，其并不一定是lib下出现的顺序

```java
/**
 * Returns an array of strings naming the files and directories in the
 * directory denoted by this abstract pathname.
 *
 * <p> If this abstract pathname does not denote a directory, then this
 * method returns {@code null}.  Otherwise an array of strings is
 * returned, one for each file or directory in the directory.  Names
 * denoting the directory itself and the directory's parent directory are
 * not included in the result.  Each string is a file name rather than a
 * complete path.
 *
 * <p> There is no guarantee that the name strings in the resulting array
 * will appear in any specific order; they are not, in particular,
 * guaranteed to appear in alphabetical order.
 *
 * <p> Note that the {@link java.nio.file.Files} class defines the {@link
 * java.nio.file.Files#newDirectoryStream(Path) newDirectoryStream} method to
 * open a directory and iterate over the names of the files in the directory.
 * This may use less resources when working with very large directories, and
 * may be more responsive when working with remote directories.
 *
 * @return  An array of strings naming the files and directories in the
 *          directory denoted by this abstract pathname.  The array will be
 *          empty if the directory is empty.  Returns {@code null} if
 *          this abstract pathname does not denote a directory, or if an
 *          I/O error occurs.
 *
 * @throws  SecurityException
 *          If a security manager exists and its {@link
 *          SecurityManager#checkRead(String)} method denies read access to
 *          the directory
 */
public String[] list() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkRead(path);
    }
    if (isInvalid()) {
        return null;
    }
    return fs.list(this);
}
```

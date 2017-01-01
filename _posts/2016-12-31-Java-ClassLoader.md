---
layout: post
title:  "Java类加载器"
date:   2017-01-01 13:59:44 +0800
categories: Java
---

**Java类加载器**是Java应用系统在编译之后的将字节码从磁盘加载到计算机系统内存中的方式，主要的类加载器有以下几种

- **启动类加载器** null 用C/C++实现，在JVM中不可见

- **拓展类加载器** sun.misc.Launcher$ExtClassLoader

- **应用类加载器** sun.misc.Launcher$AppClassLoader

---


## 1.Java ClassLoader 介绍

> Java类加载器（英语：Java Classloader）是Java运行时环境（Java Runtime Environment）的一部分，负责动态加载Java类到Java虚拟机的内存空间中。[1]类通常是按需加载，即第一次使用该类时才加载。由于有了类加载器，Java运行时系统不需要知道文件与文件系统。--[维基百科](https://zh.wikipedia.org/wiki/Java%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8)


## 2.类加载方法loadClass
  在JDK中，类加载的过程如下

```plain
    /**
     * Loads the class with the specified <a href="#name">binary name</a>.  The
     * default implementation of this method searches for classes in the
     * following order:
     *
     * <ol>
     *
     *   <li><p> Invoke {@link #findLoadedClass(String)} to check if the class
     *   has already been loaded.  </p></li>
     *
     *   <li><p> Invoke the {@link #loadClass(String) <tt>loadClass</tt>} method
     *   on the parent class loader.  If the parent is <tt>null</tt> the class
     *   loader built-in to the virtual machine is used, instead.  </p></li>
     *
     *   <li><p> Invoke the {@link #findClass(String)} method to find the
     *   class.  </p></li>
     *
     * </ol>
     *
     * <p> If the class was found using the above steps, and the
     * <tt>resolve</tt> flag is true, this method will then invoke the {@link
     * #resolveClass(Class)} method on the resulting <tt>Class</tt> object.
     *
     * <p> Subclasses of <tt>ClassLoader</tt> are encouraged to override {@link
     * #findClass(String)}, rather than this method.  </p>
     *
     * <p> Unless overridden, this method synchronizes on the result of
     * {@link #getClassLoadingLock <tt>getClassLoadingLock</tt>} method
     * during the entire class loading process.
     *
     * @param  name
     *         The <a href="#name">binary name</a> of the class
     *
     * @param  resolve
     *         If <tt>true</tt> then resolve the class
     *
     * @return  The resulting <tt>Class</tt> object
     *
     * @throws  ClassNotFoundException
     *          If the class could not be found
     */
```
```java
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

### 2.1. loadClass流程

```flow
st=>start: loadClass
e=>end
op=>operation: c=findLoadedClass
findBootstrapClassOrNull=>operation: c = findBootstrapClassOrNull(name)
parentload=>operation: c = parent.loadClass(name, false)
findClass=>operation: c = findClass(name)
resolveClass=>operation: resolveClass(c)

cond=>condition: c==null or not?
parentisnull=>condition: parent==null or not?
isresolve=>condition: is resolve or not?
cisnull=>condition: c==null or not?

st->op->cond
cond(yes)->parentisnull
cond(no)->isresolve

parentisnull(yes)->findBootstrapClassOrNull
parentisnull(no)->parentload->cisnull

cisnull(no)->findClass->isresolve

findBootstrapClassOrNull->cisnull

isresolve(yes)->resolveClass->e
isresolve(no)->e
```
> 整个流程就是先去检查该类是否已经加载，如果已经加载，那么看是否需要解析，如果没有加载，那么判断父加载器是否不为空，父加载器先去尝试加载，若父加载器为空，那么使用启动类加载器加载，加载完毕，调用findClass查找该二进制名字所对应的类

## 3. 自定义类加载器MyClassLoader
```java

  @Override
  protected Class<?> findClass(String name) throws ClassNotFoundException{
    byte[] data = this.loadClassData(name);
    return this.defineClass(name,data,0,data.length);
  }

  private byte[] loadClassData(String name) {
    try {
        Files.readAllBytes(Paths.get(path + name + fileType));
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
  }

```

### 3.1. 测试类
```java
  public static void main(String[] args) {
    System.out.println(MyClassLoader.class.getClassLoader());
    System.out.println(MyClassLoader.class.getClassLoader().getParent());
    System.out.println(MyClassLoader.class.getClassLoader().getParent().getParent());

    MyClassLoader loader1 = new MyClassLoader("loader1");
    System.out.println(loader1.getClass().getClassLoader());
    loader1.path = "/Users/daijiajia/exercise/java/class/classloader/lib1";

    MyClassLoader loader2 = new MyClassLoader(loader1, "loader2");
    System.out.println(loader1.getClass().getClassLoader());
    loader2.path = "/Users/daijiajia/exercise/java/class/classloader/lib2";

    MyClassLoader loader3 = new MyClassLoader(null, "loader3");
    System.out.println(loader1.getClass().getClassLoader());
    loader3.path = "/Users/daijiajia/exercise/java/class/classloader";

    test(loader1);
    test(loader2);
    test(loader3);
  }


  public static void test(ClassLoader loader) {

    try {
        Class clazz = loader.loadClass("Sample1");
        Object object = clazz.newInstance();
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
  }
```

项目目录

```plain
.
├── [ 309]  JavaClassLoader.md
├── [2.6K]  MyClassLoader.class
├── [2.3K]  MyClassLoader.java
├── [ 750]  Sample1.class
└── [ 170]  Sample1.java
```

### 3.2. 使用默认系统的三种加载器

```java
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$ExtClassLoader@15db9742
null
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$AppClassLoader@73d16e93
calss Sample1 loade by :sun.misc.Launcher$AppClassLoader@73d16e93
calss Sample1 loade by :sun.misc.Launcher$AppClassLoader@73d16e93
java.nio.file.NoSuchFileException: /Users/daijiajia/exercise/java/class/classloaderSample1.class
	at sun.nio.fs.UnixException.translateToIOException(UnixException.java:86)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:107)
	at sun.nio.fs.UnixFileSystemProvider.newByteChannel(UnixFileSystemProvider.java:214)
	at java.nio.file.Files.newByteChannel(Files.java:361)
	at java.nio.file.Files.newByteChannel(Files.java:407)
	at java.nio.file.Files.readAllBytes(Files.java:3152)
	at MyClassLoader.loadClassData(MyClassLoader.java:43)
	at MyClassLoader.findClass(MyClassLoader.java:37)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at MyClassLoader.test(MyClassLoader.java:88)
	at MyClassLoader.main(MyClassLoader.java:81)
Exception in thread "main" java.lang.NullPointerException
	at MyClassLoader.findClass(MyClassLoader.java:38)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at MyClassLoader.test(MyClassLoader.java:88)
	at MyClassLoader.main(MyClassLoader.java:81)
```
> 可以看到类加载器的双亲委托加载机制，自定义的ClassLoader是在classpath中（当前路径“.”都在classpath中）也是由AppClassLoader进行加载的，和自定义的类一样，注意此处Sample1类的路径在测试代码中path都写错了，但是loder1的父加载器是AppClassLoader，loader2的父加载器是loader1，loader3的父加载器是启动类加载器，而AppClassLoader可以加载Sample1，但启动类加载器则找不到Sample1，所以loader3尝试自己调用findClass方法去加载Sample1，但是类路径错误导致最后抛出异常

### 3.3. 使用自定义MyClassLoader的findClass去加载

测试类

```java
loader3.path = "/Users/daijiajia/exercise/java/class/classloader/";
```

测试结果

```java
calss Sample1 loade by :MyClassLoader [name=loader3]
```

### 3.4. 同名类加载顺序

将Sample1.class打包至sample1.jar

```java
jar cvf sample1.jar Sample1.class
已添加清单
正在添加: Sample1.class(输入 = 750) (输出 = 439)(压缩了 41%)
```

测试类Sample1增加方法a

```java
public class Sample1 {

  public Sample1() {
    System.out.println("calss " + this.getClass().getSimpleName() + " loade by :" + this.getClass().getClassLoader());
  }

  public void a() {
    System.out.println("method a invoked by Sample1");
  }
}
```

编译之后将class打包至sample2.jar

当前路径

```plain
-rw-r--r--@ 1 daijiajia  staff   309B 12 26 15:56 JavaClassLoader.md
-rw-r--r--  1 daijiajia  staff   2.6K 12 26 17:52 MyClassLoader.class
-rw-r--r--@ 1 daijiajia  staff   2.4K 12 26 17:52 MyClassLoader.java
-rw-r--r--@ 1 daijiajia  staff   250B 12 26 17:48 Sample1.java
-rw-r--r--@ 1 daijiajia  staff     0B 12 26 16:51 Simple.md
-rw-r--r--  1 daijiajia  staff   940B 12 26 17:53 sampl2.jar
-rw-r--r--  1 daijiajia  staff   898B 12 26 17:46 sample1.jar
-rw-r--r--  1 daijiajia  staff   940B 12 26 17:54 sample2.jar
```

测试代码调研Sample1的a方法

```java
java -classpath sample1.jar:sample2.jar MyClassLoader
错误: 找不到或无法加载主类 MyClassLoader
```
> classpath默认当前路径，而此处当前路径被-cp参数覆盖，所以找不到MyClassLoader

cp参数加上当前路径"."

```java
java -classpath .:sample1.jar:sample2.jar MyClassLoader
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$ExtClassLoader@15db9742
null
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$AppClassLoader@73d16e93
calss Sample1 loade by :sun.misc.Launcher$AppClassLoader@73d16e93
Exception in thread "main" java.lang.NoSuchMethodError: Sample1.a()V
	at MyClassLoader.test(MyClassLoader.java:90)
	at MyClassLoader.main(MyClassLoader.java:79)
```
> Sample1在sample1.jar中的定义没有a方法，此时加载顺序先加载sampl1.jar中的Sample1类，sample2.jar中的同名类已经加载过便不再加载

交换java依赖库路径

```java
java -classpath .:sample2.jar:sample1.jar MyClassLoader
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$ExtClassLoader@15db9742
null
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$AppClassLoader@73d16e93
calss Sample1 loade by :sun.misc.Launcher$AppClassLoader@73d16e93
method a invoked by Sample1
```

> 此时得到正确的结果

### 参考
- [深入Java类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/)

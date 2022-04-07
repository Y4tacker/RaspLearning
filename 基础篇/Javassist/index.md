# Javassist

## 简介

`Javassist`是一个开源的分析、编辑和创建Java字节码的类库，相比于ASM上手更容易，它让我们不需要知道字节码是啥



## 基本用法

官方文档：http://www.javassist.org/html/

### 一些简单解释



| ClassPool     | ClassPool是一个基于Hashtable实现的CtClass的容器              |
| ------------- | ------------------------------------------------------------ |
| CtClass       | CtClass表示的是从ClassPool获取的类对象，可对该类就行读写编辑等操作 |
| CtMethod      | 可读写的类方法对象                                           |
| CtConstructor | 可读写的类构造方法对象                                       |
| CtField       | 可读写的类成员变量对象                                       |

内置标志符

| 表达式            | 描述                                      |
| ----------------- | ----------------------------------------- |
| `$0, $1, $2, ...` | `this`和方法参数                          |
| `$args`           | `Object[]`类型的参数数组                  |
| `$$`              | 所有的参数，如`m($$)`等价于`m($1,$2,...)` |
| `$cflow(...)`     | cflow变量                                 |
| `$r`              | 返回类型，用于类型转换                    |
| `$w`              | 包装类型，用于类型转换                    |
| `$_`              | 方法返回值                                |
| `$sig`            | 方法签名，返回`java.lang.Class[]`数组类型 |
| `$type`           | 返回值类型，`java.lang.Class`类型         |
| `$class`          | 当前类，`java.lang.Class`类型             |

### ClassPool对象

#### 创建

关于创建一般有下面两种，不过效果一致

```java
ClassPool classPool = new ClassPool(true);//true表示使用系统默认类路径
ClassPool aDefault = ClassPool.getDefault();//与上面效果一致
```

当然养成一个好的习惯，为了减少ClassPool可能导致的内存损耗，即使调用detach或者再new一个



#### classpath

一般默认的ClassPool.getDefault()的classpath是jvm的classpath，比如在使用tomcat容器时，可能会有多个类加载器作为系统类加载器，可能导致无法正确加载到用户的类，这时需要添加额外的classpath，看代码即可

```java
ClassPool pool = new ClassPool(true);
//将classpath添加到指定classpath之前
pool.insertClassPath(new ClassClassPath(Test.class));
//将classpath添加到指定classpath之后
pool.appendClassPath(new ClassClassPath(Test.class));
pool.insertClassPath("/tmp");
```

## CtClass对象

### 获取CtClass

```java
ClassPool pool = new ClassPool(true);
//通过类名获取
pool.get("java.lang.String");
pool.getOrNull("java.lang.String");
```

### 创建CtClass

```java
ClassPool pool = new ClassPool(true);
//复制一个类
pool.getAndRename("Test","Testt");
pool.get("Testt");
//创建一个新类
pool.makeClass("Y4tacker");
//通过文件
pool.makeClass(new FileInputStream(new File("/Users/y4tacker/Desktop/JavaStudy/ShiroMemShell-master/target/classes/Test.class")));
```

### CtClass基础用法

```java
ClassPool pool = new ClassPool(true);
CtClass ctClass = pool.makeClass(new FileInputStream(new File("/Users/y4tacker/Desktop/JavaStudy/ShiroMemShell-master/target/classes/Test.class")));
//类名
String simpleName = ctClass.getSimpleName();
System.out.println(simpleName);
//全类名
String name = ctClass.getName();
System.out.println(name);
CtClass[] interfaces = ctClass.getInterfaces();
CtClass superclass = ctClass.getSuperclass();
ctClass.getDeclaredMethod("bb",new CtClass[]{pool.get(String.class.getName())});
CtField b = ctClass.getField("b");
boolean array = ctClass.isArray();
ctClass.isAnnotation();ctClass.isEnum();ctClass.isFrozen();ctClass.isInterface();ctClass.isKotlin();ctClass.isModified();ctClass.isPrimitive();
//冻结一个类，使其不能修改
ctClass.freeze();
ctClass.defrost();
//删除类不必要属性，可能导致一些方法不能使用慎用
ctClass.prune();
//将该class从ClassPool中删除；
ctClass.detach();
```

### CtClass一些重要方法

看名字即可知道意思

```java
ClassPool pool = new ClassPool(true);
CtClass y4tacker = pool.makeClass("Y4tacker");
y4tacker.addInterface();
y4tacker.addMethod();
y4tacker.addConstructor();
y4tacker.addField();
```

### CtClass类编译

```java
ClassPool pool = new ClassPool(true);
CtClass y4tacker = pool.makeClass("Y4tacker");
// 将修改后的CtClass加载至当前线程的上下文类加载器中，调用该方法后无法继续修改已加载的class
y4tacker.toClass();
//获取类的字节码文件
y4tacker.getClassFile();
//获取字节码
y4tacker.toBytecode();
```

## CtMethod对象

好吧发现上面一堆东西没用，现在开始只记一些重要的方法了

一些重要方法

```java
ClassPool pool = new ClassPool(true);
CtClass ctClass = pool.get("Test");
// 获取hello方法
CtMethod helloMethod = ctClass.getDeclaredMethod("hello", new CtClass[]{pool.get("java.lang.String")});
// 方法最前面插入
helloMethod.insertBefore("System.out.println($1);");
// 方法最后面插入
helloMethod.insertAfter("System.out.println($1);");
//指定行号前插入
helloMethod.insertAt(1,"System.out.println($1);");
//将方法的内容设置为要写入的代码，当方法被 abstract修饰时，该修饰符被移除；
helloMethod.setBody("");
//创建方法
CtMethod ctMethod = new CtMethod(CtClass.voidType, "printName", new CtClass[]{}, cc);
ctMethod.setModifiers(Modifier.PUBLIC);
ctMethod.setBody("{System.out.println(name);}");
cc.addMethod(ctMethod);
```

## CtConstructor对象

```java
//添加无参的构造函数
CtConstructor cons = new CtConstructor(new CtClass[]{}, cc);
cons.setBody("{name = \"xiaohong\";}");
cc.addConstructor(cons);

//添加有参的构造函数
cons = new CtConstructor(new CtClass[]{pool.get("java.lang.String")}, cc);
cons.setBody("{$0.name = $1;}");
cc.addConstructor(cons);
```

## 一些代码片段方便快速回忆

```java
ClassPool pool = ClassPool.getDefault();

//创建一个空类
CtClass cc = pool.makeClass("Y4tacker");

//新增一个字段 private String name;
CtField param = new CtField(pool.get("java.lang.String"), "name", cc);
// 字段设置为private
param.setModifiers(Modifier.PRIVATE);
// 初始值是 "test"
cc.addField(param, CtField.Initializer.constant("test"));

//生成 getter、setter 方法
cc.addMethod(CtNewMethod.setter("setName", param));
cc.addMethod(CtNewMethod.getter("getName", param));

//添加无参的构造函数
CtConstructor cons = new CtConstructor(new CtClass[]{}, cc);
cons.setBody("{name = \"abc\";}");
cc.addConstructor(cons);

//添加有参的构造函数
cons = new CtConstructor(new CtClass[]{pool.get("java.lang.String")}, cc);
cons.setBody("{$0.name = $1;}");
cc.addConstructor(cons);

//创建一个名为printName方法，无参数，无返回值，输出name值
CtMethod ctMethod = new CtMethod(CtClass.voidType, "printName", new CtClass[]{}, cc);
ctMethod.setModifiers(Modifier.PUBLIC);
ctMethod.setBody("{System.out.println(name);}");
cc.addMethod(ctMethod);

cc.writeFile("./");
```

## 参考文章

https://zhishihezi.net/b/5d644b6f81cbc9e40460fe7eea3c7925

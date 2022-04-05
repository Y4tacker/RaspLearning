# Java Instrument

## 使用JavaAgent修改字节码的原理

​	很简单，一句话说来有点像中间人拦截的过程，使用Javaagent以后，我们加载的class都会被拦截，之后我们就可以在拦截的过程中对字节码进行修改



其他一些比较基础的可以看看我之前写的项目

[Java Instrument插桩技术](https://github.com/Y4tacker/JavaSec/blob/main/6.JavaAgent/JavaInstrument%E6%8F%92%E6%A1%A9%E6%8A%80%E6%9C%AF/JavaInstrument%E6%8F%92%E6%A1%A9%E6%8A%80%E6%9C%AF.md)

关于一些简单的用法体验

[PreMain之addTransformer与redefineClasses用法学习](https://github.com/Y4tacker/JavaSec/blob/main/6.JavaAgent/PreMain%E4%B9%8BaddTransformer%E4%B8%8EredefineClasses%E7%94%A8%E6%B3%95%E5%AD%A6%E4%B9%A0/PreMain%E4%B9%8BaddTransformer%E4%B8%8EredefineClasses%E7%94%A8%E6%B3%95%E5%AD%A6%E4%B9%A0.md)

[AgentMain(JVM启动后动态Instrument)](https://github.com/Y4tacker/JavaSec/blob/main/6.JavaAgent/AgentMain/AgentMain.md)

同时列一个这个方便我快速回忆

```java
public interface Instrumentation {

    // 增加一个 Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);

//retransformClasses 方法能对已加载的 class 进行重新定义，也就是说如果我们的目标类已经被加载的话，我们可以调用该函数，来重新触发这个Transformer的拦截，以此达到对已加载的类进行字节码修改的效果
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;

    // 判断目标类是否能够修改。
    boolean isModifiableClass(Class<?> theClass);

    // 获取目标已经加载的类。
    @SuppressWarnings("rawtypes")
    Class[] getAllLoadedClasses();

    ......
}
```


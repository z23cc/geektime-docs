你好，我是徐昊。今天我们将继续使用TDD的方式来实现注入依赖容器。

## 回顾代码与任务列表

到目前为止，我们的代码是这样的：

```
package geektime.tdd.di;
  
import jakarta.inject.Provider;
import java.util.HashMap;
import java.util.Map;
    
public class Context {
    private Map<Class<?>, Provider<?>> providers = new HashMap<>();
    
    public <ComponentType> void bind(Class<ComponentType> type, ComponentType instance) {
        providers.put(type, (Provider<ComponentType>) () -> instance);
    }
    
    public <ComponentType, ComponentImplementation extends ComponentType>
    void bind(Class<ComponentType> type, Class<ComponentImplementation> implementation) {
        providers.put(type, (Provider<ComponentType>) () -> {
            try {
                return (ComponentType) implementation.getConstructor().newInstance();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        });
    }
    
    public <ComponentType> ComponentType get(Class<ComponentType> type) {
        return (ComponentType) providers.get(type).get();
    }
}
```

任务列表状态为：

- 无需构造的组件——组件实例
- 如果注册的组件不可实例化，则抛出异常
  
  - 抽象类
  - 接口
- 构造函数注入
  
  - 无依赖的组件应该通过默认构造函数生成组件实例
  - 有依赖的组件，通过Inject标注的构造函数生成组件实例
  - 如果所依赖的组件也存在依赖，那么需要对所依赖的组件也完成依赖注入
  - 如果组件有多于一个Inject标注的构造函数，则抛出异常
  - 如果组件需要的依赖不存在，则抛出异常
  - 如果组件间存在循环依赖，则抛出异常
- 字段注入
  
  - 通过Inject标注将字段声明为依赖组件
  - 如果组件需要的依赖不存在，则抛出异常
  - 如果字段为final则抛出异常
  - 如果组件间存在循环依赖，则抛出异常
- 方法注入
  
  - 通过Inject标注的方法，其参数为依赖组件
  - 通过Inject标注的无参数方法，会被调用
  - 按照子类中的规则，覆盖父类中的Inject方法
  - 如果组件需要的依赖不存在，则抛出异常
  - 如果方法定义类型参数，则抛出异常
  - 如果组件间存在循环依赖，则抛出异常
- 对Provider类型的依赖
  
  - 注入构造函数中可以声明对于Provider的依赖
  - 注入字段中可以声明对于Provider的依赖
  - 注入方法中可声明对于Provider的依赖
- 自定义Qualifier的依赖
  
  - 注册组件时，可额外指定Qualifier
  - 注册组件时，可从类对象上提取Qualifier
  - 寻找依赖时，需同时满足类型与自定义Qualifier标注
  - 支持默认Qualifier——Named
- Singleton生命周期
  
  - 注册组件时，可额外指定是否为Singleton
  - 注册组件时，可从类对象上提取Singleton标注
  - 对于包含Singleton标注的组件，在容器范围内提供唯一实例
  - 容器组件默认不是Single生命周期
- 自定义Scope标注
  
  - 可向容器注册自定义Scope标注的回调

## 视频演示

让我们进入今天的部分：

## **思考题**

下一步将要如何重构已有代码？

欢迎把你的思考和想法分享在留言区，也欢迎你扫描详情页的二维码加入读者交流群。我们下节课再见！
<div><strong>精选留言（2）</strong></div><ul>
<li><span>JC</span> 👍（5） 💬（0）<p>第二段视频最后，是不是用 Stream.noneMatch 语义上更好些</p>2022-05-01</li><br/><li><span>肖韬</span> 👍（0） 💬（1）<p>老师，第一段视频中，按照您的 test case，我觉得应该报错。我自己写了相同的代码，确实报错了，但是您的测试用例却没报错，对此百思不得其解。

在 should_bind_type_to_a_class_with_inject_constructor 中，测试用例是这样的：

      Dependency dependency = new Dependency() {};
      context.bind(Component.class, ComponentWithInjectConstructor.class);
      context.bind(Dependency.class, dependency);

在 Context#bind(Component.class, ComponentWithInjectConstructor.class) 的实现代码中，会根据依赖类型 Dependency.class，要求context中返回它的一个实例。


但是很明显，在测试用例调用 context.bind(Component.class, ComponentWithInjectConstructor.class) 时，dependency 实例此时还不在容器中。所以这里应该报错，但是视频中却没有报错。请老师解答。</p>2023-02-12</li><br/>
</ul>
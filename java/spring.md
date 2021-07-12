---
sort: 1
---

# Spring

## Spring启动上下文核心方法
	AbstractApplicationContext.refresh()

## Spring Bean的基本属性详解
1. id: Bean的唯一标识名。
2. name: 用来为id创建一个或多个别名。多个别名之间用逗号或空格分开
3. class：用来定义类的全限定名。只有子类Bean不用定义该属性
4. parent: 子类Bean定义它所引用它的父类Bean。这时前面的class属性失效。子类Bean会继承父类Bean的所有属性，子类Bean也可以覆盖父类Bean的属性。注意：子类Bean和父类Bean是用一个java类
5. abstracct(默认为false)：用来定义Bean是否为抽象Bean. 它表示这个Bean将不会被实例化，一般用于父类Bean，因为父类Bean主要是供子类Bean继承使用
6. lazy-init(默认为default)：用来定义这个Bean是否实现懒初始化。如果为true，它将在BeanFactory启动时初始化所有的SingletonBean。反之，只在Bean请求时才开始创建SingletonBean
7. autowire(自动装配，默认为default): 它定义了Bean的自动装载方式
	1、"no" ： 不使用自动装配功能
	2、"byName"：通过Bean的属性名实现自动装配
	3、"byType"：通过Bean的类型实现自动装配
	4、"constructor": 类似于byType，但它是用于构造函数的参数的额自动组装
	5、"autodetect": 通过Bean类的反省机制(introspection)决定是使用"constructor"还是使用"byType"
8. depends-on(依赖对象): 这个Bean在初始化时依赖的对象，这个对象会在这个Bean初始化之前创建
9. init-method：用来定义Bean的初始化方法，它会在Bean组装之后调用。它必须是一个无参数的方法
10. destroy-method:用来定义Bean的销毁方法，它在BeanFactory关闭时调用。它必须是一个无参的方法。只能应用于singletonBean
11. factory-method：定义创建该Bean对象的工厂方法。用于下面的"factory-bean"，表示这个Bean是通过工厂方法创建。此时,class属性失效
12. factor-bean: 定义创建该Bean对象的工厂类。如果使用了factory-bean 则class属性失效
13. MutablePropertyValues：用于封装
14. ConstructorArgumentValues：用于封装<constructor-arg>标签的 信息，其实类里面就是有一个map，map中用构造函数的参数顺序作为Key,值作为value存储到map中
15. MethodOverrides:用于封装lookup-method和replaced-method标签的信息，同样的类里面有一个set对象添加LookupOverride对象和ReplaceOverride对象
	

## Spring解析xml文件
1. 通过XmlBeanDefinitionReader.doLoadBeanDefinitions()利用jdk sax解析xml里面的每个元素封装到BeanDefinition对象
详细解析bean标签-----> BeanDefinitionParserDelegate.parseBeanDefinitionElement()
2. <context>标签 ---> 对应spring.handlers文件中ContextNamespaceHandler
3. 自定义标签的解析逻辑：根据当前解析标签的头信息找到对应的namespaceUri，加载spring所有Jar包的spring.handlers文件并建立映射关系，根据namespaceUri从映射关系中找到对应的实现了NamespaceHandler接口的类，调用类的Init方法（注册了各种自定义标签的解析类），根据namespaceUri找到对应的解析类，然后调用Paser方法完成标签解析
4. 自定义标签的扫描过程：doScanner方法  去扫描基本包的路径下面找class文件，递归找class文件，判断class文件里面是否有includeFilter里面的注解(@Component),封装成BeanDefinition对象

AbstractApplicationContext.invokeBeanFactoryPostProcessors方法调用下面的接口
BeanDefinitionRegistryPostProcessor  在bean的实例化之前调用，可以完成BeanDefinition的操作
1. postProcessBeanDefinitionRegistry 获取注册器参数
2. postProcessBeanFactory 获取容器工厂参数


自定义扫描类 实现ClassPathBeanDefinitionScanner接口

BeanPostProcessors的接口实现类在ComponentScanBeanDefinitionParser.registerComponents（注册组件）
AbstractApplicationContext.registerBeanPostprocessors
然后在PostProcessorRegistrationDelegate.registerBeanPostprocessors拿到所有实现了BeanPostProcessors接口的类

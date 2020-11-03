# refresh()方法

## 1.prepareRefresh()方法

- 1.1 initPropertySources(),初始化一些属性设置，子类自定义个性化的属性设置方法。
- 1.2 getEnvironment().validateRequiredProperties(),检验属性的合法等。
- 1.3 earlyApplicationEvents = new LinkedHashSet<ApplicationEvent>()保存容器中的一些早期事件。

## 2.obtainFreshBeanFactory(),获取BeanFactory
- 2.1 refreshBeanFactory(),刷新BeanFactory创建了一个this.beanFactory = new DefaultListableBeanFactory();
      设置Id，

```java
@Override
	protected final void refreshBeanFactory() throws BeansException {
        //如果存在就先销毁BeanFactory（），
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
            //定制beanFactory,设置相关属性，是否允许覆盖同名称不同定义的对象以及循环依赖等
			customizeBeanFactory(beanFactory);
            //加载bean的定义
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```

- 2.2 getBeanFactory(),返回刚才GenericApplicationContext创建的BeanFactory对象

## 3.prepareBeanFactory(beanFactory),BeanFactory的预准备工作

- 3.1 设置BeanFactory的类加载器，表达式解析器
- 3.2 添加部分BeanPostProcessor[ApplicationContextAwareProcessor]
- 3.3 设置自动忽略的接口
- 3.4 注册可以解析的自动装配，可以在任何组件中自动注入：BeanFactory,ResourceLoader,ApplicationEventPublisher, ApplicationContext
- 3.5 添加BeanPostProcessor
- 3.6 添加对AspectJ的支持
- 3.7 给BeanFactory中注册一些能用的组件

## 4.postProcessBeanFactory(beanFactory) BeanFactory准备工作完成后进行的后置处理工作

- 4.1 子类通过重写这个方法来在BeanFactory创建并预备完成以后做进一步的设置

## 5.invokeBeanFactoryPostProcessors(beanFactory),执行BeanFactoryPostProcessor

BeanFactoryPostProcessor,BeanFactory的后置处理器，在BeanFactory标准初始化执行

- 5.1 执行invokeBeanFactoryPostProcessors方法

​              **先处理BeanDefinitionRegistryPostProcessor**
- 

   - 5.1.1 获取所有的BeanDefinitionRegistryPostProcessor
   
   - 5.1.2 先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor
   
   - 5.1.3 再执行了Ordered接口的BeanDefinitionRegistryPostProcessor
   
  - 5.1.4 最后执行没有实现任何优先级或者顺序接口的BeanDefinitionRegistryPostProcesso
**在处理BeanFactoryPostProcessor**   
  - 5.1.5 获取所有的BeanFactoryPostProcessor
  
  - 5.1.6 先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor
  
   - 5.1.7 再执行了Ordered接口的BeanFactoryPostProcessor
  
  -   5.1.8 最后执行没有实现任何优先级或者顺序接口的BeanFactoryPostProcessor

## 6.registerBeanPostProcessors(beanFactory),注册BeanPostProcessor

BeanPostProcessor

DestructionAwareBeanPostProcessor

InstantiationAwareBeanPostProcessor

SmartInstantiationAwareBeanPostProcessor

MergedBeanDefinitionPostProcessor

- 6.1 获取所有的BeanPostProcessor,后置处理器默认可以通过PriorityOrdered,Ordered接口来执行优先级
- 6.2 先注册PriorityOrdered优先级接口的BeanPostProcessor,添加到BeanFactory中，BeanFactory.addBeanPostProcessor(postProcessor);
- 6.3 后注册Ordered优先级接口的BeanPostProcessor,添加到BeanFactory中
- 6.4 最后注册没有任何优先级接口的BeanPostProcessor,添加到BeanFactory中
- 6.5 最终注册MergedBeanDefinitionPostProcessor
- 6.6 注册一个ApplicationListenerDetector,来在Bean创建完成后检查是否是ApplicationListener


## 7.initMessageSource(),初始化MessageSource组件（做国际化功能，消息绑定，消息解析）

- 7.1 获取BeanFactory
- 7.2 查看是否有id为messageSource的组件，如果有赋值给messageSource属性，没有自己创建一个
- 7.3 把创建好的MessageSource注册在容器中，获取国际化配置文件的时候，可以自动注册MessageSource

## 8.initApplicationEventMulticaster()初始化事件派发器

- 8.1 获取BeanFactory
- 8.2 从BeanFactory中提取applicationEventMulticaster的ApplicationEventMulticaster
- 8.3 如果上一步没有配置，创建一个SimpleApplicationEventMulticaster
- 8.4 将创建的组件添加到BeanFactory中，以后其他组件可以直接自动注入

## 9.onRefresh()方法，留给子类重写在容器刷新的时候可以自定义逻辑

## 10.registerListeners(),给容器中的所有ApplicationListener注册进来

- 10.1 从容器中拿到所有的ApplicationListener
- 10.2 将每个监听器都添加到事件派发器中
- 10.3 派发之前步骤产生的事件

## 11.finishBeanFactoryInitialization(beanFactory)初始化所有剩下的单实例bean

- 11.1 beanFactory.preInstantiateSingletons();

  - 11.1.1 获取容器中的所有Bean，依次进行初始化和创建对象

  - 11.1.2 获取Bean的定义信息，RootBeanDefinition

  - 11.1.3 Bean不是抽象的，是单实例的，不是懒加载的

  - ​             判断是不是FactoryBean，是否是实现FactoryBean接口的Bean

  - ​             如果不是，调用getBean创建对象

  - ​             getBean()

  - ```
          1.doGetBean(name,null,null,false)
          2.先获取缓存中保存的单实例Bean，如果能获得到说明这个Bean之前被创建过，从下面的地方查找
             private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
          3.缓存中获取不到，开始Bean的创建对象流程
          4.标记当前bean已经被创建
          5.获取Bean的定义信息
          6.获取当前Bean的依赖其他的bean,如果有按照getBean的方式，把依赖的Bean先创建出来
          7.启动单实例bean的创建流程
              createBean(beanName,mbd,args)
              Object bean = resolveBeforeInstantiation(beanName,mbdToUse)，让BeanPostProcess拦截返回代理对象
                     InstantiationAwareBeanPostProcessor提前执行，
                     先触发postProcessBeforeInstantiation()方法
                     如果有返回值，触发postProcessAfterInitialization()方法
             如果前面的 InstantiationAwareBeanPostProcessor没有返回代理对象
             调用Object beanInstance = doCreateBean(beanName,mbdToUse,args);
                 创建Bean实例，createBeanInstance(beanName,mbd,args)
      		         利用工厂方法或者对象的构造器创建出Bean的实例
                 applyMergedBeanDefinitionPostProcessors(mbd,beanType,beanName)
                 populateBean(beanName,mbd,instanceWrapper)为Bean属性赋值
                        InstantiationAwareBeanPostProcessor后置处理器
                             postProcessAfterInstantiation()
                        InstantiationAwareBeanPostProcessor后置处理器
                             postProcessPropertyValues()
                        为属性赋值，利用setter方法
                             applyPropertyValues(beanName, mbd, bw, pvs)
                 initializeBean(beanName,exposedObject,mbd)
      					invokeAwareMethods，调用XXXAware接口
                             BeanNameAware，BeanClassLoaderAware，BeanFactoryAware
                        applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
                             BeanPostProcessor.postProcessBeforeInitialization方法
                        invokeInitMethods(beanName, wrappedBean, mbd);执行初始化方法
                             是否是InitializingBean接口的方法，执行接口规定的初始化
                             如果不是，是否自定义初始化方法
                        applyBeanPostProcessorsAfterInitialization初始化之后的方法
                             BeanPostProcessor.postProcessAfterInitialization方法
                 registerDisposableBeanIfNecessary中注册Bean的销毁方法
               将创建的Bean添加到缓存中singletonObjects  
     所有的Bean都创建完成之后，是否是SmartInitializingSingleton接口如果是就执行afterSingletonsInstantiated方法
    ```

    ​           

## 12.finishRefresh()完成BeanFactory的初始化创建工作，IOC容器创建完成

- 12.1 initLifecycleProcessor()初始化和生命周期有关的后置处理器，查看是否有lifecycleProcessor组件，LifecycleProcessor,如果没有创建默认的
- 12.2 getLifeCycleProcessor().onRefresh()方法
- 12.3 发布容器完成事件
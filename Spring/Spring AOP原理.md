# AOP原理

AOP原理【看给容器中注册了什么组件，该组件有什么功能，什么时候工作】

想看那个方法直接在那个方法处打断点，这样可以看见调用栈，在invokeBeanFactoryPostProcessors中处理

   @Import(AspectJAutoProxyRegistrar.class)
       利用AspectJAutoProxyRegistrar自定义给容器中注册bean
       internalAutoProxyCreator = AnnotationAwareAspectJAutoProxyCreator

```
registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
	public static final String AUTO_PROXY_CREATOR_BEAN_NAME =
			"org.springframework.aop.config.internalAutoProxyCreator
```

   给容器中注册一个AnnotationAwareAspectJAutoProxyCreator

   AnnotationAwareAspectJAutoProxyCreator
       ->AspectJAwareAdvisorAutoProxyCreator
           ->AbstractAdvisorAutoProxyCreator
               ->AbstractAdvisorAutoProxyCreator
                   ->AbstractAutoProxyCreator
                       ->implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware

   AbstractAutoProxyCreator.setBeanFactory()
   AbstractAutoProxyCreator有后置处理器的逻辑

   AbstractAdvisorAutoProxyCreator.setBeanFactory()->initBeanFactory()

   AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()

## 创建和注册AnnotationAwareAspectJAutoProxyCreator

1.传入配置类，创建ioc容器
2.注册配置类，调用refresh()刷新容器
3.registerBeanPostProcessors(beanFactory);注册bean的后置处理器方便拦截bean的创建
    3.1 先获取ioc容器已经定义的需要创建对象的所有BeanPostProcessor
    3.2 给容器加别的BeanPostProcessor
    3.3 优先注册实现了PriorityOrdered接口的BeanPostProcessor
    3.4 在给容器注册实现了Ordered接口的BeanPostProcessor
    3.5 没有实现注册借口的BeanPostProcessor
    3.6 注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中
        创建internalAutoProxyCreator的BeanPostProcessor[AnnotationAwareAspectJAutoProxyCreator]
        1>创建Bean的实例
        2>populateBean属性赋值
        3>initializeBean初始化Bean
            invokeAwareMethods()处理Aware接口的调用
            applyBeanPostProcessorsBeforeInitialization(),应用后置处理器的BeforeInitialization方法
            invokeInitMethods方法
            applyBeanPostProcessorsAfterInitialization()
        4>**BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功**
    3.7 把BeanPostProcessor注册到BeanFactory中
        beanFactory.addBeanPostProcessor(postProcessor)

## AnnotationAwareAspectJAutoProxyCreator执行时机

4.finishBeanFactoryInitialization(beanFactory)完成BeanFactory初始化工作，创建剩下的单实例Bean
    4.1 遍历获取容器中的所有Bean，依次创建对象getBean(beanName)
           getBean()->doGetBean()->getSingleton()

​    4.2  创建bean

​		AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前进行拦截，它是一个InstantiationAwareBeanPostProcessor

​    	4.2.1 先从缓存中获取当前bean，如果能获取到，说明bean之前被创建过，直接使用，否则在创建，只要创建好bean就会被缓存起来。
​        4.2.2 createBean()
​            	resolveBeforeInstantiation,希望后置处理器能返回一个代理对象，如果能返回代理对象就使用，如果不能就继续

​						后置处理器尝试返回对象

​								bean = applyBeanPostProcessorsBeforeInstantiation()

​                                		拿到所有的后置处理器，如果是InstantiationAwareBeanPostProcessor就执行                              							 								BeanPostProcessorsBeforeInstantiation

​                                if(bean!=null){

​                                          bean =   applyBeanPostProcessorsAfterInitialization() 

​                doCreateBean()真正去创建一个bean,和3.6流程一样

## 创建AOP的代理过程

1 每一个bean创建之前，调用postProcessBeforeInstantiation()
    关心MathCalculator和LogAspect的创建

```java
	if (this.advisedBeans.containsKey(cacheKey)) {
				return null;
			}
			if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
				this.advisedBeans.put(cacheKey, Boolean.FALSE);
				return null;
			}
```

​    1.1 判断当前bean是否存在advisedBeans中（保存了需要增强的bean）    

​    1.2 判断当前bean是否是基础类型的Advice，Pointcut,AopInfrastructureBean，或者是否是切面（@Aspect）
​    1.3 是否需要跳过
​        	1.3.1 获取候选的增强器（切面里面的通知方法）List<Advisor> candidateAdvisors
​        			  每一个封装通知方法的增强器InstantiationModeAwarePointcutAdvisor
​       	 	       判断每一个增强器是否是AspectJPointcutAdvisor类型，返回true
​        	1.3.2 永远返回false

2  PostProcessorsAfterInitialization
        AbstracAutoProxyCreator#wrapIfNecessary(bean,beanName,cacheKey);
        2.1 获取能在当前bean使用的所有增强器（通知方法）object[] specificInterceptors
              2.1.1 找到候选的所有bean的增强器（找到那些方法是需要切入当前bean方法的）
              2.1.2 获取到能在bean使用的增强器
              2.1.3 给增强器排序
        2.2 保存当前bean在advisedBeans中
        2.3 如果当前bean需要增强，创建当前bean的代理对象
             2.3.1 获取所有的增强器（通知方法）
             2.3.2 保存到proxyFactory中
             2.3.3 创建代理对象
                		JdkDynamicAopProxy（config）JDK动态代理
                		ObjenesisCglibAopProxy(config) cglib动态代理
        2.4 给容器返回当前组件使用cglib增强了的代理对象
        2.5 容器中获取的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行

## 目标方法的执行

1 CglibAopProxy.intercept()方法，拦截目标方法执行
2 根据ProxyFactory对象获取将要执行目标方法的拦截器链。
   List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
    2.1 List<Object> interceptorList保存所有的拦截器
    2.2 遍历所有的增强器，并将其转化为Interceptor
       	MethodInterceptor[] interceptors = registry.getInterceptors(advisor)
    2.3 将增强器转为List<MethodInterceptor>
       	如果是MethodInterceptor,直接加入到集合中
       	如果不是，使用AdvisorAdapter将增强器转化为MethodInterceptor
       	转换完成返回MethodInterceptor数组
3 如果没有拦截器链，直接执行目标放啊
4 如果有拦截器链，把需要执行的目标对象，目标方法，拦截器链等信息传入创建一个CglibMethodInvocation对象
    并调用Object retVal = mi.proceed()
5 拦截器链的触发过程
    5.1 如果没有拦截器执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法
    5.2 链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器会等待下一个拦截器执行完成返回以后再来执行
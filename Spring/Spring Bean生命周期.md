<img src="Resources\BeanFactory中Bean的生命周期.png" style="zoom:60%;" />

```java
//创建POJO类
public class Car implements BeanFactoryAware, BeanNameAware, InitializingBean, DisposableBean {

    private String brand;
    private String color;
    private int maxSpeed;

    private BeanFactory beanFactory;
    private String beanName;

    public Car(){
        System.out.println("v2-调用Car()构造函数");
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public int getMaxSpeed() {
        return maxSpeed;
    }

    public void setMaxSpeed(int maxSpeed) {
        this.maxSpeed = maxSpeed;
    }

    public void setBrand(String brand){
        System.out.println("v5-调用setBrand（）设置属性");
        this.brand = brand;
    }

    public void introduce(){
        System.out.println("brand:"+brand+";color"+color+";maxSpeed:"+maxSpeed);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("v7-调用BeanFactoryAware.setBeanFactory().");
        this.beanFactory=beanFactory;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("v6-调用BeanNameAware.setBeanName()");
        this.beanName = name;
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("v12-调用DisposableBean.destroy().");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("v9-调用InitializingBean.afterPropertiesSet().");
    }

    public void myInit(){
        System.out.println("v10-调用init-method方法指定的myInit（）,将maxSpeed设置240.");
        this.maxSpeed = 240;
    }

    public void myDestroy(){
        System.out.println("v13-调用destroy-method所指定的myDestroy().");
    }
}



//实现自定义的BeanPostProcessor
public class BeanPostProcessor implements org.springframework.beans.factory.config.BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("car")){
            Car car = (Car)bean;
            if(car.getColor()==null){
                System.out.println("v8-调用BeanPostProcessor。postProcessBeforeInitialization，color为空，设置为默认黑色");
                car.setColor("黑色");
            }
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("car")){
            Car car = (Car)bean;
            if(car.getMaxSpeed()>=200){
                System.out.println("v11-调用BeanPostProcessor。postProcessAfterInitialization，color为空，设置为默认黑色");
                car.setMaxSpeed(200);
            }
        }
        return bean;
    }
}


//实现自定义的实例化BeanPostProcessor
/*
总结InstantiationAwareBeanPostProcessor方法:
postProcessBeforeInstantiation()在bean实例化前回调,返回实例则不对bean实例化,返回null则进行spring bean实例化(doCreateBean);
postProcessAfterInstantiation()在bean实例化后在填充bean属性之前回调,返回true则进行下一步的属性填充,返回false:则不进行属性填充
postProcessProperties在属性赋值前的回调在applyPropertyValues之前操作可以对属性添加或修改等操作最后在通过applyPropertyValues应用bean对应的wapper对象
*/
public class MyInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessorAdapter {

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if("car".equals(beanName)){
            System.out.println("v1-InstantiationAware BeanPostProcessor.postProcessBeforeInstantiation");
        }
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if("car".equals(beanName)){
            System.out.println("v3-InstantiationAware BeanPostProcessor.postProcessAfterInstantiation");
        }
        return true;
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if("car".equals(beanName)){
            System.out.println("v4-InstantiationAware BeanPostProcessor.postProcessPropertyValues");
            //在这里pvs获取的是xml文件中定义的属性值，如果需要修改属性值在这个方法里修改。然后调用set方法注入到对象里
            MutablePropertyValues propertyValues = new MutablePropertyValues();
            propertyValues.addPropertyValue("brand","qwe");
            return  propertyValues;
        }
        return pvs;
    }
}

//调用
public static void main(String[] args) {
        Resource res = new ClassPathResource("META-INF/bean7.xml");
        BeanFactory bf = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader((DefaultListableBeanFactory)bf);
        reader.loadBeanDefinitions(res);


        ((ConfigurableBeanFactory)bf).addBeanPostProcessor(new BeanPostProcessor());

        ((ConfigurableBeanFactory)bf).addBeanPostProcessor(new MyInstantiationAwareBeanPostProcessor());

        Car car1 = (Car)bf.getBean("car");
        car1.introduce();
        car1.setColor("红色");

        Car car2 = (Car)bf.getBean("car");
        System.out.println("car1==car2:"+(car1==car2));
        ((DefaultListableBeanFactory)bf).destroySingletons();
    }

```

![](Resources\BeanFactory执行结果.png)

![](Resources\Bean分类.png)
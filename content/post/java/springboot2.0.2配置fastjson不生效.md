---
title: 记一次踩坑:springboot2.0.2配置fastjson不生效
author: 要名俗气
type: post
date: 2023-03-08T06:50:58+00:00
url: /2023/springboot-fastjson-doesnot-work
description: 最近在尝试搭建springboot+dubbo+shiro基于注解的一个项目,突发奇想想把消息转换器从jackson换成fastjson,于是就开始了折腾之路. 轻车熟路的去自定了一个去继承,然后就发现这个竟然过时了?what?点进去看源码,可以看到从spring5.0开始就被@Deprecated,原来是java8中支持接口中有默认方法,所以我们现在可以...
image: https://images.iminling.com/app/hide.php?key=SmJabnJsdTlXbDExYlljU3RzemUyTVRlcHFScHFZRUJYSTR1c1BoYkhrZHE3L2RpK05qbkpLQlprcTFPY2QvY3lJaVB3Y3M9
categories:
  - Spring
tags:
  - fastjson
  - spring
  - springboot
---
最近在尝试搭建springboot+dubbo+shiro基于注解的一个项目,突发奇想想把消息转换器从jackson换成fastjson,于是就开始了折腾之路.

轻车熟路的去自定了一个`SpringMvcConfigure`去继承`WebMvcConfigurerAdapter`,然后就发现这个`WebMvcConfigurerAdapter`竟然过时了?what?点进去看源码:

```java
/**
 * An implementation of {@link WebMvcConfigurer} with empty methods allowing
 * subclasses to override only the methods they're interested in.
 *
 * @author Rossen Stoyanchev
 * @since 3.1
 * @deprecated as of 5.0 {@link WebMvcConfigurer} has default methods (made
 * possible by a Java 8 baseline) and can be implemented directly without the
 * need for this adapter
 */
@Deprecated
public abstract class WebMvcConfigurerAdapter implements WebMvcConfigurer {}
```

可以看到从spring5.0开始就被@Deprecated,原来是java8中支持接口中有默认方法,所以我们现在可以直接实现`WebMvcConfigurer`,然后选择性的去重写某个方法,而不用实现它的所有方法.

于是就实现了`WebMvcConfigurer`:

```java
@Configuration
public class SpringMvcConfigure implements WebMvcConfigurer {

    /**
     * 配置消息转换器
     * @param converters
     */
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
        //自定义配置...
        FastJsonConfig config = new FastJsonConfig();
        config.setSerializerFeatures(SerializerFeature.QuoteFieldNames,
                SerializerFeature.WriteEnumUsingToString,
                /*SerializerFeature.WriteMapNullValue,*/
                SerializerFeature.WriteDateUseDateFormat,
                SerializerFeature.DisableCircularReferenceDetect);
        fastJsonHttpMessageConverter.setFastJsonConfig(config);
        converters.add(fastJsonHttpMessageConverter);
    }

}
```

本以为这样子配置就可以完事儿,但是诡异的事情发生了,我明明注释了`SerializerFeature.WriteMapNullValue`,可是返回的json中仍然有为`null`的字段,然后我就去`com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter`中的`write`和`writeInternal`打了断点,再次执行,竟然什么都没有发生,根本没有走这两个方法,于是在自定义的`SpringMvcConfigure`中`configureMessageConverters`方法内打了断点,想看看这个方法参数`converters`里边到底有什么:

![](https://images.iminling.com/app/hide.php?key=Q3JqbUgyZFo4WENDVlE0ZG1hV1NUQjM4K2lxR2NjT21FdGhmWGlnbGRxQ2FpU29CWkZQMW9qTTdNWEhIcVYzVDVWb3djYkU9)

看到这里就想到,肯定是我自己添加的fastjson在后边,所以没有生效,所以就加了以下代码:

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters = converters.stream()
                .filter((converter)-> !(converter instanceof MappingJackson2HttpMessageConverter))
                .collect(Collectors.toList());
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
    //自定义配置...
    FastJsonConfig config = new FastJsonConfig();
    config.setSerializerFeatures(SerializerFeature.QuoteFieldNames,
            SerializerFeature.WriteEnumUsingToString,
            /*SerializerFeature.WriteMapNullValue,*/
            SerializerFeature.WriteDateUseDateFormat,
            SerializerFeature.DisableCircularReferenceDetect);
    fastJsonHttpMessageConverter.setFastJsonConfig(config);
    converters.add(fastJsonHttpMessageConverter);
}
```

竟然还没有生效,后来开始追踪,开始方法是从`org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport`类中的一个bean配置:

```java
@Bean
public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
    RequestMappingHandlerAdapter adapter = createRequestMappingHandlerAdapter();
    adapter.setContentNegotiationManager(mvcContentNegotiationManager());
    //就是从这里开始设置converters的,然后从这里一路追踪下去.
    adapter.setMessageConverters(getMessageConverters());
    adapter.setWebBindingInitializer(getConfigurableWebBindingInitializer());
    adapter.setCustomArgumentResolvers(getArgumentResolvers());
    adapter.setCustomReturnValueHandlers(getReturnValueHandlers());

    if (jackson2Present) {
        adapter.setRequestBodyAdvice(Collections.singletonList(new JsonViewRequestBodyAdvice()));
        adapter.setResponseBodyAdvice(Collections.singletonList(new JsonViewResponseBodyAdvice()));
    }

    AsyncSupportConfigurer configurer = new AsyncSupportConfigurer();
    configureAsyncSupport(configurer);
    if (configurer.getTaskExecutor() != null) {
        adapter.setTaskExecutor(configurer.getTaskExecutor());
    }
    if (configurer.getTimeout() != null) {
        adapter.setAsyncRequestTimeout(configurer.getTimeout());
    }
    adapter.setCallableInterceptors(configurer.getCallableInterceptors());
    adapter.setDeferredResultInterceptors(configurer.getDeferredResultInterceptors());

    return adapter;
}
```

`getMessageConverters()`方法:

```java
protected final List<HttpMessageConverter<?>> getMessageConverters() {
    if (this.messageConverters == null) {
        this.messageConverters = new ArrayList<>();
        configureMessageConverters(this.messageConverters);//进入这一步
        if (this.messageConverters.isEmpty()) {
            addDefaultHttpMessageConverters(this.messageConverters);
        }
        extendMessageConverters(this.messageConverters);
    }
    return this.messageConverters;
}
```

`org.springframework.web.servlet.config.annotation.DelegatingWebMvcConfiguration`:

```
@Override
protected void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    this.configurers.configureMessageConverters(converters);
}
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    for (WebMvcConfigurer delegate : this.delegates) {
        delegate.configureMessageConverters(converters);
    }
}
```

this.delegates包含了springboot的一个默认配置类类:`org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration`里边有一个参数

```
private final HttpMessageConverters messageConverters;
```

for循环里的`delegate.configureMessageConverters(converters)`调用了`WebMvcAutoConfiguration`中的`configureMessageConverters`方法:

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.addAll(this.messageConverters.getConverters());
}
```

执行完这个后,就给`converters`中添加了`10`个转换器了,就是上图中的10个.
`this.delegates`中还有一个就是我们自定义的那个,执行完后,在我们自定义的那个`SpringMvcConfigure`发现我添加的fastjson添加进去了,但是`org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport.getMessageConverters()`,发现converters并没有发现我们添加进去的`FastJsonHttpMessageConverter`,这时突然又想起来:**_java8的stream api每次都是生成一个新的对象,所以导致converters已经不是传递过来的那个converters的引用了(这里也证明了java是值传递,不是引用传递)._**

于是再次改变那个lambda表达式为普通的增强for循环:

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    /*converters = converters.stream().
            filter((converter)-> !(converter instanceof MappingJackson2HttpMessageConverter))
            .collect(Collectors.toList());*/
    for (HttpMessageConverter<?> converter : converters) {
        if (converter instanceof MappingJackson2HttpMessageConverter){
            converters.remove(converter);
        }
    }
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
    //自定义配置...
    FastJsonConfig config = new FastJsonConfig();
    config.setSerializerFeatures(SerializerFeature.QuoteFieldNames,
            SerializerFeature.WriteEnumUsingToString,
            /*SerializerFeature.WriteMapNullValue,*/
            SerializerFeature.WriteDateUseDateFormat,
            SerializerFeature.DisableCircularReferenceDetect);
    fastJsonHttpMessageConverter.setFastJsonConfig(config);
    converters.add(fastJsonHttpMessageConverter);
}
```

再次运行,wtf?报错了:`ConcurrentModificationException`,原来使用for循环遍历过程中不能进行remove操作,于是换成`Iterator`:

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    /*converters = converters.stream()
            .filter((converter)-> !(converter instanceof MappingJackson2HttpMessageConverter))
            .collect(Collectors.toList());
    for (HttpMessageConverter<?> converter : converters) {
        if (converter instanceof MappingJackson2HttpMessageConverter){
            converters.remove(converter);
        }
    }*/
    Iterator<HttpMessageConverter<?>> iterator = converters.iterator();
    while(iterator.hasNext()){
        HttpMessageConverter<?> converter = iterator.next();
        if(converter instanceof MappingJackson2HttpMessageConverter){
            iterator.remove();
        }
    }
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
    //自定义配置...
    FastJsonConfig config = new FastJsonConfig();
    config.setSerializerFeatures(SerializerFeature.QuoteFieldNames,
            SerializerFeature.WriteEnumUsingToString,
            /*SerializerFeature.WriteMapNullValue,*/
            SerializerFeature.WriteDateUseDateFormat,
            SerializerFeature.DisableCircularReferenceDetect);
    fastJsonHttpMessageConverter.setFastJsonConfig(config);
    converters.add(fastJsonHttpMessageConverter);
}
```

再次运行,我去,终于解决了,先是删除`MappingJackson2HttpMessageConverter`,然后添加`FastJsonHttpMessageConverter`,但是不是到为什么进过一系列操作后,`MappingJackson2HttpMessageConverter`还是添加进去了,但是由于`FastJsonHttpMessageConverter`在`MappingJackson2HttpMessageConverter`之前添加,所以对结果不影响.至此,解决了这个问题.

#### 总结

  1. 最重要的还是解决了springboot2.0.2配置fastjson不生效的问题
  2. 更加明白stream api返回的都是全新的对象
  3. 更理解java是值传递而不是引用传递
  4. 了解到想要在迭代过程中对集合进行操作要用Iterator,而不是直接简单的for循环或者增强for循环

如有不正确的地方还请指出,谢谢.
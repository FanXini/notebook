# 组合注解

用一个注解整合多个注解的功能

比如创建一个新注解WiselyConfiguration，整合@ComponentScan和@Configuration

**@WiselyConfiguration**

```java
package com.wisely.highlight_spring4.ch3.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration //1
@ComponentScan //2
public @interface WiselyConfiguration {

	Class<?>[] basePackageClasses() default {};
	
	String[] value() default {}; //3

}

```


---
title: spring
date: 2021-04-20 23:36:29
tags: java spring
---
1. **spring扩展点**
   FactroyBean  
   BeanPostProcess  
   InstantiationAwareBeanPostProcessor  
   BeanNameAware  
   ApplicationContextAware  
   BeanFactoryAware  
   BeanFactoryPostProcessor  
   InitialingBean  
   DisposableBean
   
2. **Spring 单元测试**

3. **spring 事务管理**
   一般声明Spring的事务管理方式就是通过xml文件声明aop配置,但是后面越来越多的通过声明式的方式也就是
   通过注解的方式(```@org.springframework.transaction.annotation.Transactional```)
   接下来通过分析源码的方式解析Spring事务管理的处理流程
   首先Spring解析xml会有一个命名空间的机制,通过在jar包的META-INF下面添加spring.handlers、
   spring.schemas和可选的spring.tooling文件内容  
   这里需要说明一下*Spring*事务管理是单独放在*spring-tx*包依赖里的,所以它的*META-INF*文件夹下面有相应的文件
   spring.schemas文件内容
   
   ```xml
   http\://www.springframework.org/schema/tx/spring-tx-2.0.xsd=org/springframework/transaction/config/spring-tx-2.0.xsd
   http\://www.springframework.org/schema/tx/spring-tx-2.5.xsd=org/springframework/transaction/config/spring-tx-2.5.xsd
   http\://www.springframework.org/schema/tx/spring-tx-3.0.xsd=org/springframework/transaction/config/spring-tx-3.0.xsd
   http\://www.springframework.org/schema/tx/spring-tx-3.1.xsd=org/springframework/transaction/config/spring-tx-3.1.xsd
   http\://www.springframework.org/schema/tx/spring-tx-3.2.xsd=org/springframework/transaction/config/spring-tx-3.2.xsd
   http\://www.springframework.org/schema/tx/spring-tx-4.0.xsd=org/springframework/transaction/config/spring-tx-4.0.xsd
   http\://www.springframework.org/schema/tx/spring-tx-4.1.xsd=org/springframework/transaction/config/spring-tx-4.1.xsd
   http\://www.springframework.org/schema/tx/spring-tx.xsd=org/springframework/transaction/config/spring-tx-4.1.xsd
   ```
   
   spring.handlders文件内容
   
   ```xml
   http\://www.springframework.org/schema/tx=org.springframework.transaction.config.TxNamespaceHandler
   ```
   
   关键就在于上面handlers文件里的内容,Spring会用指定的TxNamespaceHandler类来处理xml里的定义
   TxNamespaceHandler
   
   ```java
   public class TxNamespaceHandler extends NamespaceHandlerSupport {
   	static final String TRANSACTION_MANAGER_ATTRIBUTE = "transaction-manager";
   	static final String DEFAULT_TRANSACTION_MANAGER_BEAN_NAME = "transactionManager";
   	static String getTransactionManagerName(Element element) {
   		return (element.hasAttribute(TRANSACTION_MANAGER_ATTRIBUTE) ?
   				element.getAttribute(TRANSACTION_MANAGER_ATTRIBUTE) : DEFAULT_TRANSACTION_MANAGER_BEAN_NAME);
   	}
   	@Override
   	public void init() {
   		registerBeanDefinitionParser("advice", new TxAdviceBeanDefinitionParser());
   		registerBeanDefinitionParser("annotation-driven", new AnnotationDrivenBeanDefinitionParser());
   		registerBeanDefinitionParser("jta-transaction-manager", new JtaTransactionManagerBeanDefinitionParser());
   	}
   
   }
   ```
   
   关键的init方法里定义了以tx标签前缀为首的标签定义以及对应的标签解析逻辑处理对象
   
   advice标签顾名思义就是利用aop的方式来定义具体的切面逻辑
   annotation-driven标签则确定了注解驱动的逻辑
   jta-transaction-manager标签还没研究
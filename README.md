# Activiti REST - Spring HATEOAS error

This is a minimal project to demonstrate error that occurs when using ```activiti-spring-boot-starter-rest-api``` together with ```spring-boot-starter-hateoas```.

Run the project using Gradle task ```bootRun```. Startup fails with following error:

```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration$HypermediaConfiguration$HalObjectMapperConfiguration': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private com.fasterxml.jackson.databind.ObjectMapper org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration$HypermediaConfiguration$HalObjectMapperConfiguration.primaryObjectMapper; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type [com.fasterxml.jackson.databind.ObjectMapper] is defined: expected single matching bean but found 2: objectMapper,_halObjectMapper
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:334)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1210)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:755)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:757)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:480)
	at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:118)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:686)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:320)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:957)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:946)
	at demo.DemoApplication.main(DemoApplication.java:15)
Caused by: org.springframework.beans.factory.BeanCreationException: Could not autowire field: private com.fasterxml.jackson.databind.ObjectMapper org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration$HypermediaConfiguration$HalObjectMapperConfiguration.primaryObjectMapper; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type [com.fasterxml.jackson.databind.ObjectMapper] is defined: expected single matching bean but found 2: objectMapper,_halObjectMapper
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:561)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:331)
	... 16 common frames omitted
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type [com.fasterxml.jackson.databind.ObjectMapper] is defined: expected single matching bean but found 2: objectMapper,_halObjectMapper
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1054)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:942)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:533)
	... 18 common frames omitted
```

Cause of the error lies in ```RestApiAutoConfiguration``` class:

```java
@Bean
public ObjectMapper objectMapper() {
  // To avoid instantiating and configuring the mapper everywhere
  ObjectMapper mapper = new ObjectMapper();
  return mapper;
}
```

This instance of ```ObjectMapper``` deactivates instances annotated with ```@Primary``` from ```JacksonAutoConfiguration.JacksonObjectMapperConfiguration``` class:

```java
@Configuration
@ConditionalOnClass({ ObjectMapper.class, Jackson2ObjectMapperBuilder.class })
static class JacksonObjectMapperConfiguration {

	@Bean
	@Primary
	@ConditionalOnMissingBean(ObjectMapper.class)
	public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
		return builder.createXmlMapper(false).build();
	}

}
```

Workaround is to manually configure ```ObjectMapper``` instance annotated with ```@Primary```, as shown in ```DemoApplication``` class. To demonstrate workaorund run the project with ```workaround``` profile.

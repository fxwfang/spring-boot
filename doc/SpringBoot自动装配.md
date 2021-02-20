### SpringBoot 自动装配原理
### 问题

```
1. 怎么找到需要装配的类
2. 什么时间点进行自动装配
3. SpringBoot怎么整合到Spring中的？ invokeBeanDefinitionRegistryPostProcessors
4. 

```

### SpringBoot 解析过程

```
SpringApplication.run()-》context = createApplicationContext()//创建上下文-》//判断启动方式{REACTIVE、SERVLET、非web环境} 初始化上下文（以非web环境启动为例）
                      -》创建AnnotationConfigApplicationContext
                             -》this.reader = new AnnotatedBeanDefinitionReader(this)初始化一个读取器reader 
                               -》AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry)注册注解驱动默认各种执行器 （配置类执行器ConfigurationClassPostProcessor.class实现了BeanDefinitionRegistryPostProcessors接口）
                        -》refreshContext(context)//刷新上下文
                                -》AbstractApplicationContext#refresh
                                    -》invokeBeanFactoryPostProcessors(beanFactory);//调用所有注册的BeanFactoryPostProcessor的Bean 
                                        -》PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
                                            -》invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);//调用BeanDefinitionRegistryPostProcessors 实现Ordered接口的 
                                                -》postProcessor.postProcessBeanDefinitionRegistry(registry);
                                                    -》ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry(registry);
                                                        -》processConfigBeanDefinitions(registry);
                                                            -》parser.parse(candidates);//解析
                                                                -》ConfigurationClassParser#parse(AnnotationMetadata,String)//根据类型选择不同的解析方式
                                                                    -》ConfigurationClassParser#processConfigurationClass//解析配置类（判断条件注解是否满足不满足直接返回）
                                                                        -》ConfigurationClassParser#doProcessConfigurationClass
                                                                            -》 ConfigurationClassParser#processImports//解析import标签
                                                                                -》selectImports（）//如果实现DeferredImportSelector接口 则放到deferredImportSelectors中等配置类解析完再调用 否则直接调用selectImports（）选出要加载的类递归ConfigurationClassParser#processImports
                                                                -》ConfigurationClassParser#processDeferredImportSelectors//调用实现DeferredImportSelector的加载选择器 后面的流程和19行差不多
                                                            -》 this.reader.loadBeanDefinitions(configClasses);//加载 注册 BeanDefinitions 有条件注解判断
                                                            -》//判断加载的BeanDefinition是否变化，如果BeanDefinition数量发生变化切还有没有被加载的类信息重复执行parser.parse(candidates) 否则结束
```


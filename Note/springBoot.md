# SpringBoot笔记

<!-- TOC START min:2 max:4 link:true asterisk:false update:true -->
  - [1、@SpringBootConfiguration注解](#1springbootconfiguration注解)
<!-- TOC END -->

### 1、@SpringBootConfiguration注解
- SpringBoot应用标在某个类上，说明这个类是主配置类
- SpringBoot应该运行这个类的main方法来启动SpringBoot应用
-

包含
1. @Congfiguration: 用来标注配置类
  > 配置类也是容器中的一个组件(**@Compponent**)
2. @EnableAutoConfiguration: 表示通知SpringBoot开启自动配置功能
  > 底层为**@Import({AutoConfigurationImportSelector.class})**，
  > 作用是扫描主配置类所在包下及所有子包的所有组件

  >AutoConfigurationImportSelector是导入组件的选择器，给容器中导入了很多自动配置类
  >(xxxAutoConfiguration)

### Spring Boot/Spring Cloud

#### 什么是 spring boot

spring boot 是为 spring 服务的，是用来简化新 spring 应用的初始搭建以及开发过程的。

#### 为什么要用 spring boot

- 配置简单
- 独立运行
- 自动装配
- 无代码生成和 xml 配置
- 提供应用监控
- 易上手
- 提升开发效率

#### spring boot 核心配置文件

- bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，且 boostrap 里面的属性不能被覆盖；
- application (. yml 或者 . properties)：用于 spring boot 项目的自动化配置。

#### spring boot 有哪些方式可以实现热部署

- 使用 devtools 启动热部署，添加 devtools 库，在配置文件中把 spring. devtools. restart. enabled 设置为 true；
- 使用 Intellij Idea 编辑器，勾上自动编译或手动重新编译。
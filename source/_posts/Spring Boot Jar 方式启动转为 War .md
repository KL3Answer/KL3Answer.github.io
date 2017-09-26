---
title: Spring Boot Jar 方式启动转为 War 
date: 2015-09-21 16:33:40
tags: 
	- 学习笔记
---

使用Spring Boot可以让项目被maven package 成 jar 之后直接 以jar的方式运行(运行在自带的tomcat中)，但是如果以war的方式运行怎么办呢？


1. 修改pom.xml
 *  packaging 节点由jar改成war
        
            <packaging>war</packaging>  

 * 排除Spring Boot 自带的tomcat所依赖的相关jar包，并使用所在的tomcat容器中的相关jar包

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <exclusions>
                    <exclusion>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-starter-tomcat</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
                <scope>provided</scope>
            </dependency>
2.  增加SpringBootServletInitializer,并将配置类加入SpringApplicationBuilder

  *  e.g.

            public class SpringBootStartApplication extends SpringBootServletInitializer {
                @Override
                protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
                    return builder.sources(Application.class);
                }
            }

    注:其实也可以通过@Configuration取代override configure方法的方式来加入application。
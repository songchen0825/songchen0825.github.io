---
title: flowable流程图乱码问题
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-02 17:07:24
img:
coverImg:
password:
summary:
tags:
- 乱码
- 疑难杂症
categories: [Support service component,Process engine,flowable]
---
# 问题概述

发布环境Linux ，未安装指定字体 ，所以造成字体混乱，或者 `UnsatisfiedLinkError libfontmanager.so`等问题 。具体处理方法如下 ： 

# 选择字体，并拷贝到系统下（Linux环境）

程序在运行并调用绘制流程图的接口时可能会引用上述设置的相关字体库。而在windwos中字体库位于
目录： 控制面板\所有控制面板项\字体 如图所示：

![](1.png)

将simsun.tcc文件直接放到linux的字体目录 /usr/share/fonts/下 

# 选择字体，并拷贝到系统下（Docker环境）

修改dockerfile文件
添加两行如下： (如果COPY命令后的simsun.ttc文件名大小写根据实际的写，/usr/share/fonts/后的这个simsun.ttc就小写吧)
命令：
COPY SIMSUN.TTC /usr/share/fonts/simsun.ttc
ENV LANG C.UTF-8

# 配置configuration

系统采用springboot，配置一个全局对象存储字体信息。

```java
import org.flowable.spring.SpringProcessEngineConfiguration;
import org.flowable.spring.boot.EngineConfigurationConfigurer;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FlowableConfig  implements EngineConfigurationConfigurer<SpringProcessEngineConfiguration>{


    @Override
    public void configure(SpringProcessEngineConfiguration engineConfiguration) {
        engineConfiguration.setActivityFontName("宋体");
        engineConfiguration.setLabelFontName("宋体");
        engineConfiguration.setAnnotationFontName("宋体");
    }
    
}
```
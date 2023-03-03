---
title: 依靠Springboot注解对数据进行脱敏
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-03 17:11:29
img:
coverImg:
password:
summary:
tags:
- 日志
- 脱敏
categories: [Programming Language,Java language,框架（Framework）,Spring]
---
# 简介
专注安全领域，实现内容脱敏展示，期望做到可灵活配置，灵活启用，并且最好内置丰富插件，支持手机号、邮箱、身份证号、住址、中文名、座机号、银行卡、自定义等多种类型的脱敏配置。

由此诞生了本控件：secure-ext-spring-boot-starter

# 优点
易集成： 只需引入starter包，简单到无需任何初始化配置；

灵活： 具体到方法级；

内置丰富： 内置多种默认类型，且可根据自身需求，支持自定义脱敏规则

自动化： 支持深度脱敏，自动寻找返回值中嵌套对象包含的需脱敏的属性

# 引入依赖

```xml
<dependency>
  <groupId>io.gitee.chemors</groupId>
  <artifactId>secure-ext-spring-boot-starter</artifactId>
  <version>Lastest Version</version>
</dependency>
```

# 开启脱敏配置 
```yaml
sensitive:
  enable: #默认为true
  depth: true #是否开启深度脱敏
  packages: # 扫描包路径，默认为空
  log-info: #日志脱敏
    enable: true
    categories:
      - keywords: name,chineseName
        pre-length: 0
        suf-length: 1
      - keywords: mail,email
        pre-length: 3
        suf-length: 3
      - keywords: mobile,phone
        pre-length: 3
        suf-length: 4
      - keywords: address,addr
        pre-length: 3
        suf-length: 4
```


# 添加注解
注意：

方法注解表示该方法的返回值需要脱敏
属性注解标识具体的脱敏规则
##  添加方法注解
```java
@Desensitization
public Obj test(){
    // 业务逻辑，构建返回对象Obj
    return Obj;
}
```
## 添加属性注解

```java
@DesensitizationProp(SensitiveTypeEnum.MOBILE_PHONE)
 private String mobile;
```

# 默认类型说明

CHINESE_NAME //中文名
ID_CARD // 身份证号
FIXED_PHONE // 电话
MOBILE_PHONE // 手机
ADDRESS //地址
EMAIL //邮箱
BANK_CARD //银行卡号
PASSWORD // 密码
CUSTOM //自定义 （配合 DesensitizationProp 中preLength和sufLength 进行个性化定义）

# 测试rest

```java
@RestController
@RequestMapping("demo/log")
@Slf4j
public class LoggerController {

    @GetMapping("enums")
    public String logConvertByEnum(String mobile,String name){
        log.info("log sensitive demo begin");

        SensitiveEntity sensitiveEntity = new SensitiveEntity();
        sensitiveEntity.setName(name);
        sensitiveEntity.setMobile(mobile);

        SensitiveSubEntity sensitiveSubEntity = new SensitiveSubEntity();
        sensitiveSubEntity.setAddress("中国 河南 郑州");
        sensitiveEntity.setSubEntities(Arrays.asList(sensitiveSubEntity));
        SensitiveEntity sensitiveEntity2 = new SensitiveEntity();
        sensitiveEntity2.setName(name + "222");
        sensitiveEntity2.setMobile(mobile + "333");
        log.info("基于实体类的脱敏---》sensitiveEntity1 is {} and sensitiveEntity2 is {}",sensitiveEntity,sensitiveEntity2);

        return mobile + "^" + name;
    }

    @GetMapping("json")
    public String logConvertByJson(String mobile,String name){

        JSONObject jsonObject = new JSONObject();
        jsonObject.putOpt("chineseName",name);
        jsonObject.putOpt("mobile",mobile);

        log.info("基于json的脱敏---》jsonObject is jsonObject {} ",jsonObject);
        
        return mobile + "^" + name;
    }

    @GetMapping("str")
    public String logConvertByStr(String mobile,String name){
        log.info("基于字符串的脱敏---》name is name^{} , and mobile is mobile^{}",name, mobile);
        return mobile + "^" + name;
    }
}
```

# 手动脱敏工具类 
```java
DesensitizedUtil.mobilePhone(mobile);
```
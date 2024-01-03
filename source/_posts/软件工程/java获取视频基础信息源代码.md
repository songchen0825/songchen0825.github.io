---
title: java获取视频基础信息源代码
cover: false
date: 2024-01-03 11:01:54
tags:
 - java
categories:
 - [软件工程,编码功能案例,java]
top:
coverImg:
---

``` yaml
<dependency>  
    <groupId>ws.schild</groupId>  
    <artifactId>jave-core</artifactId>  
    <version>2.7.3</version>  
</dependency>  
<dependency>  
    <groupId>com.github.dadiyang</groupId>  
    <artifactId>jave</artifactId>  
    <version>1.0.2</version>  
</dependency>
```


``` java
try{  
    File file = File.createTempFile(UUID.randomUUID().toString(),null);  
    try (FileOutputStream fos = new FileOutputStream(file)) {  
        fos.write(bytes);  
    }  
    Encoder encoder = new Encoder();  
    FileChannel fc = null;  
    MultimediaInfo mi = encoder.getInfo(file);  
    long duration = mi.getDuration();  
    int width = mi.getVideo().getSize().getWidth();  
    int height = mi.getVideo().getSize().getHeight();  
    String format = mi.getFormat();  
    int audioChannels = mi.getAudio().getChannels();  
    String audioDecoder = mi.getAudio().getDecoder();  
    int audioSamplingRate = mi.getAudio().getSamplingRate();  
    String videoDecoder = mi.getVideo().getDecoder();  
    float videoFrameRate = mi.getVideo().getFrameRate();  
  
} catch (IOException e) {  
    throw new GenericException(e);  
} catch (EncoderException e) {  
    throw new RuntimeException(e);  
}
```
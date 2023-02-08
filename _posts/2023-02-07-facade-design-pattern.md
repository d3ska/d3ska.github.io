---
title: "Facade Design Pattern"
categories:
  - Blog
tags:
  - Design patterns
  - Facade design pattern
  - SOLID
---

The Facade design pattern is a structural pattern that allows for hiding the complexity of a subsystem by providing a simple interface to it. 
The interface may not include all the functionality that the subsystem has, but only the necessary and important ones for the client. Additionally, other classes such as domain objects and services should have package private visibility. 
This ensures that other clients cannot interact with the implementation in any way other than through the Facade, leading to a highly decoupled relationship.
The downside of that approach may be the fact that a facade can become a god object, that's coupled to other classes in our module/package...

So it's good to be aware of it and eventually when we notice it, it may be great idea to separate a facade to more granular ones.

With facade, it's easy to make changes such as refactoring or modifying domain, subsystem objects without causing conflicts with the clients that use them.

![img]({{site.url}}/assets/blog_images/2023-02-07-facade-design-pattern/facade-uml.png)


The implementation itself it's pretty easy, let's take a look.

```java
class VideoFile {
    private String name;
    private String codecType;

    public VideoFile(String name) {
        this.name = name;
        this.codecType = name.substring(name.indexOf(".") + 1);
    }

    public String getCodecType() {
        return codecType;
    }

    public String getName() {
        return name;
    }
}
```

```java
class MPEG4CompressionCodec implements Codec {
    public String type = "mp4";

}

class OggCompressionCodec implements Codec {
    public String type = "ogg";
}

public class MPEG4CompressionCodec implements Codec {
    public String type = "mp4";

}
```

```java
class BitrateReader {
    public static VideoFile read(VideoFile file, Codec codec) {
        System.out.println("BitrateReader: reading file...");
        return file;
    }

    public static VideoFile convert(VideoFile buffer, Codec codec) {
        System.out.println("BitrateReader: writing file...");
        return buffer;
    }
}

class AudioMixer {
    public File fix(VideoFile result){
        System.out.println("AudioMixer: fixing audio...");
        return new File("tmp");
    }
}
```

All the classes above has package private visibility, and we use them in the facade which would be the API for our package/module for the clients.
It's actually the only entry point for the external clients, and they are not even aware of the inner implementation, which makes it easy to test, refactor and so on...


```java
public class VideoConversionFacade {
    public File convertVideo(String fileName, String format) {
        System.out.println("VideoConversionFacade: conversion started.");
        VideoFile file = new VideoFile(fileName);
        Codec sourceCodec = CodecFactory.extract(file);
        Codec destinationCodec;
        if (format.equals("mp4")) {
            destinationCodec = new MPEG4CompressionCodec();
        } else {
            destinationCodec = new OggCompressionCodec();
        }
        VideoFile buffer = BitrateReader.read(file, sourceCodec);
        VideoFile intermediateResult = BitrateReader.convert(buffer, destinationCodec);
        File result = (new AudioMixer()).fix(intermediateResult);
        System.out.println("VideoConversionFacade: conversion completed.");
        return result;
    }
}
```

The example is from [this](https://refactoring.guru/design-patterns/facade) article. I recommend it to you, to check it out.


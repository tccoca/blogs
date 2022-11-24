title: Java中文乱码的一种解决方案
date: 2016-03-28 20:21:18
tags:
- Java
categories: 
- 开发笔记
---
在jsp中常见的乱码解决办法无外乎是关于get和post两种方式的，但只有切实地在实践中使用时才会注意或者说注重到其他方式。例如，在http请求头中传送中文参数，出现乱码，如何解决？

实际场景：使用Spring提供的RestTemplate向WebService发送put请求，使用HttpHeader类装载需要传递的参数（包括中文）。请求端系统使用的是utf-8编码，而服务端使用的是gbk编码，使用http监听工具查看所发出的http请求信息，发现header中的中文参数乱码。

尝试的方法:

1. 在服务端接收到参数时，utf-8转gbk，无效。
2. 在服务端接收到参数时，iso-8859-1转gbk，无效。
3. 在发送请求前将中文参数转码，utf-8转iso-8859-1，无效。
4. 在请求端，HttpHeader设定ContentType为“application/json;UTF-8"，无效。

<!-- more -->

代码如下：

``` Java
new String(remark.getBytes("UTF-8"), "ISO-8859-1");
--------------------------
headers.setContentType(Media.valueOf("application/json;UTF-8"));
```

写到这里，有人应该感觉到这有点“病急乱投医”的感觉了，没有头绪地在试着各种方式。是的，起初我觉得是请求header中采用了ISO-8859-1的编码，但尝试后很显然不是；后来我觉着是否是RestTemplate中采用的HttpMessageConverter方式所决定的，但没能找到很好的证明方式，查资料说的是StringHttpMessageConverter默认采用的是ISO-8859-1编码，可我觉得我指定了ContentType为application/json，RestTemplate不应该去调用StringHttpMessageConverter啊，其中的原理还有待深究。个人感觉这种情况出问题的可能性最大。

最后，在网上看到一篇文章后，看了一种建议方式，并且是可行的，就是使用URLEncode，将中文参数在传参前进行encode。这里以GBK编码是为了在服务器端接收参数后无需再转码了，如下：

``` Java
list.add(URLEncode.encode(name, "GBK"));
```

URLEncode方式可以解决这种特定场景的中文乱码问题，相信理解其原理后还可以运用到更多的场景。目前我在网上看到的，关于用URLEncode处理中文乱码最多的场景就是文件下载时中文文件名乱码。

关于Java中文乱码的原理及解决办法可以参看一下[这里](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/)。

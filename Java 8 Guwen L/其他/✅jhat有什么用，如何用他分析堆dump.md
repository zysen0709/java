# 典型回答

jhat(Java Heap Analysis Tool),是一个用来分析java的堆情况的命令。使用jmap生成的Java堆的Dump文件可以用jhat命令将其转成html的形式，然后通过http访问可以查看堆情况。

jhat命令解析会Java堆dump并启动一个web服务器，然后就可以在浏览器中查看堆的dump文件了。

# 扩展知识

## 使用

使用jmap命令生成dump：

```c
$ jmap -dump:format=b,file=heapDump 62247
Dumping heap to /Users/hollis/workspace/test/heapDump ...
Heap dump file created
```

以上命令可以将进程6900的堆dump文件导出到heapDump文件中。查看当前目录就能看到heapDump文件。

接下来，解析Java堆转储文件,并启动一个 web server：

```c
$ jhat heapDump

Reading from heapDump...
Dump file created Thu Jan 21 18:59:51 CST 2016
Snapshot read, resolving...
Resolving 341297 objects...
Chasing references, expect 68 dots....................................................................
Eliminating duplicate references....................................................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

使用jhat命令，就启动了一个http服务，端口是7000 ，然后在访问http://localhost:7000/<br />页面如下：<br />[![](https://cdn.nlark.com/yuque/0/2023/png/5378072/1696856886268-0af6bd97-9507-4900-972b-48f25f7e4302.png#averageHue=%23fbfaf9&clientId=ue6657b03-7bc2-4&from=paste&id=ue378ce52&originHeight=369&originWidth=957&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u0312ecd3-74ec-42e7-ae50-6b2ca2ffaf1&title=)](http://www.hollischuang.com/wp-content/uploads/2016/01/QQ20160121-1.png)

接下来，就可以在浏览器里面看到dump文件之后就可以进行分析了。这个页面会列出当前进程中的所有对像情况。

该页面提供了几个查询功能可供使用：

```c
All classes including platform//
Show all members of the rootset
Show instance counts for all classes (including platform)
Show instance counts for all classes (excluding platform)
Show heap histogram
Show finalizer summary
Execute Object Query Language (OQL) query
```

一般查看堆异常情况主要看这个两个部分：<br />**Show instance counts for all classes (excluding platform)**，平台外的所有对象信息。如下图：<br />[![](https://cdn.nlark.com/yuque/0/2023/png/5378072/1696856921301-88ef08f2-5d89-4424-bf15-0b1dbdd9857a.png#averageHue=%23fbfbfa&clientId=ue6657b03-7bc2-4&from=paste&id=u207a50a7&originHeight=234&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ube45922e-89d6-4795-bb0d-e96f1d6abcb&title=)](http://www.hollischuang.com/wp-content/uploads/2016/01/QQ20160121-3.png)<br />**Show heap histogram** 以树状图形式展示堆情况。如下图：<br />[![](https://cdn.nlark.com/yuque/0/2023/png/5378072/1696856921573-1ea1be3a-4149-4208-bbfd-11cbacb2a3c5.png#averageHue=%23eaeae9&clientId=ue6657b03-7bc2-4&from=paste&id=ud48d7b8b&originHeight=440&originWidth=945&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u22db7063-2a1f-4187-bcd5-0e2b617cff7&title=)](http://www.hollischuang.com/wp-content/uploads/2016/01/QQ20160121-2.png)<br />具体排查时需要结合代码，观察是否大量应该被回收的对象在一直被引用或者是否有占用内存特别大的对象无法被回收。

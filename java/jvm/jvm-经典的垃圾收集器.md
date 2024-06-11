# java基础-经典的垃圾收集器
{docsify-updated}

<center>
<img src="pics/gc-collector.png" alt="" width=40% height=40%>
</center>

图3-6展示了七种作用于不同分代的收集器，如果两个收集器之间存在连线，就说明它们可以搭配使用，图中收集器所处的区域，则表示它是属于新生代收集器抑或是老年代收集。


JDK11 启用 ZGC，添加以下命令行参数：
`-XX:+UnlockExperimentalVMOptions -XX:+UseZGC` 
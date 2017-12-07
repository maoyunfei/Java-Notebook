## 以下是java启动命令的语法说明:
（[官方文档说明](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/java.html)）

<img src="https://github.com/maoyunfei/Java-Notebook/blob/master/Java%20Basic/images/java%E5%90%AF%E5%8A%A8%E5%91%BD%E4%BB%A4%E8%AF%AD%E6%B3%95.jpg?raw=true" width = "350" height = "350" align=center />

## 以下是[options]的说明以及一些常用的:

**Standard Options** 所有运行环境都支持

* -D 用于设置系统变量，由于spring boot会从系统属性读取属性，所以使用`@Value("myDir")`即可获取。

<img src="https://github.com/maoyunfei/Java-Notebook/blob/master/Java%20Basic/images/-D%20option.jpg?raw=true"  width = "50%" height = "50%" align=center />

* -jar 用于指定启动的jar文件，jar文件的manifest必须知道Main-Class

<img src="https://github.com/maoyunfei/Java-Notebook/blob/master/Java%20Basic/images/-jar%20option.jpg?raw=true"  width = "150%" height = "150%" align=center />

**Nonstandard Options** 由Java HotSpot VMs默认提供

* -Xmn 设置新生代的大小

<img src="https://github.com/maoyunfei/Java-Notebook/blob/master/Java%20Basic/images/-Xmn%20option.jpg?raw=true" width = "30%" height = "30%" align=center />

* -Xms 设置内存分配池的最小值，即初始值

<img src="https://github.com/maoyunfei/Java-Notebook/blob/master/Java%20Basic/images/-Xms%20option.jpg?raw=true" align=center />

* -Xmx 设置内存分配池的最大值

<img src="https://github.com/maoyunfei/Java-Notebook/blob/master/Java%20Basic/images/-Xmx%20option.jpg?raw=true" align=center />

对于服务器部署，-Xms和-Xmx通常设置为相同的值。

## 以下是[arguments]说明：

语法为**--{name}={value}**

例如：
`java -jar app.jar --name="Spring"` 。由于spring boot会从command line argument读取属性，所以使用`@Value("name")`即可获取。




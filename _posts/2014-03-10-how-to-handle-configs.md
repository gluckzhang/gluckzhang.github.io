---
date: 2014/3/10 10:21:18 
layout: post
title: 如何处理Web项目的配置信息
thread: 201403101021
categories: Java
tags: Java Config
---

Web开发过程中，有不少配置项需要管理，例如数据库连接、某些操作的开关、缓冲区大小等，那么如何合理的管理这些配置信息呢~？

##1. 最低端方式
最早写配置项时非常的粗暴，就这么回事儿：

```
/*
 * 取得数据库连接
 */
public static Connection getConnection() {
    String url = "jdbc:microsoft:sqlserver://localhost:1433;DatabaseName=hit";
    String user = "sa";
    String password = "101";
    Connection conn = null;
    try {
        Class.forName("com.microsoft.jdbc.sqlserver.SQLServerDriver");
        conn = DriverManager.getConnection(url, user, password);
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace(System.out);
        conn = null;
    }
    return conn;
}
```

这种方式好像没什么优点可以讲，让我们抛弃它吧，太low~~

##2. 写在web.xml中
WEB-INF目录下的web.xml是项目中一个基本配置文件，一些全局的配置项写在这里蛮不错，比如版本信息、当前状态是开发还是发布（因为不同的状态可能使用了不同的数据源）等。应用级别的配置项形式：

```
<context-param>
	<param-name>version</param-name>
	<param-value>2.3</param-value>
</context-param>
```

在需要使用配置信息的地方，先取到ServletContent对象，然后获取配置信息：

	String version = servletContext.getInitParameter("version");

此外，web.xml中还可以给特定的Filter、Servlet提供初始化信息，例如：

```
<filter>
	<filter-name>EncodingFilter</filter-name>
	<filter-class>com.hit10.filters.EncodingFilter</filter-class>
	<init-param>
		<param-name>CharacterEncoding</param-name>
		<param-value>utf-8</param-value>
	</init-param>
</filter>
```

以上片段为编码过滤器`EncodingFilter`提供了一个初始化参数“CharacterEncoding”，过滤器中获取方法为：

```
public class EncodingFilter implements Filter {
	private String characterEncoding;
	public void init(FilterConfig config) throws ServletException {
		// 初始化方法时可以拿到所需的参数
		this.characterEncoding = config.getInitParameter("CharacterEncoding");
	}
}
```

##3. 借助Java.util.Properties类记录配置

其实各类语言都会提供一些配置读写支持，例如Python可以方便地读取`.ini`文件，C#可以使用`App.config`文件保存配置，而Java开发则可以借助JDK中自带的Properties类。  
对应的配置文件是一个扩展名为`.properties`的纯文本，“#”开头表示注释，每一行记录都以键值对的形式记录一条配置信息。例如：  

```
# 数据库相关配置
url=jdbc:mysql://127.0.0.1:3306/upan
……省略若干
username=upan
password=abcdef
```

配置信息的读取方法：

```
// 省略了除读取配置相关的代码
public class ConnectionSource {
	// 配置文件路径
	final static String CONFIG_FILE_NAME = "config/db_dev.properties";

	private static Properties readProperties(InputStream fis) throws IOException {
		Properties p = new Properties();
		// 看，多简单。传入一个文件流，直接将配置信息load进来~
		p.load(fis);
		return p;
	}

	public static void init() {
		Properties p = null;
		try {
			InputStream configFileStream = ConnectionSource.class.getClassLoader().getResourceAsStream(CONFIG_FILE_NAME);
			if (configFileStream == null) {
				// 配置文件不存在时，应该有一套默认的配置
				System.err.println("config file not found at {" + CONFIG_FILE_NAME + "}");
				System.err.println("system will use default config");
				// DefaultConfig中构造了一个带有默认配置信息的Properties
				p = DefaultConfig.getDefaultDBConfig();
			} else {
				p = readProperties(configFileStream);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}

```

##4. 可能存在的路径问题
在Java Web项目开发中，如果要对某个类的某些方法进行测试，我们可能会写一个main方法去调用目标，省去了发布的麻烦。但是如果流程中涉及到了读取配置文件的操作，就可能会存在路径不同的问题。  
以Eclipse建立的Web项目为例，配置文件`src/config/db_dev.properties`的存储路径可能为：

	作为Java Application运行时：
	build/classes/config/db_dev.properties

	部署到服务器上之后：
	WEB-INF/classes/config/db_dev.properties

因为路径的起始部分不同，所以最好通过工具类获取到配置文件的路径，以这个db_dev.properties的位置为例，我们可以在需要使用配置信息的类中这样获取文件：

	// CONFIG_FILE_NAME的值为"config/db_dev.properties"
	// ConnectionSource是当前类名
	ConnectionSource.class.getClassLoader().getResourceAsStream(CONFIG_FILE_NAME);


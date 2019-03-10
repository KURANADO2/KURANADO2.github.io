---
title: Java 设计模式之 Adapter 模式
date: 2018-12-23 21:27:00
comments: true
categories: [Java]
tags: [Java, 设计模式, Adapter 模式]
---

记得自己在公司转正的时候，被同事问到了，当时没回答上来，所以写下这篇博客

<!-- more -->

Adapter 模式即适配器模式。

## 什么是适配器模式

> 将一个类的接口转换成客户希望的另一个接口。Adapter 模式使得原本由于接口不兼容而不能一起工作的类可以在一起工作

适配器模式中的角色：

- Target：目标接口。客户所期望的接口，可以是具体的类，也可以是抽象类或接口
- Adaptee：需要适配的类
- Adapter：适配器。通过包装一个需要适配的对象，把原接口转换成目标接口

## 栗子

前两天网购了一块键盘，收到货时却是这样的：

![](http://imgblog.kuranado.com/ps2 keyboard.jpg)

......，黑心店家竟然发了块 PS2 接口的键盘给我

可把我给气坏了！！！

![](http://imgblog.kuranado.com/qihuaile.jpg)

因为我的笔记本是这样的。。。：

![](http://imgblog.kuranado.com/laptop usb port.jpg)

它只有 USB 接口

黑心店家不给退货，没办法，翻箱倒柜，DIY 了一个 PS2 转 USB 的转换器：

![](http://imgblog.kuranado.com/ps2 to usb converter.jpg)

最终如愿以偿的用上了新键盘

![](http://imgblog.kuranado.com/sugoyineko.jpg)

好了，就以这个场景为例，简单写下代码吧：

### Adaptee

拥有 PS2 接口的键盘是需要被适配的类

```
package com.kuranado.adaptor;

/**
 * PS2 键盘（要被适配的对象）
 * @Author: Xinling Jing
 * @Date: 2018-12-23 13:28
 */
public class PS2KeyboardAdaptee {

    public void specificRequest() {
        System.out.println("处理打字请求");
    }

}
```

### Target

客户（也就是我的笔记本）所期望的 USB 接口

```
package com.kuranado.adaptor;

/**
 * USB 接口（目标接口，客户所期望的接口）
 * @Author: Xinling Jing
 * @Date: 2018-12-23 13:33
 */
public interface USBTarget {

    void handleRequest();

}
```

### Adapter

PS2 和 USB 的转换器，通过组合的方式包装了被适配的对象，并调用被适配对象所具有的功能。
因为客户端只关心 USB 接口的使用，所以需要实现 USBTarget

```
package com.kuranado.adaptor;

/**
 * PS2 到 USB 转接口（适配器）
 * @Author: Xinling Jing
 * @Date: 2018-12-23 13:43
 */
public class PS22USBAdapter implements USBTarget {

    private PS2KeyboardAdaptee adaptee;

    @Override
    public void handleRequest() {
        adaptee.specificRequest();
    }

    public PS22USBAdapter(PS2KeyboardAdaptee adaptee) {
        this.adaptee = adaptee;
    }
}
```

### Client

客户端调用 Target 接口

```
package com.kuranado.adaptor;

/**
 * 电脑客户端类
 * @Author: Xinling Jing
 * @Date: 2018-12-23 13:31
 */
public class ComputerClient {

    public void test(USBTarget target) {
        target.handleRequest();
    }

    public static void main(String[] args) {
        ComputerClient client = new ComputerClient();
        PS2KeyboardAdaptee adaptee = new PS2KeyboardAdaptee();
        USBTarget target = new PS22USBAdapter(adaptee);
        client.test(target);
    }
}
```

程序运行效果：

```
处理打字请求
```

总结几者的关系如下图：

![](http://imgblog.kuranado.com/adapter-uml.png)

## 实际业务中的栗子

上面的例子比较简单，但真正业务中该如何应用适配器模式呢？此处把《研磨设计模式》中的例子拿过来与大家一起学习

### 1. LogModel：日志类，用于保存日志相关信息：

```
/**
 * 日志类
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:01
 */
@Data
public class LogModel implements Serializable {

	private static final long serialVersionUID = -2324527735778406382L;
	
	private String logId;
	/**
	 * 日志内容
	 */
	private String logContent;
	/**
	 * 操作人
	 */
	private String operateUser;
	/**
	 * 操作时间
	 */
	private String operateTime;

	public LogModel() {
	}

	public LogModel(String logId, String logContent, String operateUser, String operateTime) {
		this.logId = logId;
		this.logContent = logContent;
		this.operateUser = operateUser;
		this.operateTime = operateTime;
	}
}
```

### 2. LogFileOperateApi：从文件中读取日志或向文件中写入日志的接口：

```
/**
 *
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:08
 */
public interface LogFileOperateApi {

	List<LogModel> readLogFile();

	void writeLogFile(List<LogModel> logModels);
}
```

### 3. LogFileOperateApi 接口的实现类：

```
/**
 *
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:10
 */
public class LogFileOperateApiImpl implements LogFileOperateApi {

	// 默认的日志路径
	private String logFilePathName = "/Users/jing/Code/GitHub/DesignPatterns/src/main/resources/AdaptorLog.log";

	public LogFileOperateApiImpl(String logFilePathName) {
		if (logFilePathName != null && logFilePathName.trim().length() > 0) {
			this.logFilePathName = logFilePathName;
		}
	}

	@Override
	@SuppressWarnings("unchecked")
	public List<LogModel> readLogFile() {
		File file;
		ObjectInputStream objectInputStream = null;
		List<LogModel> logModels = null;
		try {
			file = new File(logFilePathName);
			if (file.exists()) {
				InputStream inputStream = new FileInputStream(file);
				if (inputStream.available() != 0) {
					objectInputStream = new ObjectInputStream(new BufferedInputStream(new FileInputStream(file)));
					logModels = (List<LogModel>) objectInputStream.readObject();
				} else {
					return null;
				}
			}
		} catch (IOException | ClassNotFoundException e) {
			e.printStackTrace();
		} finally {
			if (objectInputStream != null) {
				try {
					objectInputStream.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		return logModels;
	}

	@Override
	public void writeLogFile(List<LogModel> logModels) {
		ObjectOutputStream objectOutputStream = null;
		File file;
		try {
			file = new File(logFilePathName);
			objectOutputStream = new ObjectOutputStream(new BufferedOutputStream(new FileOutputStream(file)));
			objectOutputStream.writeObject(logModels);
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (objectOutputStream != null) {
				try {
					objectOutputStream.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}
}
```

### 4. Client:

```
/**
 * 客户端
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:01
 */
public class Client {

	public static void main(String[] args) {

		LogModel logModel = new LogModel("0001", "这是第一条测试日志", "JING", "2018-08-03 09:44:35");

		List<LogModel> logModels = new ArrayList<>();
		logModels.add(logModel);

		LogFileOperateApi logFileOperateApi = new LogFileOperateApiImpl("");
		logFileOperateApi.writeLogFile(logModels);
		List<LogModel> models = logFileOperateApi.readLogFile();
		System.out.println(models);
	}
}
```

客户端创建了一个日志对象，并将该日志对象写入了文件，然后从文件中读取出日志，并打印出来：

```
[LogModel(logId=0001, logContent=这是第一条测试日志, operateUser=JING, operateTime=2018-08-03 09:44:35)]
```

这样程序正常运行着，可是突然有一天 leader 说为了方便日志管理，要求把日志存储到数据库中，于是你快速定义了将日志存取到数据库的接口：

### 5. LogDbOperateApi:

```
/**
 *
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:15
 */
public interface LogDbOperateApi {

	/**
	 * 将日志保存到数据库
	 * @param logModel
	 */
	void createLog(LogModel logModel);

	/**
	 * 更新数据库中的日志
	 * @param logModel
	 */
	void updateLog(LogModel logModel);

	/**
	 * 删除数据库中的日志
	 * @param logModel
	 */
	void removeLog(LogModel logModel);

	/**
	 * 获取数据库中的所有日志
	 * @return
	 */
	List<LogModel> getAllLog();

}
```

然后实现该接口：

### 6. LogDbOperateApiImpl：

```
/**
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:48
 */
public class LogDbOperateApiImpl implements LogDbOperateApi {

    @Override
    public void createLog(LogModel logModel) {
        System.out.println("成功插入日志:" + logModel.toString() + "到数据库中");
    }

    @Override
    public void updateLog(LogModel logModel) {
        System.out.println("成功更新数据库中日志:" + logModel.toString());
    }

    @Override
    public void removeLog(LogModel logModel) {
        System.out.println("成功删除数据库中日志:" + logModel.toString() + "到数据库中");
    }

    @Override
    public List<LogModel> getAllLog() {
        System.out.println("已找到数据库中的所有日志");
        return null;
    }
}
```

### 7. Client:

```
/**
 * 客户端
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:01
 */
public class Client {

	public static void main(String[] args) {

		LogModel logModel = new LogModel("0001", "这是一条测试日志", "JING", "2018-08-03 09:44:35");
		LogDbOperateApi logDbOperateApi = new LogDbOperateApiImpl();
		logDbOperateApi.createLog(logModel);
	}
}
```

程序运行结果：

```
成功插入日志:LogModel(logId=0001, logContent=这是一条测试日志, operateUser=JING, operateTime=2018-08-03 09:44:35)到数据库中
```

**到这里我们把 LogFileOperateApi 叫做第一版接口，LogDbOperateApi 叫做第二版接口**

好啦，所有的工作都做完了，终于可以开开心心的去撩妹啦

刚和妹子约好晚上共度良宵，leader 却又找到了你，因为他觉得还 是 把 日 志 存 储 到 文 件 中 比 较 好！！！

![](http://imgblog.kuranado.com/womeishengqi.jpeg)

此刻内心想法：I have a line of MMP to tell you when the perfect timing comes to us. （╯' - ')╯︵ ┻━┻

刚毕业初来乍到，这个问题解决不了日后岂不让人看扁，怎么升职加薪，迎娶白富美？所以你硬着头皮想到了这么几个解决办法：

- 方法一：修改客户端调用，重新修改为调用第一版的接口
- 方法二：按照第二版的接口重新实现一个将日志存取到文件的实现类
- 方法三：不修改客户端调用，编写一个适配器，将第二版的接口适配到第一版的实现上，也就是使用适配器模式

这三个方法哪个更可取呢？

- 方法一：现在所有的业务都使用第二版接口，要更改为第一版接口的话，即要更改整个项目所有地方，费时费力
- 方法二：已经完成的功能何必再重做一遍呢
- 方法三：复用已有代码，省时省力

### 8. Adapter

```
/**
 *
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:59
 */
public class Adapter implements LogDbOperateApi {

	private LogFileOperateApi adaptee;

	public Adapter(LogFileOperateApi adaptee) {
		this.adaptee = adaptee;
	}

	@Override
	public void createLog(LogModel logModel) {
		List<LogModel> logModels = adaptee.readLogFile();
		logModels.add(logModel);
		adaptee.writeLogFile(logModels);
	}

	@Override
	public void updateLog(LogModel logModel) {
		List<LogModel> logModels = adaptee.readLogFile();
		for (int i = 0; i < logModels.size(); i ++) {
			if (logModels.get(i).getLogId().equals(logModel.getLogId())) {
				logModels.set(i, logModel);
				break;
			}
		}
		adaptee.writeLogFile(logModels);
	}

	@Override
	public void removeLog(LogModel logModel) {
		List<LogModel> logModels = adaptee.readLogFile();
		logModels.remove(logModel);
	}

	@Override
	public List<LogModel> getAllLog() {
		return adaptee.readLogFile();
	}
}
```

### 9. Client

此时客户端只要做一点小修改即可：

```
/**
 * 客户端
 * @Author: Xinling Jing
 * @Date: 2018-12-23 19:01
 */
public class Client {

	public static void main(String[] args) {

		LogModel logModel = new LogModel("0002", "这是第二条测试日志", "JING", "2019-09-04 10:55:46");
		List<LogModel> logModels = new ArrayList<>();
		logModels.add(logModel);
		LogFileOperateApi logFileOperateApi = new LogFileOperateApiImpl("");
		LogDbOperateApi target = new Adapter(logFileOperateApi);
		target.createLog(logModel);
		System.out.println(target.getAllLog());
	}
}
```

```
[LogModel(logId=0001, logContent=这是第一条测试日志, operateUser=JING, operateTime=2018-08-03 09:44:35), LogModel(logId=0002, logContent=这是第二条测试日志, operateUser=JING, operateTime=2019-09-04 10:55:46)]
```

整体结构：

![](http://imgblog.kuranado.com/1545569585.png)

第二版接口对应适配器中的 Target，第一版的实现扮演适配器中的 Adaptee。整个适配器模式中最关键的就是 Adapter，它需要实现第二版的接口，但在内部实现的时候通过对象组合的方式调用第一版已经实现的功能。

好啦，这回终于可以结束工作，时间也还早，可以放心的去陪妹子逛街啦！

## 工作中的应用场景

- 旧系统的改造和升级
- 系统维护

## 常见实现

### Java IO

- java.io.InputStreamReader(InputStream)：通过适配器将字节流转换为我们需要的字符流

## 参考资料

- [【GOF23设计模式】 适配器模式](https://www.youtube.com/watch?v=HKVZNuZwVhQ&t=1280s&list=PLkQ01vCRt9bUFnXOD66dabj2NM9Wbjhoc&index=7)
- [学长博客](https://mrdear.cn/2018/03/14/experience/design_patterns--adapter/)
- 《研磨设计模式》


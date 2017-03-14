---
 layout: post
 title:  selenium-grid2 远程并发控制用例执行
 tags: [ 并发, selenium, 线程, browser, java]
 comments: true
 image:
  background: triangular.png
---

今天闲来无事，随意看了一下selenium，突然注意到grid这个功能以前都是，在读有关selenium的文档时候知道有这么个grid远程控制的功能，但一直没有去试过。所以呢，今天就简单的做了这么个小的实验。
首先需要的内容有：
1. slenium-server（包含了HUB和node在里面）
2. 浏览器驱动器 （Firefox 不需要webdriver已经内置了）
3. 虚拟机或者本机也行，主要为了自己可以区分开

一：部署selenium-grid环境，本机执行 命令  Java -jar selenium-server-standalone-2.40.0.jar -port 4555 -role hub  ，执行这个-role hub 意思是将server之行为hub控制集，类似于集线器的意思，可以集中控制Node，执行结果如下：

二：执行 node 命令 ，将node绑定到hub 绑定规则是通过hub开启的机器IP ，接着打开另一个CMD命令窗口执行 命令 
　　　　java -jar selenium-server-standalone-2.40.0.jar -port 4556 -role node -hub http://127.0.0.1:4555/grid/register  
　　　　这个命令中的IP是我本机已经启动了HUB的机器IP，如果你的启动HUB命令的机器IP是 ：172.65.38.44 则，执行node 的机器的命令改为如下：
　　　　java -jar selenium-server-standalone-2.40.0.jar -port 4556 -role node -hub http://172.65.38.44:4555/grid/register
　　　　至于端口号，可以自己指定，只要不和已经使用的端口号冲突就可以，不过还有一点是我这个机器装的是单网卡，如果是多网卡需要指定实际IP作为选择使用哪张网卡。
三.环境搭建完毕，接下来，我运行的代码很简单，这个是从网上当下来的 ，如果你想多个浏览器同时运行，建议使用多线程的方式编程，用循环是不好使的。再次说一点比较基础的知识，
在webdriver中运行浏览器的时候，如果安装的浏览器不是默认的安装路径 需要指定安装路径属性 ，解决方式 ，可以参考这个链接http://sariyalee.iteye.com/blog/1688271

这里面针对的都是本地执行脚本的情况，而对于远程控制执行的时候需要 

　　　　　　
```java
DesiredCapabilities ffDesiredcap = DesiredCapabilities.firefox();  
ffDesiredcap.setCapability("firefox_binary","C:\\Program Files (x86)\\Mozilla Firefox\\firefox.exe");  
WebDriver wd = new RemoteWebDriver(new URL("http://127.0.0.1:4555/wd/hub"), ffDesiredcap);  
```

这样可以执行成功，否则会一直提示Cannot find firefox binary in PATH. Make sure firefox is installed. OS appears to be: XP  
对于远程控制，你执行 System.setProperty("webdriver.firefox.bin","D:\\Program Files (x86)\\Mozilla Firefox\\firefox.exe");是不会起效果的。

以上代码代表的意思，大家下来可以去查询，恕不在这里讨论了。

四.执行的代码粘贴上来，如下。

本地执行：

```java　　
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.*;
import org.openqa.selenium.chrome.*;
import org.openqa.selenium.htmlunit.*;
import org.openqa.selenium.ie.InternetExplorerDriver;
public class selenium_grid_localhost
{

    public static void main(String arg[]) throws Exception
    { 
       System.setProperty("webdriver.firefox.bin","D:\\Program    Files (x86)\\Mozilla Firefox\\firefox.exe");

           WebDriver driver = new FirefoxDriver();
           driver.get("http://www.dangdang.com");
           System.out.println(driver.getCurrentUrl());
        
        }
}    
```
远程执行代码：

```java　
import org.openqa.selenium.*;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver; 
import java.lang.Thread;
import java.net.URL; 
 
 public class selenium_grid_remote
 {
     public static void main(String arg[]) throws Exception
     {      
     DesiredCapabilities ffDesiredcap = DesiredCapabilities.firefox();
     DesiredCapabilities chromeDesiredcap = DesiredCapabilities.chrome();
     DesiredCapabilities ieDesiredcap = DesiredCapabilities.internetExplorer();
 
     ffDesiredcap.setCapability("firefox_binary","C:\\Program Files (x86)\\Mozilla Firefox\\firefox.exe");
     WebDriver wd = new RemoteWebDriver(new URL("http://127.0.0.1:4555/wd/hub"), ffDesiredcap);
     wd.get("http://www.baidu.com");
     Thread.sleep(1200);
     System.out.println(wd.getCurrentUrl());
    } 
}
```
 
五：以上代码执行成功则代表环境是没有问题的，接下来需要注意的东西，是我从网上找来需要注意的东西，仅供大家参考：

多线程并发运行WebDriver的步骤：1.运行hub 2.运行node 3.运行test case 。下面说下具体实现方法。

1. 运行hub。在命令行中输入：java -jar selenium-server-standalone-2.37.0.jar -role hub -maxSession 40 -port 4444

参数中必须指明-role hub 才是运行hub。默认端口是4444，如果端口被占用就需要指定其他。-maxSession是最大处理的会话请求，我这里设置为40。如果不指定的话，默认是1（即单线程模式了）。

2. 运行node。（先说下运行一个node情况）在命令行中输入（下面的命令是一行敲完）：

```
java -Dwebdriver.ie.driver=D:\IEDriverServer.exe -jar selenium-server-standalone-2.37.0.jar -role node -hub http://127.0.0.1:4444/grid/register -maxSession 20 -browser "browserName=internet explorer,version=9,platform=WINDOWS,maxInstances=20" -port 5555
```
由于node是可以运行在不通系统上的，所以指定驱动位置-Dwebdriver.ie.driver=D:\IEDriverServer.exe。参数中必须指明-role node才是运行node。参数-hub 后面是第一步中hub的IP和端口：http：//hub的IP：端口/grid/register  。node默认的maxSession的值就是5（最多并发5个浏览器），即启动一个node会默认有5个firefox、1个chrome、1个IE的实例。如果用IE浏览器的话，就算你的测试case是多线程，最终也会是一个一个的执行。但是如果在后面的-browser的参数中指明maxInstances=5，那么就会同时运行5个浏览器。-browser参数是指明node可以用的浏览器信息。注意，如果node的maxSession和maxInstances设置的有问题，那么hub的命令窗口中会给出警告。通过这里能够知道你的node是否设置成功。运行node后，窗口中也会显示该node的信息。-port是端口号，默认端口是5555，如果端口被占用就需要指定其他。如果你启动第二个node的话，端口就必须指定了，不能是5555。

我设置的node是只运行IE，并且并发数是20，最多有20个IE浏览器在运行。node中的maxSession的值不能超过hub中的。如果想多线程并发要在hub和node的参数中同时指明maxSession值。node中如果用IE浏览器，指明maxSession后还需要指明同样大小的maxInstances值。我的例子最终会同时运行20个IE浏览器。maxSession是说node可以有几个浏览器同时运行，而maxInstances是说某个浏览器可以有几个同时运行。由于我的电脑运行20个IE已经有些卡了，那么可以再另外一个电脑上再运行一个20Session大小的node。个人测试结果：运行一个20Session大小的node和运行2个10Session大小的node没什么差别。运行多个node主要还是为了能够分布式的测试，不至于一个电脑打开太多浏览器。

3.运行test case。首先将上面代码中的44和47行注释掉，将48行注释打开。我们需要用远程的方式将请求提交给hub（后面的/wd/hub是固定的）。

```java
WebDriver driver = new RemoteWebDriver(new URL("http://hub的IP:端口/wd/hub"),capability);
```

由于是远程的方式，所以44行的设置就没什么用了。下面你可以运行你的程序了，你会发现同时启动20个线程，就会有20个IE浏览器同时在运行。

六.我整理的东西完了，有不对的地方大家多多讨论，谢谢。
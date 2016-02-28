# 待定 #

## 第一章 一个简单的Web服务器 ##
本章介绍了java web servers是如何工作的。Web服务器也叫作HTTP（超文本传输协议）服务器，因为服务器使用HTTP协议与客户端（通常是浏览器）进行通信。Java Web服务器使用两个重要的类java.net.Socket 与java.net.ServerSocket来进行HTTP通信。因此，本章首先讨论HTTP，与这两个重要的类，然后，继续介绍简单的Web服务器。

### 超文本传输协议(HTTP) ###
什么是HTTP？HTTP是服务器与浏览器之间，用来通信的协议。它其实就是一个请求与响应的协议。客户端请求一个资源，服务器对此请求作出响应。HTTP协议的传输依靠的是TCP协议（网络知识，译注），默认使用的TCP端口是80。第一版的HTTP协议的版本号是HTTP/0.9，后来被HTTP/1.0所替代。当前使用的版本是HTTP/1.1（HTTP/2.0已经到来，译注），具体协议规范请访问
http://www.w3.org/Protocols/HTTP/1.1/rfc2616.pdf。

*注意：本章只是概要性的介绍下HTTP 1.1协议，目的只是帮助读者更好的理解服务器与客户端之间的通信格式。如果你需要更详细的协议规范，请访问上面的网址*

HTTP协议里，都是客户端通过建立一个TCP连接与发起一个HTTP请求来开始通信的。服务器是没法主动与客户端建立连接的，只能响应（这个好理解，只有陌生人主动联系你的，哪有你主动联系某个陌生人的，因为你压根不知道要联系哪个？译注）。服务器与客户端都是能终止一条已经建立的连接的（其实就是打电话时，谁都可以挂电话。译注）。例如，网站打开太慢，你等得不耐烦，点了“停止加载”按钮，这就中断了与服务器的连接。

### HTTP请求 ###
一个HTTP请求有3部分组成：
- 方法(Method)——统一资源标识符(URI)——协议/版本
- 请求头部(Request headers)
- 实体(Entity body)

下面是一个HTTP请求例子：
>POST /examples/default.jsp HTTP/1.1
>Accept: text/plain; text/html
>Accept-Language: en-gb
>Connection: Keep-Alive 
>Host: localhost 
>User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 98) 
>Content-Length: 33 
>Content-Type: application/x-www-form-urlencoded Accept-Encoding: gzip, deflate 
>
>lastName=Franks&firstName=Michael

第一行声明了“方法 ——统一资源标识符 ——协议/版本”。
>POST /examples/default.jsp HTTP/1.1

POST是请求方法。“/examples/default.jsp”是统一资源标识符，“HTTP/1.1”是采用的协议与版本。
每个HTTP请求只能使用一个HTTP规范里定义好的方法。HTTP 1.1支持的请求方法有GET, POST, HEAD, OPTIONS, PUT, DELETE,TRACE。GET与POST是使用得最普遍的方法。
统一资源标识符（URI），指出了所请求的资源的位置（并非一定得是真正的物理位置，任何虚拟的都可以。译注）。URI里的位置是相对服务器的根目录而言的。因此，URI总是以“/”开头。统一资源定位符（URL），实际上只是URI的一种（Web中分得没那么清，可以认为二者一样。译注），具体请参考http://www.ietf.org/rfc/rfc2396.txt。
请求的头部，包含了许多客户端有用的信息和请求的实体。例如，它包含了浏览器的设置语言，请求实体的长度，等等。请求的头之间，使用回车换行（CRLF）分割开（其实就是行尾敲个Enter，译注）。（回车换行，即enter，包含两个动作“移到行首，移到下一行”，windows里用/r/n,Unix 使用/n。计算机技术知识。译注）
请求的头部，与请求的实体之间，通过一空行进行分割。在请求格式里面，这一空行是非常重要的，因为，它告诉服务器，请求的实体从这一空行之后开始。在许多Web开发书籍中，这一空行被当作HTTP请求的第四部分。
在之前这个HTTP请求的例子中，实体部分只是简单的：
lastName=Franks&firstName=Michael
当然，这只是一个例子。实际情况中，这个实体部分一般都是相当长的。

### HTTP响应 ###
跟HTTP请求一样，HTTP响应也是由三部分组成：
- 协议(Protocol)——状态码(Status code)——描述(Description)
- 响应头部(Response headers)
- 实体(Entity body)

下面是一个HTTP响应例子：
>HTTP/1.1 200 OK 
>Server: Microsoft-IIS/4.0 
>Date: Mon, 5 Jan 2004 13:13:33 GMT 
>Content-Type: text/html 
>Last-Modified: Mon, 5 Jan 2004 13:13:12 GMT 
>Content-Length: 112 
>
>```html
><html> 
><head> 
><title>HTTP Response Example</title> 
></head> 
><body> Welcome to Brainy Software </body> 
></html>```

响应的第一行类似于请求的第一行。响应的第一行告诉我们，使用的是HTTP/1.1协议，请求成功（200就是成功），一切OK。
响应头部里包含的有用信息跟请求头部里的类似。响应的实体，是HTML内容。响应头部与实体之间，同样是用一空白行隔开。

### Socket类 ###
socket（中文译为套接字，译注）是一条网络连接（非物理，译注）的终端。应用程序通过它才能向网络上读取、写入数据。不同电脑上的两个应用程序，通过收发字节流来进行通信。当一个应用程序向另一个应用程序发送消息时，它必须得知道另一个应用程序的IP地址和socket的端口（就好比，你向一个集团内部办公室打电话时，你不仅要知道他们集团的电话号码，还要知道办公室所在的分机号码。译注）。在java中，java.net.Socket类代表了我们刚才所谈到的socket。
当你要创建一个socket时，你可以使用Socket类的众多构造方法中的一个。其中一个构造方法通过接收一个ip，一个端口号来创建：
```java
public Socket (java.lang.String host, int port)
```
其中，host远程机器的名字或ip地址，port是需要连接到的远程应用的端口号。例如，要连接到yahoo.com（端口80），你只需要如下进行构造
```java
new Socket ("yahoo.com", 80)
```
一旦成功创建了一个Socket的实例，你就可以使用该实例来发送、接收字节流了。为了发送字节流，首先，你必须执行Socket类的`getOutputStream`方法，这样才能获得一个`java.io.OutputStream`对象。为了给远程应用发送文本信息，通常都是从返回的`java.io.OutputStream`对象中，构造一个`java.io.PrintWriter`对象。为了接收其它连接发过来的字节流，你必须执行Socket类的`getInputStream`方法，该方法返回一个`java.io.InputStream`对象。
下面的代码片段创建了一个能跟本机（127.0.0.1）发送HTTP请求、接收HTTP响应的Socket。它创建了一个`StringBuffer`对象来保存响应内容，并在控制台打印出来。
```java
Socket socket = new Socket("127.0.0.1", "8080"); 
OutputStream os = socket.getOutputStream();
boolean autoflush = true; 
PrintWriter out = new PrintWriter( socket.getOutputStream(), autoflush); 
BufferedReader in = new BufferedReader( new InputStreamReader( socket.getInputstream() ));
 
// send an HTTP request to the web server 
out.println("GET /index.jsp HTTP/1.1"); 
out.println("Host: localhost:8080");
out.println("Connection: Close"); 
out.println(); 

// read the response 
boolean loop = true; 
StringBuffer sb = new StringBuffer(8096); 
while (loop) { 
	if ( in.ready() ) { 
		int i=0;
		while (i!=-1) {
			i = in.read(); 
			sb.append((char) i);
		}
		loop = false;
	}
	Thread.currentThread().sleep(50);
}

// display the response to the out console 
System.out.println(sb.toString()); 
socket.close();
```
请注意，你必须以HTTP协议的格式发送一个HTTP请求，才能从Web服务器处得到一个正确的响应。如果你读了之前的小节，你应该能够理解上面代码中关于HTTP 请求的部分。

### ServerSocket类 ###
上一小节的Socket类代表了一个客户端socket，例如，你无论什么时候想连接到远程服务时，你都会构造一个Socket。那么，如果你想实现的不是一个客户端程序，而是一个服务端程序，例如HTTP服务器、FTP服务器之类的。该怎么做呢？你需要一条不同的途径。因为，服务器根本不知道客户端会在什么时候连接进来，因此它需要永不停歇的7X24运行着。为了使你的程序能永不停歇的运行，你需要使用`java.net.ServerSocket`类。该类是服务端Socket的一个实现（socket是不分什么客户端、服务端的，你编写的程序，如果是提供服务的，那就是服务端。java里分Socket，Server Socket只是方便开发者而已，译注）。
Server Socket跟Socket是不相同的。Server Socket担任的角色是等候者，等候来自客户端发起的连接请求。一旦有连接请求到来，Server Socket就会创建一个Socket实例来跟客户端进行通信。
为了创建一个Server Socket，你需要使用`ServerSocket`类的四个构造方法中的任意一个。你必须指定Server Socket需要监听的IP与端口。通常而言，都是监听在本机，即127.0.0.1。Server Socket监听的IP，称作“绑定IP”。Server Socket的另一个重要属性是它的“积压量”，即它能容纳的最大连接数（其实，就是网络术语中吞吐量的吞，译注）。
下面是一个具体的`ServerSocket`构造方法：
```java
public ServerSocket(int port, int backLog, InetAddress bindingAddress);
```
请注意，在该构造方法中，bindingAddress必须是`java.net.InetAddress`的一个实例。创建InetAddress对象的一种简单方法是，调用它的`getByName`静态方法，传入一个包含地址的字符串。如下所示：
```java
InetAddress.getByName("127.0.0.1");
```
下面的代码创建了一个监听在本机8080端口的ServerSocket，它的最大连接量设为了1。
```java
new ServerSocket(8080, 1, InetAddress.getByName("127.0.0.1"));
```
一旦创建了一个`ServerSocket`实例，你就可以调用它的接收方法，让它一直处于等待状态，直到有它所监听地址端口的连接请求到达。接收方法，只会在有连接请求到达时，才会返回，且返回的是一个Socket实例。接下来，该Socket实例就可以用来跟客户端程序发送、接收字节流了，正如前一小节所介绍的那样。

### 应用程序 ###
本章的web服务器程序，在ex01.pyrmont包中，由三个类组成：
- HttpServer
- Request
- Response

程序的入口点（main方法）在HttpServer类中。该main方法创建了一个HttpServer的实例，然后，调用它的等待方法。该方法的意义，如它的名字一样，“等待”。等待客户端的连接到达，处理连接，然后发送响应给客户端。它会一直等待，直到它接收到“关闭”命令。
本章的应用程序只能发送文件系统上的静态资源，例如HTML文件，图片。同时，它也会把到达的连接信息在控制台打印出来。然而，它还不会给客户端发送任何响应头部，例如时间、缓存等。
接下来的小节中，让我们来详细看一下这三个类。

### HttpServer类 ###
HttpServer类代表了一个Web服务器程序，它的代码如“例1.1”所示。注意，为了节省空间，“等待”方法的代码在“例1.2”中给出，此处不再给出。

例1.1	HttpServer类
```java
 package ex01.pyrmont;
 import java.net.Socket;
 import java.net.ServerSocket;
 import java.net.InetAddress;
 import java.io.InputStream;
 import java.io.OutputStream;
 import java.io.IOException;
 import java.io.File;
 public class HttpServer {

 /** WEB_ROOT is the directory where our HTML and other files reside. 
 * For this package, WEB_ROOT is the "webroot" directory under the 
 * working directory. 
 * The working directory is the location in the file system 
 * from where the java command was invoked. */ 
 
	public static final String WEB_ROOT = System.getProperty("user.dir") + File.separator + "webroot";
 	// shutdown command 
	private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";
 	// the shutdown command received 
	private boolean shutdown = false;
    public static void main(String[] args) { 
		HttpServer server = new HttpServer();
		server.await();
	}

	public void await() { 
		...
	}
```

例1.2	HttpServer类的await方法
```java
	public void await() {
		ServerSocket serverSocket = null;
		int port = 8080;
		try {
			serverSocket = new ServerSocket(port, 1,
					InetAddress.getByName("127.0.0.1"));
		} catch (IOException e) {
			e.printStackTrace();
			System.exit(1);
		}
		// Loop waiting for a request
		while (!shutdown) {
			Socket socket = null;
			InputStream input = null;
			OutputStream output = null;
			try {
				socket = serverSocket.accept();
				input = socket.getInputStream();
				output = socket.getOutputStream();
				// create Request object and parse
				Request request = new Request(input);
				request.parse();
				// create Response object
				Response response = new Response(output);
				response.setRequest(request);
				response.sendStaticResource();
				// Close the socket
				socket.close();
				// check if the previous URI is a shutdown command
				shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
			} catch (Exception e) {
				e.printStackTrace();
				continue;
			}
		}
	}	
```
该Web服务器程序能提供服务器上的静态资源，这些资源位于WEB_ROOT变量所声明的文件夹（及其子文件夹）中。WEB_ROOT变量的内容初始化如下：
```java
public static final String WEB_ROOT =System.getProperty("user.dir") + File.separator + "webroot";
```
上面的代码中，有一个叫webroot文件夹，该文件夹不仅包含了所有的静态资源，还包含了一些Servlet，你可以用它们来进行测试。
当请求一个静态资源时，你只用在你的浏览器地址栏中输入：
http://machineName:port/staticResource
上面地址中，machineName就是你向服务器发出请求时，服务器的名称或地址。如果你的服务器程序运行在本机，那machinename就可以使用localhost代替。staticResource是WEB_ROOT下的静态资源。
例如，如果你的HttpServer运行在本机，你通过浏览器向它请求index.html资源时，可以在浏览器中输入如下地址：
http://localhost:8080/index.html
要停止该Web服务器程序是，你只用通过浏览器发送一条包含关闭命令字符串的URL即可。关闭命令字符串预先定义在了`HttpServer`类的`SHUTDOWN`静态变量中：
```java
private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";
```
所以，当你要关闭服务器程序时，只用输入如下URL：
http://localhost:8080/SHUTDOWN 
接下来，让我们来探讨一下“列1.2”中的await方法。
为什么命名为await，而不是wait？因为wait是`java.lang.Object`类的方法啦。
`await`方法，首先创建了一个`ServerSocket`实例，接着就进入while循环。
>```java
>serverSocket = new ServerSocket(port, 1,InetAddress.getByName("127.0.0.1"));
...
// Loop waiting for a request
while (!shutdown) {
...
}
>```

while循环内的代码将会一直阻塞在`ServerSocket`实例的`accept`方法上，直到有HTTP请求到达8080端口。
```java
socket = serverSocket.accept();
```
只要接收到HTTP请求，`await`将继续往下执行，从返回的`socket`对象处，获得`java.io.InputStream`与`java.io.OutputStream`，如下所示：
```java
input = socket.getInputStream();
output = socket.getOutputStream();
```
紧接着，`await`将创建一个`ex01.pyrmont.Request`类的实例，并调用它的`parse`方法来解析收到的HTTP请求，从复杂的HTTP协议格式中，提取出信息。
```java 
// create Request object and parse
Request request = new Request(input);
request.parse ();
```
然后，`await`方法将创建一个`ex01.pyrmont.Response`对象，并把之前创建的Request对象传给它，并调用它的`sendStaticResource`方法来发送资源。
最后，`await`方法调用Socket的close方法来关闭该Socket，即关闭连接。接下来，`await`方法对刚才HTTP请求的URI进行判断，判断URI字符串是否就是预先定义好的关闭命令字符串，如果是的话，就会跳出while，进而结束整个程序。如果不是的话，继续循环，直到有下一条HTTP请求到来。

### Request类 ###
类`ex01.pyrmont.Request`代表了HTTP请求。创建该类的实例时，需要在构造方法中传入一个`InputStream`对象，该对象是由负责跟客户端通信的Socket返回的。我们可以通过`InputStream`对象的任一跟“读”相关的方法中，得到HTTP请求的原始格式数据。
Request类的代码如“例1.3”所示，它的两个方法，`parse`与`getUri`分别在“例1.4”与“例1.5”中给出。

例1.3	Request类
```java
package ex01.pyrmont;
import java.io.InputStream;
import java.io.IOException;
public class Request {
private InputStream input;
private String uri;
public Request(InputStream input) {
this.input = input;
}
public void parse() {
...
}
private String parseUri(String requestString) { 
...
}
public String getUri() {
return uri;
}
}
```

例1.4 parse方法
```java
public void parse() {
// Read a set of characters from the socket
StringBuffer request = new StringBuffer(2048);
int i;
byte[] buffer = new byte[2048];
try {
i = input.read(buffer);
}
catch (IOException e) {
e.printStackTrace();
i = -1;
}
for (int j=0; j<i; j++) {
request.append((char) buffer[j]);
}
System.out.print(request.toString());
uri = parseUri(request.toString());
}
```

例1.5 parseUri方法
```java
private String parseUri(String requestString) {
int index1, index2;
index1 = requestString.indexOf(' ');
if (index1 != -1) {
index2 = requestString.indexOf(' ', index1 + 1);
if (index2 > index1)
return requestString.substring(index1 + 1, index2);
}
return null;
}
```
`parse`方法的作用是解析HTTP请求的原始格式数据，它的工作并不是很多。该方法从`InputStream`里读取字节流，并把它还原为HTTP请求格式，存储进`StringBuffer`变量， 然后，通过调用私有方法`parseUri`来进一步解析HTTP请求，得到请求的URI。公共方法`getUri`则是向外界返回URI变量内容。
*注意	更多关于HTTP 原始请求数据的处理过程，将在第三章及以后的章节中展开。*
要理解`parse`与`parseUri`方法是如何工作的，你需要之前章节中关于“超文本传输协议”的知识储备。本章中，我们暂时只讨论HTTP请求的第一行，姑且称为“请求行”吧。还记得它的格式吗？
>方法(Method)——统一资源标识符(URI)——协议/版本

上面的各个元素之间，使用空格分割开来。下面举个具体的例子：
>GET /index.html HTTP/1.1

`parse`方法从`InputStream`（HttpServer传过来的）中读取所有的字节流，并把它们存储在一个字节数组中`buffer`。然后，使用一个叫request的`StringBuffer`来从这些字节中还原出字符，并把最后组成的字符串传递给`parseUri`方法。
`parseUri`方法首先找到第一个空格，与第二个空格的位置，接着截取出这两个位置之间的内容，就是URI。

### Response类 ###
类`ex01.pyrmont.Response`代表了一个HTTP响应。它的代码如“例1.6”所示。

例1.6 Response类
```java
package ex01.pyrmont;

import java.io.OutputStream;
import java.io.IOException;
import java.io.FileInputStream;
import java.io.File;

/*
  HTTP Response = Status-Line
    *(( general-header | response-header | entity-header ) CRLF)
    CRLF
    [ message-body ]
    Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
*/

public class Response {

  private static final int BUFFER_SIZE = 1024;
  Request request;
  OutputStream output;

  public Response(OutputStream output) {
    this.output = output;
  }

  public void setRequest(Request request) {
    this.request = request;
  }

  public void sendStaticResource() throws IOException {
    byte[] bytes = new byte[BUFFER_SIZE];
    FileInputStream fis = null;
    try {
      File file = new File(HttpServer.WEB_ROOT, request.getUri());
      if (file.exists()) {
        fis = new FileInputStream(file);
        int ch = fis.read(bytes, 0, BUFFER_SIZE);
        while (ch!=-1) {
          output.write(bytes, 0, ch);
          ch = fis.read(bytes, 0, BUFFER_SIZE);
        }
      }
      else {
        // file not found
        String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
          "Content-Type: text/html\r\n" +
          "Content-Length: 23\r\n" +
          "\r\n" +
          "<h1>File Not Found</h1>";
        output.write(errorMessage.getBytes());
      }
    }
    catch (Exception e) {
      // thrown if cannot instantiate a File object
      System.out.println(e.toString() );
    }
    finally {
      if (fis!=null)
        fis.close();
    }
  }
}

```
首先注意一点，Response类的构造方法，接受的是一个`java.io.OutputStream`对象，如下所示。
```java
public Response(OutputStream output) {
this.output = output;
}
```
Response对象是HttpServer里的await方法创建的。await方法从socket处获得了OutputStream对象，并以此构造了Response。
Response类有两个公共方法：`setRequest`与`sendStaticResource`。其中，`setRequest`方法用来为Response设置Request对象的引用。
`sendStaticResource`则是用来发送类似HTML文件等静态资源的。该方法首先通过传入一个父路径与子路径给File类的构造方法，以此来实例化一个`java.io.File`对象。
```java
File file = new File(HttpServer.WEB_ROOT, request.getUri());
```
然后，会检查该file对象是否存在，如果存在的话，就会进一步通过该file对象来构造一个`java.io.FileInputStream`对象，然后，通过FileInputStream对象的read方法，读出指定长度的字节，并把这些字节写入OutputStream对象。注意，在本例中，静态资源的内容是被原模原样地发送给浏览器的。
```java
if (file.exists()) {
        fis = new FileInputStream(file);
        int ch = fis.read(bytes, 0, BUFFER_SIZE);
        while (ch!=-1) {
          output.write(bytes, 0, ch);
          ch = fis.read(bytes, 0, BUFFER_SIZE);
        }
      }
```

如果指定文件不存在，`sendStaticResource`方法将会给浏览器发送一条“资源不存在”的错误信息。
```java
 // file not found
        String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
          "Content-Type: text/html\r\n" +
          "Content-Length: 23\r\n" +
          "\r\n" +
          "<h1>File Not Found</h1>";
        output.write(errorMessage.getBytes());
```

### 运行程序 ###
在你的工作目录，编译你的java类，然后在命令行中输入：
`java ex01.pyrmont.HttpServer`
就可以启动服务器程序了。
你可以在你的浏览器地址栏中输入
>http://localhost:8080/index.html 

来测试一下你的程序。
一切正常的话，你将在浏览器中看到如下界面，图1.1。

图1.1
![](images/chapter1-1.png)

同时，你会在控制台看到如下类似的信息：
>GET /index.html HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.109 Safari/537.36
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4

>GET /images/logo.gif HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Accept: image/webp,image/*,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.109 Safari/537.36
Referer: http://localhost:8080/index.html
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4

### 本章小结 ###
在本章中，我们知道了一个简单的Web服务器程序是如何工作的。本章的服务器程序只有三个类，功能还并不完善，但是，却是一个很好的学习样例。在下一章中，我们将讨论如何处理动态内容。

Raphael 2016-2-28 11:03:43 翻译到第25页。
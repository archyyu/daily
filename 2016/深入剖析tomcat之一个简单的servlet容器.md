上一篇，我们讲解了如果开发一个简单的Http服务器，这一篇，我们扩展一下，让我们的服务器具备servlet的解析功能。

简单介绍下Servlet接口
如果我们想要自定义一个Servlet，那么我们必须继承Servlet，并且实现下面几个重要的方法
```
public void init(ServletConfig config) throws ServletException
public void service(ServletRequest request,ServletResponse response) throws ServletException,java.io.IOException
public void destroy()
public ServletConfig getServletConfig()
public String getServletInfo() 
```

五个方法中，init,destroy,service都是和servlet的生命周期相关的方法。当实例化某个servlet类之后，servlet会调用init进行初始化，当servlet的请求到达之后，就会调用service方法，并将servletRequest和servletResponse对象作为参数传入，前者包含客户端的Http请求的信息，后者包含服务器的响应信息。

这个简单的Servlet容器的流程如下
* 等待http请求
* 对应的servletRequest对象和servletResponse对象，
* 判断请求的类型，如果是请求静态资源，则找到静态资源的文件，返回给客户端
* 如果是Servlet请求，载入servlet类，调用service()方法，传入servletRequest对象和servletResponse对象

涉及到的主要的类
* SimpleServletContainerServer
* Request
* Response
* Servlet
* PrimitiveServlet
* StaticProcessor
* ServletProcessor


关于Request和Response的定义在上一篇幅有定义，这里我们稍微扩展了一下，碍于篇幅，不在这里展示。

PrimitiveServlet类，继承自Servlet，Servlet请求的处理类
类定义：
```
package servletContainer;

import java.io.IOException;

import base.Request;
import base.Response;
import base.ServletConfig;
import interf.Servlet;

public class PrimitiveServlet implements Servlet {

	@Override
	public void init(ServletConfig config) {
		System.out.println("PrimitiveServlet init");
	}

	@Override
	public void service(Request request, Response response) throws IOException {
		response.getOutput().write("Primitive Servlet".getBytes());
	}

	@Override
	public void destroy() {
		
	}

	@Override
	public ServletConfig getServletConfig() {
		return null;
	}

	@Override
	public String getServletInfo() {
		return null;
	}

}

```

SimpleServletContainerServer 类
功能：程序入口，监听Http请求，并且负责创建Request和Response
类定义
```
package servletContainer;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;


import base.Request;
import base.Response;
import servletContainer.processor.ServletProcessor;
import servletContainer.processor.StaticProcessor;

public class SimpleServletContainerServer {
	private static final String SHUT_DOWN = "/SHUTDOWN";
	
	private boolean shutdown = false;
	
	private ServletProcessor servletProcessor = new ServletProcessor();
	
	private StaticProcessor staticProcessor = new StaticProcessor();
	
	public static void main(String args[]){
		SimpleServletContainerServer server = new SimpleServletContainerServer();
		server.init();
		server.await();
	}
	
	public void init(){
	 	servletProcessor.init();
	 	staticProcessor.init();
	}
	
	public void await(){
		
		ServerSocket serverSocket = null;
		int port = 8080;
		
		try{
			serverSocket = new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
		}
		catch (IOException e){
			e.printStackTrace();
			System.exit(-1);
		}
		
		while(!shutdown){
			Socket socket = null;
			InputStream input = null;
			OutputStream output = null;
			                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    
			try{
				
				socket = serverSocket.accept();
				input = socket.getInputStream();
				output = socket.getOutputStream();
				
				Request request = new Request(input);
				request.parse();
				
				Response response = new Response(output);
				response.setRequest(request);
				
				if(request.getUri().startsWith("/servlet/")){
					servletProcessor.process(request, response);
				}
				else{
					staticProcessor.process(request, response);
				}
				
				socket.close();
				shutdown = request.getUri().equals(SHUT_DOWN);
				
			}
			catch (Exception e){
				e.printStackTrace();
				System.exit(1);
			}
		}
		
	}
	
}

```

我们引入了StaticProcessor和ServletProcessor进行逻辑的处理，我们看下这两个类的定义
首先这两个类都继承自IProcessor接口
```
package servletContainer.processor;

import base.Request;
import base.Response;

public interface IProcessor {
	
	public void init();
	
	public void process(Request request,
			Response response);
}
```


StaticProcessor类主要是处理静态资源请求
类定义
```
package servletContainer.processor;

import base.Request;
import base.Response;

public class StaticProcessor implements IProcessor {

	@Override
	public void process(Request request, Response response) {
		try{
			response.sendStaticResource();
		}
		catch (Exception e){
			e.printStackTrace();
		}
	}

	@Override
	public void init() {
		
	}
	
}

```

ServeletProcessor主要负责处理Servlet请求，初始化的时候，初始化所有的Servlet子类，接收到servlet的http请求之后，根据请求名称，调用对应的service函数。
类定义
```
package servletContainer.processor;

import java.util.HashMap;
import java.util.Map;

import base.Request;
import base.Response;
import interf.Servlet;
import servletContainer.PrimitiveServlet;

public class ServletProcessor implements IProcessor {
	
	private Map<String,Servlet> map = new HashMap<String,Servlet>();
	
	public ServletProcessor() {
		
	}
	
	public void init(){
		PrimitiveServlet servlet = new PrimitiveServlet();
		servlet.init(null);
		map.put("PrimitiveServlet", servlet);
	}

	@Override
	public void process(Request request, Response response) {
		String uri = request.getUri();
		String servletName = uri.substring(uri.lastIndexOf("/") + 1);
		
		Servlet servlet = map.get(servletName);
		try{
			if(servlet != null){
				servlet.service(request, response);
			}
			else{
				String errorMessage = "HTTP/1.1 404 File Not Found\r\n" + 
						"Content-Type: text/html\r\n" +
						"Content-Length:23\r\n" +
						"\r\n" + 
						"<h1>File Not Found</h1>";
				response.getWriter().print(errorMessage.getBytes());
			}
		}
		catch (Exception e){
			e.printStackTrace();
		}
		catch (Throwable e){
			e.printStackTrace();
		}
		
	}
	
}

```

结果
我们在eclipse里运行结果

![运行结果1](http://upload-images.jianshu.io/upload_images/1261094-d0ba98af7da11eaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![运行结果2](http://upload-images.jianshu.io/upload_images/1261094-47cf0b7f5bd15559.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

restful
====
前后端分离JAVA Restful API实现
服务端采用resteasy和netty的整合

1 添加maven依赖
-------
```
  <dependency>
			<groupId>org.jboss.resteasy</groupId>
			<artifactId>resteasy-netty4</artifactId>
			<version>3.0.12.Final</version>
	</dependency>
	<dependency>
				<groupId>io.netty</groupId>
				<artifactId>netty-all</artifactId>
				<version>4.0.31.Final</version>
	</dependency>
	<dependency>
			   <groupId>org.jboss.resteasy</groupId>
			   <artifactId>resteasy-jackson-provider</artifactId>
			   <version>2.3.5.Final</version>
	</dependency>
	<dependency>
			 	<groupId>org.jboss.resteasy</groupId>
			 	<artifactId>resteasy-multipart-provider</artifactId>
			 	<version>2.3.1.GA</version>
	</dependency>
```
	
2 代码部分
-------

```
package com.fs.common.service;

import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;

import org.jboss.resteasy.plugins.guice.GuiceResourceFactory;
import org.jboss.resteasy.plugins.server.embedded.SecurityDomain;
import org.jboss.resteasy.spi.ResteasyDeployment;
import org.jboss.resteasy.spi.ResteasyProviderFactory;

import com.fs.common.service.annotation.NettyController;
import com.fs.common.service.handler.CorsHeadersChannelHandler;
import com.fs.common.service.handler.OPTIONHandler;
import com.fs.common.utils.LoggerUtils;
import com.google.common.collect.Lists;
import com.google.inject.Binding;
import com.google.inject.Injector;
import com.google.inject.Key;

import io.netty.channel.ChannelHandler;
/**
 * 通用restfull服务类
 * 
 * @author kevin
 */
public class RestfulServer {
	private final String _serverIP;
	private final int _port;
	private MyNettyJaxrsServer _netty;

	public RestfulServer(String ip, int port) {
		_serverIP = ip;
		_port = port;
	}

	/**
	 * 启动restfull服务进程
	 * 
	 * @param injector
	 * @throws Exception
	 */
	public void start(Injector injector) throws Exception {
		ResteasyDeployment deployment = new ResteasyDeployment();
		deployment.setSecurityEnabled(true);
		deployment.setProviderFactory(ResteasyProviderFactory.getInstance());
		
		Map<Key<?>,Binding<?>> bindings = injector.getBindings();
		bindings.forEach((k,v)->{
			Type annotationType = k.getAnnotationType();
			if (annotationType != null && annotationType == NettyController.class) {
				Type type = k.getTypeLiteral().getType();
				deployment.getResourceFactories()
						.add(new GuiceResourceFactory(injector.getProvider((Class<?>) type), (Class<?>) type));
			}
		});

		//开启restfull服务
		start(deployment,null);
		
		LoggerUtils.serverLogger.info("Listening on:" + _serverIP + ":" + _port);
		LoggerUtils.serverLogger.info("has stared");
	}

	/**
	 * 启动服务
	 */
	private void start(ResteasyDeployment deployment,SecurityDomain domain) throws Exception {
		_netty = new MyNettyJaxrsServer();
		_netty.setDeployment(deployment);
		_netty.setPort(_port);
		_netty.setRootResourcePath(_serverIP);
		_netty.setSecurityDomain(domain);
		
		List<ChannelHandler> customHandlers = Lists.newArrayList(new CorsHeadersChannelHandler(),new OPTIONHandler()) ;
		
		_netty.setCustomHandlers(customHandlers);
		_netty.start();
	}

	/**
	 * 关闭
	 * 
	 * @throws Exception
	 */
	public void stop() {
		if(_netty == null){
			return;
		}
		
		try {
			_netty.stop();
		} catch (Exception e) {
			LoggerUtils.serverLogger.error("netty stop error:",e);
		}
		
		_netty = null;
	}

}
```
 

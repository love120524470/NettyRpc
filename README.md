# NettyRpc
An RPC framework based on Netty, ZooKeeper and Spring  
中文详情：[Chinese Details](http://www.cnblogs.com/luxiaoxun/p/5272384.html)
### Features:
* Simple code and framework
* Non-blocking asynchronous call and Synchronous call support
* Long lived persistent connection
* High availability, load balance and failover
* Service Discovery support by ZooKeeper
### Design:
![design](https://images2015.cnblogs.com/blog/434101/201603/434101-20160316102651631-1816064105.png)
### How to use
1. Define an interface:

		public interface HelloService { 
			String hello(String name); 
			String hello(Person person);
		}

2. Implement the interface with annotation @RpcService:

		@RpcService(HelloService.class)
		public class HelloServiceImpl implements HelloService {
			public HelloServiceImpl(){}
			
			@Override
			public String hello(String name) {
				return "Hello! " + name;
			}

			@Override
			public String hello(Person person) {
				return "Hello! " + person.getFirstName() + " " + person.getLastName();
			}
		}

3. Run zookeeper

   For example: zookeeper is running on 127.0.0.1:2181

4. Start server:

   Start server with spring: RpcBootstrap

   Start server without spring: RpcBootstrapWithoutSpring

5. Use the client:
 
		ServiceDiscovery serviceDiscovery = new ServiceDiscovery("127.0.0.1:2181");
		final RpcClient rpcClient = new RpcClient(serviceDiscovery);
		// Sync call
		HelloService helloService = rpcClient.create(HelloService.class);
		String result = helloService.hello("World");
		// Async call
		IAsyncObjectProxy client = rpcClient.createAsync(HelloService.class);
		RPCFuture helloFuture = client.call("hello", "World");
   		String result = (String) helloFuture.get(3000, TimeUnit.MILLISECONDS);


### 需要优化的地方
1，消费方和服务提供方没有保持心跳，在zookeeper挂掉的情况下，无法确认已缓存的服务提供方的连接是否有效
2，消费方只是简单的对服务提供方做了roundrobin的路由策略，不支持一致性hash算法，这样在增加了服务提供方的情况下，会造成全局的服务抖动？
3，服务端和消费端严重依赖Spring，对于一个中间件产品，对外依赖应该越少越好
4，目前只支持probuf的序列化方式
5，目前不支持http以及泛化调用

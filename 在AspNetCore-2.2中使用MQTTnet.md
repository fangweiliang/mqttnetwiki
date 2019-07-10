# 在AspNetCore 2.2中使用MQTTnet
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MQTTnet可以说是.Net 开发者的福音，在众多MQTT开源项目中，也有.Net的一份子。感谢 [Christian](https://github.com/chkr1011 "Christian")！当发现这个开源项目时，我非常有兴趣。于是我开始了一个实验的项目。我并不是一个软件开发高手，项目代码写得很糟糕。所以并没有开源分享。以下我简单介绍关键代码的实现。同时也提出我遇到的问题，我们来共同学习吧！

创建一个设备信息表：Devices.cs
```C#
public class Devices
{
	[Key]
	public int Id { get; set; }

	[Display(Name = "Tid")]
	public int Tid { get; set; }

	public int UserId { get; set; }

	[StringLength(64), Required]
	public string ClientId { get; set; }

	[StringLength(64), Required]
	public string Secretkey { get; set; } = "";

	[StringLength(64)]
	public string Location { get; set; }

	[StringLength(100)]
	public string Address { get; set; }

	[Column(TypeName ="bit")]
	public bool IsOnline { get; set; }

	[Column(TypeName = "bit")]
	public bool Telemetry { get; set; } = true;

	[DisplayFormat(ApplyFormatInEditMode = true, DataFormatString = "{0:yyyy-MM-dd HH:mm:ss}")]
	public DateTime CreateTime { get; set; } = DateTime.Now;
}
```
创建一个设备上报数据记录表：DeviceRawData.cs
```C#
[Table("Device_RawData")]
public class DeviceRawData
{
	[Key]
	public int Id { get; set; }

	[Range(1, int.MaxValue)]
	public int DevId { get; set; }

	[StringLength(30)]
	public string Sensor { get; set; }

	[StringLength(1000), Column(TypeName = "text")]
	public string RawData { get; set; }

	[DisplayFormat(ApplyFormatInEditMode = true, DataFormatString = "{0:yyyy-MM-dd HH:mm:ss}")]
	public DateTime CreateTime { get; set; } = DateTime.Now;
}
```
创建一个设备黑名单的表：DeviceBlacklist.cs
```C#
[Table("Device_Blacklist")]
public class DeviceBlacklist
{
	[Key]
	public int Id { get; set; }

	[StringLength(64), Required]
	public string ClientId { get; set; }

	[StringLength(255), Required]
	public string IPAddress { get; set; }

	[Column(TypeName = "bit")]
	public bool IsEnabled { get; set; }

}
```
设备上报的数据我定义了一个标准，以实现通过反序化获得各项数据。因此创建一个用于序列化消息的类：
```C#
[Serializable]
public class SensorMessage
{
    public string ClientId { get; set; }
    public string Sensor { get; set; }
    public JObject Data { get; set; }
}
```
接下来开始实现MQTT服务的核心功能。在Nuget中安装库 **MQTTnet.AspNetCore**。为了代码的层次更清晰，我建立了一个扩展方法。
```C#
public static class BrokerServerAdapter
{

	public static IServiceCollection UseBrokerServer(this IServiceCollection services)
	{
		//this adds a hosted mqtt server to the services
		services.AddHostedMqttServer(builder =>
		{
			builder.WithDefaultEndpointPort(1883);
		   
			builder.WithPersistentSessions();
#if DEBUG
			builder.WithStorage(new RetainedMessageHandler());
#endif
			builder.WithDefaultCommunicationTimeout(TimeSpan.FromSeconds(30));
			builder.WithApplicationMessageInterceptor(x => new ApplicationMessageRecorder(x));
			builder.WithConnectionValidator(new BrokerConnectionValidator());
			builder.Build();
		});
		//this adds tcp server support based on System.Net.Socket
		services.AddMqttTcpServerAdapter();

		//this adds websocket support
		services.AddMqttWebSocketServerAdapter();

	   

		return services;
	}

}
```
然后在 **Startup.cs** 中调用它。
```C#
public void ConfigureServices(IServiceCollection services)
{
	.....
	.....
	services.UseBrokerServer();
}
```
从我实现的扩展方法中可以看到有几处关键：
- new ApplicationMessageRecorder() --> 消息拦截
- new BrokerConnectionValidator() --> 客户端连接实现的方法
- new RetainedMessageHandler() --> 消息持久化

首先来看 BrokerConnectionValidator 的实现
```C#
public class BrokerConnectionValidator : IMqttServerConnectionValidator
{
	private static readonly ILog Logger = LogManager.GetLogger(Startup.LoggerRep.Name, typeof(BrokerConnectionValidator));
	public Task ValidateConnectionAsync(MqttConnectionValidatorContext context)
	{
		try
		{
		   
			using (var dbContext = new MySqlContext())
			{

				var IPAddress = context.Endpoint.Substring(0, context.Endpoint.IndexOf(":"));
				//检查连接的客户端是否在黑名单内
				var HasBlacked = dbContext.DeviceBlacklist
					.Where(w => w.IsEnabled == true && (w.IPAddress == IPAddress || w.ClientId == context.ClientId))
					.ToList()
					.Any();

				if (HasBlacked || string.IsNullOrWhiteSpace(context.ClientId))
				{
					context.ReasonCode = MQTTnet.Protocol.MqttConnectReasonCode.ClientIdentifierNotValid;
					return Task.CompletedTask;
				}
				// 连接的客户端是否已经登记。
				// 同时这里也可以进行授权相关的验证
				var Secretkey = context.Password;

				var source = dbContext.Devices.FirstOrDefault(w => w.ClientId == context.ClientId);
				if (source == null)
				{
					// 我这里实现了自动添加设备方法
					dbContext.Devices.Add(new Devices
					{
						ClientId = context.ClientId,
						IsOnline = true // 在线状态
					});
				}
				else
				{
					var entry = dbContext.Entry(source);
					entry.State = EntityState.Modified;
					entry.Property(w => w.IsOnline).IsModified = true;
					entry.Property(w => w.IsOnline).CurrentValue = true; // 更新为设备在线状态
				}
				dbContext.SaveChanges();
			}
		}
		catch (Exception ex)
		{
			context.ReasonCode = MQTTnet.Protocol.MqttConnectReasonCode.ServerUnavailable;
			Logger.Error(ex.Message, ex);
		}

#if DEBUG
		Logger.Info($"ClientId: {context.ClientId}" +
			$"; Username: {context.Username}" +
			$"; Password: {context.Password}" +
			$"; ProtocolVersion: {context.ProtocolVersion}" +
			$"; CleanSession: {context.CleanSession}" +
			$"; Endpoint: {context.Endpoint}" +
			$"; AssignedClientIdentifier: {context.AssignedClientIdentifier}" +
			$"");
#endif

		return Task.CompletedTask;
	}
}
```
这里我遇到了一个问题。设备上线状态已经这里获得，但是设备断开这里并没有实现该方法。而我在ActiveMQ中就可以实现设备下线的状态记录。代码片断是这样：

```JAVA
package com.activemq.plugins;

import org.apache.activemq.broker.Broker;
import org.apache.activemq.broker.Connection;
import org.apache.activemq.broker.ConnectionContext;
.....

public class DbAuthenticationBroker extends AbstractAuthenticationBroker {
    private static final Logger LOG = LoggerFactory.getLogger(DbAuthenticationBroker.class);

    private JdbcTemplate jdbcTemplate;
    private boolean checkUserAccess = true;

    private SimpleDateFormat dateformater = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");// 设置日期格式

    public DbAuthenticationBroker(Broker next, JdbcTemplate jdbcTemplate, boolean checkUserAccess) {
        super(next);
        this.jdbcTemplate = jdbcTemplate;
        this.checkUserAccess = checkUserAccess;
    }

    /**
     * 创建连接的时候拦截
     *
     * @param context
     * @param info
     * @throws Exception
     */
    @Override
    public void addConnection(ConnectionContext context, ConnectionInfo info) throws Exception {

        SecurityContext securityContext = context.getSecurityContext();
        String clientId = info.getClientId();
        if (securityContext == null && this.checkUserAccess) {
            LOG.info("username: " + info.getUserName() + "，password=" + EncryptHelper.md5(info.getPassword()));
            securityContext = authenticate(info.getUserName(), info.getPassword(), null);
        }

        if (!checkClient(clientId)) {
            throw new Exception("client id (" + clientId + ") is not found.");
        }

        try {
     
            context.setSecurityContext(securityContext);
            securityContexts.add(securityContext);
            super.addConnection(context, info);
        } catch (Exception e) {
            securityContexts.remove(securityContext);
            context.setSecurityContext(null);
            throw e;
        }

        updateStatus(context, info, true); //设备上线了
    }

    /**
     * 认证
     *
     * @param username
     * @param password
     * @param info
     * @return
     * @throws SecurityException
     */
    @Override
    public SecurityContext authenticate(String username, String password, X509Certificate[] peerCertificates) {
        Users user = getUser(username);
        // 验证用户信息
        if (user != null && user.getPassword().equals(EncryptHelper.md5(password + user.getHexChars()))) {
            return new SecurityContext(username) {
                @Override
                public Set<Principal> getPrincipals() {
                    Set<Principal> groups = new HashSet<Principal>();
                    groups.add(new GroupPrincipal(user.getRoleName()));// 默认加入了users的组
                    return groups;
                }
            };
        } else {
            LOG.warn("authenticate failed");
            throw new SecurityException("user authenticate failed");
        }
    }

    @Override
    public void removeConnection(ConnectionContext context, ConnectionInfo info, Throwable error) throws Exception {
        
        updateStatus(context, info, false); // 这里可以更新设备为下线状态

        SecurityContext securityContext = context.getSecurityContext();
        securityContexts.remove(securityContext);
        context.setSecurityContext(null);
    }


}
```

------------
消息拦截方法 ApplicationMessageRecorder.cs
```C#
public class ApplicationMessageRecorder
{
	public ApplicationMessageRecorder(MqttApplicationMessageInterceptorContext context)
	{
	    var appMessage = context.ApplicationMessage;
	    var clientId = context.ClientId;
	    var message = Encoding.UTF8.GetString(appMessage.Payload);
	    if (string.IsNullOrWhiteSpace(message))
	    {
			return;
	    }
	    try
	    {
			using (var dbContext = new MySqlContext())
			{
				var device = dbContext.Devices
				.Select(s => new { s.ClientId, s.Id, s.Telemetry })
				.AsNoTracking()
				.FirstOrDefault(w => w.ClientId == clientId);
				if (device != null)
				{
					var devId = device.Id;
					string Sensor = "";
					try
					{
						var data = JsonConvert.DeserializeObject<SensorMessage>(message);
						if (dataPoint.Count == 0 || data == null)
						{
							return;
						}
						Sensor = data.Sensor;
						 dbContext.DeviceRawData.Add(new DeviceRawData
						{
							DevId = devId,
							RawData = message,
							IsSuccess = ser,
							Sensor  = sen
						});
						dbContext.SaveChanges();

					}
					catch (Exception ex)
					{
						//Logger.Error(ex.Message, ex);
					}
				}
			}
			context.AcceptPublish = true;
	    }
	    catch (Exception ex)
	    {
			context.AcceptPublish = false;
	    }
	}
}
```
消息持久化 RetainedMessageHandler.cs，我对消息持久化的理解非常片面。因此，没有实现什么业务功能。在这我想描述一下我的困惑。
```C#
public class RetainedMessageHandler : IMqttServerStorage
{
	private static readonly ILog Logger = LogManager.GetLogger(Startup.LoggerRep.Name, typeof(RetainedMessageHandler));
	public Task<IList<MqttApplicationMessage>> LoadRetainedMessagesAsync()
	{

		IList<MqttApplicationMessage> retainedMessages  = null;

		return Task.FromResult(retainedMessages);
	}

	public Task SaveRetainedMessagesAsync(IList<MqttApplicationMessage> messages)
	{
		//var last = messages.LastOrDefault();
		//Logger.Info($"Last: {last.Topic } >> { Encoding.Default.GetString(last.Payload) }");

		foreach (var item in messages)
		{
			Logger.Info($"{item.Topic } >> { Encoding.Default.GetString(item.Payload) }");
		}
		return Task.CompletedTask;
	}
}
```
SaveRetainedMessagesAsync 方法中，收到的消息并不是被排序的。所以你无法获得哪一条是最新传来的消息。如果想实现消息保留量的限制，这变得比较困难。如下你正好看到我的困惑，而且你也很确定地知道如何实现。请告诉我。谢谢！

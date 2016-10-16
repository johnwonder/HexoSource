title: orchard-events
date: 2015-10-08 10:54:14
tags:
---

##   Orchard的Events是如何驱动的

###  ShellDescriptorManager类

ShellDescritorManager有个UpdateShellDescriptor方法
{% codeblock lang:c# %}
public void UpdateShellDescriptor(int priorSerialNumber, IEnumerable<ShellFeature> enabledFeatures, IEnumerable<ShellParameter> parameters) {
            ShellDescriptorRecord shellDescriptorRecord = GetDescriptorRecord();
            var serialNumber = shellDescriptorRecord == null ? 0 : shellDescriptorRecord.SerialNumber;
            if (priorSerialNumber != serialNumber)
                throw new InvalidOperationException(T("Invalid serial number for shell descriptor").ToString());

            if (shellDescriptorRecord == null) {
                shellDescriptorRecord = new ShellDescriptorRecord { SerialNumber = 1 };
                _shellDescriptorRepository.Create(shellDescriptorRecord);
            }
            else {
                shellDescriptorRecord.SerialNumber++;
            }

            shellDescriptorRecord.Features.Clear();
            foreach (var feature in enabledFeatures) {
                shellDescriptorRecord.Features.Add(new ShellFeatureRecord { Name = feature.Name, ShellDescriptorRecord = shellDescriptorRecord });
            }


            shellDescriptorRecord.Parameters.Clear();
            foreach (var parameter in parameters) {
                shellDescriptorRecord.Parameters.Add(new ShellParameterRecord {
                    Component = parameter.Component,
                    Name = parameter.Name,
                    Value = parameter.Value,
                    ShellDescriptorRecord = shellDescriptorRecord
                });
            }

            _events.Changed(GetShellDescriptorFromRecord(shellDescriptorRecord), _shellSettings.Name);
        }
{% endcodeblock %}

看最后的_events.Changed 是调用了IShellDescriptorManagerEventHandler接口的Changed方法。
那为什么一个EventHandler会命名为_events呢？答案在EventsRegistrationSource类中
###  EventsRegistrationSource类
{% codeblock lang:c# %}
public IEnumerable<IComponentRegistration> RegistrationsFor(Service service, Func<Service, IEnumerable<IComponentRegistration>> registrationAccessor) {
            var serviceWithType = service as IServiceWithType;
            if (serviceWithType == null)
                yield break;

            var serviceType = serviceWithType.ServiceType;
            if (!serviceType.IsInterface || !typeof(IEventHandler).IsAssignableFrom(serviceType) || serviceType == typeof(IEventHandler))
                yield break;

            //interface proxy with target其实跟class proxy差不多，在创建代理对象时client指定接口，并且提供一个实现了该接口的对象作为真实对象，DP将创建这个接口的代理对象，对代理对象方法的调用经过拦截器处理之后，最终将调用真实对象相应的方法。与class proxy的不同之处在于，真实对象的方法不必是virtual类型也可以实现拦截
//interface proxy without target比较特殊，创建代理时只需要指定一个接口就可以，DP自动根据接口构造一个实现的类，作为代理对象的类型，但这个代理类只能用于拦截目的，无法像class proxy一样在拦截器中调用真实对象的处理方法。比如在提供了多个拦截器时，最后一个拦截器的接口方法中不能调用invocation.Proceed()方法，否则会抛异常（因为真实对象根本不存在，只有一个假的代理对象）
            var interfaceProxyType = _proxyBuilder.CreateInterfaceProxyTypeWithoutTarget(
                serviceType,
                new Type[0],
                ProxyGenerationOptions.Default);

            //关于代理
            //http://www.cnblogs.com/leiwei/p/3456695.html
            //http://www.cnblogs.com/RicCC/archive/2010/03/15/castle-dynamic-proxy.html
            //http://www.cnblogs.com/Tyrale/archive/2009/06/29/1512907.html

            var rb = RegistrationBuilder
                .ForDelegate((ctx, parameters) => {
                    var interceptors = new IInterceptor[] { new EventsInterceptor(ctx.Resolve<IEventBus>()) };
                    var args = new object[] { interceptors, null };
                    return Activator.CreateInstance(interfaceProxyType, args);
                })
                .As(service);

            yield return rb.CreateRegistration();
        }
{% endcodeblock %}

在构造器中指定了拦截器 EventsInterceptor ，拦截器又传入了EventBus.

###  EventsInterceptor类
我们来看看拦截方法是怎样的：
{% codeblock lang:c# %}
        public void Intercept(IInvocation invocation) {
            var interfaceName = invocation.Method.DeclaringType.Name;
            var methodName = invocation.Method.Name;

            var data = invocation.Method.GetParameters()
                .Select((parameter, index) => new { parameter.Name, Value = invocation.Arguments[index] })
                .ToDictionary(kv => kv.Name, kv => kv.Value);

            var results = _eventBus.Notify(interfaceName + "." + methodName, data);

            invocation.ReturnValue = Adjust(results, invocation.Method.ReturnType);
        }
{% endcodeblock %}

调用了eventBus的Notify方法
### DefaultOrchardEventBus类
{% codeblock lang:c# %}
  public IEnumerable Notify(string messageName, IDictionary<string, object> eventData) {
            // call ToArray to ensure evaluation has taken place
            return NotifyHandlers(messageName, eventData).ToArray();
        }
        //IDictionary<string, object>型参数包含调用方法时要传递的参数
        //接口名称和方法名称查找某一个类型的方法并调用
        private IEnumerable<object> NotifyHandlers(string messageName, IDictionary<string, object> eventData) {
            string[] parameters = messageName.Split('.');
            if (parameters.Length != 2) {
                throw new ArgumentException(T("{0} is not formatted correctly", messageName).Text);
            }
            string interfaceName = parameters[0];//接口名称
            string methodName = parameters[1];//方法名称

            var eventHandlers = _eventHandlers();
            foreach (var eventHandler in eventHandlers) {
                IEnumerable returnValue;
                if (TryNotifyHandler(eventHandler, messageName, interfaceName, methodName, eventData, out returnValue)) {
                    if (returnValue != null) {
                        foreach (var value in returnValue) {
                            yield return value;
                        }
                    }
                }
            }
        }
{% endcodeblock %}

其实就是调用了各个实现IEventHandler接口的类。
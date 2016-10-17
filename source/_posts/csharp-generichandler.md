title: csharp-generichandler
date: 2015-10-20 14:32:59
tags:
---

##   C# 泛型委托书写

###

{% codeblock lang:csharp %}

	//先声明泛型委托
	public delegate T  GetHandler<T>(string propertyName,T t);

	//再定义
	GetHandler<string> strHandler = new GetHandler<string>((ss, s) => { return ""; });

	//再调用
	string str = strHandler("Id", "ss");
{% endcodeblock %}

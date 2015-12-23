#东软物联云权限系统

当你在访问东软物联云平台（以下简称IoT）的HTTP接口的时候，IoT会对你的请求进行权限验证。  

目前，IoT的权限系统主要由两大部分组成。  

* 租户门户的权限系统
* 设备接入的权限系统



##IoT租户门户权限系统使用说明

如果你通过我们的门户网站来访问IoT，那么你并不需要了解我们IoT工作的机制，因为我们的门户网站已经为你封装好了处理权限问题的一切操作。然而，如果你想要设计自己的应用程序，通过直接调用我们的HTTP接口来使用我们的服务，那么了解IoT的权限系统是如何工作的就是非常重要的。  


IoT租户门户的权限系统主要包含以下组成部分：  

* 注册租户
* 租户登录IoT
* 租户访问API

###注册租户

为了流畅的使用IoT为租户提供的各种服务，首先要做的是注册一个IoT的租户。接下来的你几乎所有的操作都将基于你注册的这个租户的信息来进行。  

IoT的租户可以进行哪些操作？  

* 管理工程信息
* 管理设备信息
* 管理设备群组
* 管理租户状态
* 管理设备地理位置
* 其他

如果你需要通过自己的应用来注册一个IoT帐号，你只需要按照下面的URL来发送请求即可：  

    /V1/tenantportal/private/tenant

在发送的请求的请求体中，你需要按照下面的样例来组织数据：  

>开始倒数3，2，1  
删除我的孤单  
more and more 尽是深刻

    需要注意的是，该URL需要以POST的方式发送。

按照规定的格式准备好HTTP的请求以后，你就可以向IoT发送这一请求。之后你将得到来自IoT的HTTP响应。响应体中的数据可能如下所示：  

>爱亮了，爱笑了  
I'm always online

###登录

经过上述的操作步骤以后，你已经获得了一个IoT帐号。现在你可以用它来访问IoT的服务。但是在访问除了登录以外的其他服务之前，你必须要先得到一个token，否则你的请求将不会通过权限验证。当然你现在可以通过登录操作来获得token。

要实现登录，你只需要访问/V1/tenantportal/public/tenant/login，并且按照指定的格式向IoT传送刚刚通过调用注册服务获得的IoT的租户信息即可。

    访问该URL需要以POST的方式，但是你不需要在请求体当中携带任何数据。
    你的用户名和密码可以通过写在请求头中发送到IoT。
    之后IoT就会分别从名为username和password的请求头中获取你的用户名和密码，完成登录操作。

IoT会首先匹配您的用户名和密码信息，如果它们是正确的，IoT就会将用户名所对应的帐号信息作为相应体返回给您。同时IoT会进行一系列的计算，并最终为您的用户名生成一个token。token是一个长字符串，它被存放在名为token的响应头中。

>注意：
*>用来登录的用户名和密码必须是您为自己申请的用户名和密码，不能是您在IoT中为您的设备申请的用户名和密码或者已经失效的用户名和密码，否则您发出的请求最终会以收到状态码为500的HTTP响应而告终。
*>如果您收到的响应的HTTP状态码是401，说明您向IoT请求的密码可能出现了拼写错误。

如果您想使用自己的应用程序访问我们的IoT的服务，那么，这时您必须让您的应用程序中记录下响应头中的token。您记录下的token会在以后的IoT的其他服务的访问中成为主角！

###访问API

现在你应该已经通过登录服务的调用获得了token，现在我们来访问IoT的API。下面的例子将会以创建一个工程为例，向你说明IoT的权限系统是如何起作用的。

IoT创建工程的HTTP接口是/V1/tenantportal/public/project。现在我们将下列数据组织在POST请求的请求体中:

{
"name":"测试工程39dcxw08",
"description":"用来测试token的测试工程",
"tenantId":"20"
}

正如上面的数据格式所显示的那样，name属性和description属性的内容可以随意填写，而tenantId这一属性需要填写刚才你通过注册获得的租户信息中给出的tenantId。

如果你完全按照上面的步骤来操作，并且发送该HTTP请求，这时你会发现IoT并不会返回给你一个HTTP状态码为200的HTTP响应。IoT返回给你的HTTP响应的HTTP状态码将会是401，并且响应体中的数据如下：

{
    "result": "invalid token"
}

这是因为在这次的请求中，你并没有添加token的信息在请求头中。而创建工程的服务是不允许直接访问的，你需要登录后才能获得访问该服务的权限。

在登录操作以后，你获得了一个名为token的长字符串。现在就是这个名为token的字符串（以下简称token）发挥作用的时候。

重新按照上面的步骤准备好一个带发送的POST请求，与上次请求不同的是，现在你需要在请求头中加入token。再次发送以后，你将会得到一个HTTP状态码为200的HTTP响应。响应体中的数据可能如下所示：

{
    "id": 331,
    "name": "测试工程39dcxw08",
    "description": "用来测试token的测试工程",
    "partitionPolicy": null,
    "status": 1,
    "domain": "E03CA690C5A94F53A5BFEDD63FB944DD",
    "createTime": "2015-12-22 02:40:18",
    "updateTime": null,
    "tenantId": 20
}

同时，如果你细心留意一下本次请求所获得响应的相应头，你会发现本次的HTTP响应又携带了一个名为token的响应头回来。而如果你将本次返回的token与上次进行请求所使用的token进行比对的话，你最终会发现——这两个token是不一样的！

这是什么原因呢？

因为IoT会对每次生成的token加一个访问时间长度的限制，这使得每个token有效的时间是固定的。通过调用登录服务获得的token的有效时间只有15分钟，如果你在这期间没有使用该token去访问IoT的API，那么该token就会失效。如果想要继续访问IoT的API，你必须重新调用登录服务，获得一个新的token才行。

然而，由于每个token的有效时间只有15分钟。所以完成登录操作以后，你无论使用不使用登录服务返回给你的token，在15分钟以后它都会失效。那么这是否意味着，每隔15分钟，我们都需要重新调用一次登录服务来获得下个15分钟的API访问的资格呢？

当然不需要这么麻烦！事实上，你每次使用token访问API所获得的HTTP响应头中的token就可以作为下一次访问API时所需要的token。而新的token又可以提供15分钟的访问时间。

IoT会在每次一个有效的token访问IoT的API的时候，获得你的请求头中的token，并进行解析。如果请求头中的token不能够进行解析，那么说明你使用的token是无效的。无效的原因可能是你使用的并不是IoT生成的token或者这是一个IoT生成的但是已经过期的token。如果请求头中的token可以被解析，那么IoT将通过token获得你的租户信息，并判断你的租户信息是否有效。在你的租户信息有效的情况下，IoT会根据你的租户信息为你重新创建一个基于当前时间的token。也就是说，新生成的token就可以提供从当前时间开始、持续15分钟的API访问权限。随后，新生成的token会随着你请求的API的返回信息，通过一个HTTP的响应返回给你的应用。

作为测试，你可以使用创建工程后返回的token来继续访问IoT提供的用来查看工程的API——/V1/tenantportal/public/project/{id}。以刚才的创建工程的操作为例，我们可以查看id为331的工程的信息。

向IoT的/V1/tenantportal/public/project/331发送GET请求，请求头中携带创建工程时生成的token。则得到HTTP响应的响应体中数据可能如下所示：

{
    "id": 331,
    "name": "测试工程39dcxw08",
    "description": "用来测试token的测试工程",
    "partitionPolicy": null,
    "status": 1,
    "domain": "E03CA690C5A94F53A5BFEDD63FB944DD",
    "createTime": "2015-12-22 02:40:18",
    "updateTime": null,
    "tenantId": 20
}

当然，你应该会想到，在执行完这样的一次请求以后，响应头中还会携带一个新的token。而正如上面描述过的那样，新的token又可以提供从当前时间开始、持续15分钟的API访问权限。

>注意：虽然每一次访问API的请求过后，旧的token仍然可以使用，但是由于它在创建成功15分钟后一定会失效，因此我们建议你的应用程序在每一次请求过后更新下一次访问将要使用的token，这样就可以实现登录后对IoT的服务的调用只有在15分钟无操作后才会彻底断开。否则的话，你的应用程序必须在每15分钟后重新登录，才能再次获得对IoT的API的访问权限。

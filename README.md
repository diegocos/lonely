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
* 注销

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

按照规定的格式准备好HTTP的请求以后，你就可以向IoT发送该请求。之后你将得到来自IoT的HTTP响应。响应体中的数据可能如下所示：  

>爱亮了，爱笑了  
I'm always online

###租户登录IoT

经过上述的操作步骤以后，你已经获得了一个IoT帐号。现在你可以用它来访问IoT的服务。但是在访问除了登录以外的其他服务之前，你必须要先得到一个token，否则你的请求将不会通过权限验证。当然你现在可以通过登录操作来获得token。

要实现登录，你只需要访问/V1/tenantportal/public/tenant/login，并且按照指定的格式向IoT传送刚刚通过调用注册服务获得的IoT的租户信息即可。

访问该URL需要以POST的方式，但是你不需要在请求体当中携带任何数据。你的用户名和密码可以通过写在请求头中发送到IoT。之后IoT就会分别从名为username和password的请求头中获取你的用户名和密码，完成登录操作。  

IoT会首先匹配你的用户名和密码信息，如果它们是正确的，IoT就会将用户名所对应的租户信息作为相应体返回给您。同时IoT会进行一系列的计算，并最终为你的生成一个token。token是一个长字符串，它被存放在IoT的缓存中，并附在名为token的响应头中跟随租户信息返回。

>注意：  
用来登录的用户名和密码必须是你通过IoT的注册服务为自己申请的用户名和密码，不能是你在IoT中为你的设备申请的用户名和密码或者已经失效的用户名和密码，否则你发出的请求最终会以收到状态码为500的HTTP响应而告终。  
如果你收到的响应的HTTP状态码是401，说明你向IoT请求的密码可能出现了拼写错误。

如果你想使用自己的应用程序访问我们的IoT的服务，那么，这时你必须确认你的应用程序记录下了响应头中的token。你的应用程序中记录下的token会在下面将要进行说明的IoT的API服务的访问中发挥重要作用。

###租户访问API

经过上面的步骤，你应该已经通过登录服务的调用获得了token，现在我们来访问IoT的API服务。下面的例子将会以创建一个工程为例，向你说明IoT的权限系统是如何起作用的。

IoT创建工程的HTTP接口是/V1/tenantportal/public/project。现在我们将下列数据组织在POST请求的请求体中:

>{  
    "name":"测试工程39dcxw08",  
    "description":"用来测试token的测试工程",  
    "tenantId":"20"  
}  

正如上面的数据格式所显示的那样，name属性和description属性的内容可以随意填写，而tenantId这一属性需要填写刚才你通过注册获得的租户信息中给出的tenantId。

如果你完全按照上面的步骤来操作，并且发送该HTTP请求，这时你会发现IoT并不会返回给你一个HTTP状态码为200的HTTP响应。IoT返回给你的HTTP响应的HTTP状态码将会是401，并且响应体中的数据如下：

>{  
    "result": "invalid token"  
}  

这是因为在这次的请求中，你并没有添加token的信息在请求头中。而创建工程的服务在未登录的条件下是不允许直接访问的，你需要登录后才能获得访问该服务的权限。而IoT判断你是否登录的方式就是通过判断你的请求头中的token是否合法。

在登录操作以后，你获得了一个名为token的长字符串。现在这个名为token的字符串（以下简称token）可以正式发挥作用了。

重新按照上面的步骤准备好一个待发送的POST请求，与上次请求不同的是，现在你需要在请求头中加入token。再次发送以后，你将会得到一个HTTP状态码为200的HTTP响应。响应体中的数据可能如下所示：

>{    
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

同时，如果你细心留意一下本次请求所获得响应的相应头，你会发现本次的HTTP响应又携带了一个名为token的响应头回来。这时候如果你将IoT再次返回的token与之前保存的token作对比的话，一般情况下你应该会发现两次的token是相同的。但如果你发现IoT新返回的token与原token不同，那么这个时候你可能距离登录操作已经有了将近12个小时的时间。

如果在登录操作12个小时以后，你进行的本次请求，那么你的请求将无法成功，返回码是401。

这意味着什么呢？

因为IoT会对每次生成的token加一个访问时间长度的限制，这使得每个token有效的时间是固定的。通过调用登录服务获得的token的有效时间只有12小时，如果你在这期间没有使用该token去访问IoT的API，那么12小时以后，该token就会失效。token失效以后，如果还想要继续访问IoT的API，你必须重新调用登录服务，获得一个新的token才行。而如果在这12小时的期限内，你使用该token去完成了一次API的访问，就会出现两种情况：如果距离该token的失效还有20分钟以上的时间，那么IoT在完成对API请求的基本处理以后，会将原token原封不动地放在响应头中返回给你，同时token的失效时间不发生改变，也就是说，该token还是会按原定失效时间作废。但是如果本次请求距离原token的失效时间已不足20分钟，那么IoT在完成对API请求的基本处理以后，就会生成一个新的token，而新的token的有效时间将会持续到请求结束后的12小时以内。

>注意：  
新的token生成后，原token仍然可用。原token会严格按照它被创建时候的有效时间去发挥作用。但既然新的token被创建了出来，说明原token的有效时间已不足20分钟了。因此我们建议当你的应用程序收到每一个token时，都将它存下用于之后的对IoT的API的访问，而不需要去判断响应头中的token是否是新的。因为如果token离过期时间还早，IoT返回给你的原token还可以为你提供相当长的时间段内的访问许可。而如果你使用的token快过期了，IoT也会为你生成新的可以提供长时间访问的token。总之，让IoT为你进行是否使用新token的判断，而你的应用程序只管去存最后一次请求返回的token即可。

接着上面的例子，在成功地创建了一个工程以后，你可以向IoT请求查看这个工程的详细信息，请求的URL如下：

    /V1/tenantportal/public/project/{id}

由于刚才创建的工程id是331，所以可以按照如下格式组织URL数据，请求以GET方式发送：

    /V1/tenantportal/public/project/331

作为测试，你可以使用创建工程后返回的token来重新访问查看工程的API。以刚才的创建工程服务生成的工程为例，我们将会得到id为331的工程的信息如下：

>{  
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

当然，正如你所能想象到的那样，在成功执行完这样的一次请求以后，响应头中还会携带一个token。如同前面的描述，如果token是新的，说明原token距离它的生命期限已不足20分钟了。大部分情况下，返回的token与发送请求使用的token应该是同一个。当然如果原来的token是已经超期的，那么本次请求得到的响应应当是401。


###注销

用户的注销登录的请求可以向如下的URL发送：

    /V1/tenantportal/private/tenant/logout
    
以POST方式发送该请求时，需要在请求头中携带当前你正在使用的token。

按照要求将当前使用的token放在请求头中，并发送请求后，你可能会收到如下的响应：

>挫折和离别不过是声明中的点缀  
过了多年我才读懂了家人的眼泪  
发现原来自己没有说再见的勇气  
离别的伤感，感染了满城的空气  

为了验证是否成功地登出了，我们可以用刚才使用的token去访问IoT的API。以查看工程信息为例。

你收到的来自IoT的错误响应应当如下所示，错误码为401：

>没有我以后  
一个人少喝点酒  
窗台外的衣服有没有人来收  
以后的以后  
你是谁的某某某  
若是再见只会让人更难受  

登出以后，再次访问IoT的API需要重新登录。

>如果你的帐号有可能被多点登录，那么你也不需要担心一处登录的注销操作会对其他处的登录者造成影响。如果你用你的帐号发送两次登录请求，你会发现你得到的是两个不同的token。当你使用其中的一个token发出登出请求后，另一个token仍然能够继续进行IoT的各种访问。登出操作作用的对象只是token级别的。这就保证了一处登录的注销操作不会影响到其他处的登录者。


##设备接入的权限系统

相对于租户门户的权限系统，设备接入的权限系统就要显得简单很多。

设备要接入IoT首先要租户登录到IoT平台才能获得创建设备接入凭证的相关权限。

登录完成后，可以按照如下所示的URL发送创建设备接入凭证的请求：

    /V1/iotplatform/public/credential
    
该接口需要以POST方式请求。请求体中的数据可以组织如下：

>{  
    "projectId":331,  
    "type":"device",  
    "alias":"this is a t",  
    "description":"cloud",  
    "tenantId":20,  
    "mqttPermissionLevel":"project",  
    "mqttPermission":["connection","publish"],  
    "roles":[7,8],  
    "groupName":"haGroup",  
    "clientId":"es"  
}  

别忘了请求头中带上token。
    
按上述数据的格式发送请求以后，来自IoT的响应体可能如下所示：

>{  
    "id": 192,  
    "userName": "ddec8c47-4899-49b7-b8ac-b8807c2d9564",  
    "passWord": "83KrdLkB",  
    "status": 1,  
    "projectId": 331,  
    "type": "device",  
    "createTime": "2015-12-25 09:16:59",  
    "updateTime": null,  
    "alias": "this is a t",  
    "description": "cloud",  
    "tenantId": 20,  
    "mqttPermissionLevel": "project",  
    "mqttPermission": [  
        "connection",  
        "publish"  
    ],  
    "roles": [  
        7,  
        8  
    ],  
    "groupName": "haGroup",  
    "clientId": "es"  
}  

上述IoT返回的数据中，最值得注意的是userName和passWord。请务必确保你的数据库存下了IoT返回的userName和passWord，这是IoT为你的设备自动生成的随机用户名和密码。你的设备只能带着这对用户名和密码去访问接入平台才能够进行IoT的权限验证。

现在解释一下进行请求的数据中各个属性的含义。

首先请求体中的type属性非常重要，由于这是一个为设备接入而创建的凭证，所以请务必保证该字段你填写的是*device*，而不是其他的任何字符串。否则该凭证将无法为设备接入而使用。

请求中的alias代表的是将要创建的凭证的别名，这是一个必须的属性，每一个设备接入凭证都有自己的别名。description作为你对该凭证的描述是一个可有可无的属性。接下来的三个属性projectId、groupName和clientId联合起来将确认该凭证是为哪一个已经登记在IoT上的设备而创建，即创建的设备接入凭证与IoT中登记的设备必须是一一对应的。而mqttPermissionLevel将为该凭证确认其生效级别————IoT的设备接入凭证有三个默认的生效级别，分别是*project*、*group*和*device*。如果你选择将创建的设备接入凭证的生效级别定为device，那么该凭证可以连接、发布和订阅的主题就只能精确到当前设备的主题。以上面的测试为例，如果选择的是创建device级别的凭证，那么id为192的凭证就将只拥有连接和发布到E03CA690C5A94F53A5BFEDD63FB944DD/haGroup/es主题的权限，而如果你为该凭证设置了group级别的权限，那么它的连接和发布的主题就可以只精确到E03CA690C5A94F53A5BFEDD63FB944DD/haGroup/ *的级别，以此类推，如果是project级别，那么该凭证就可以对工程下所有的主题，即E03CA690C5A94F53A5BFEDD63FB944DD/ *有连接和发布的权限。

至于mqttPermission，该属性将为凭证指定在确定的对应设备和生效级别的限制下，设备使用该凭证能够对相应的主题进行的MQTT请求的动作种类，如上例所示，id为192的凭证将只拥有连接和发布的权限。如果想让凭证同时拥有订阅的权限，则需要在该属性当中追加*subscription*的值。

---

以上就是东软物联云所使用的权限系统的详细说明，如果你想要让自己的应用程序和设备能够连接到IoT，请务必按照上述文档的说明来设计你的应用程序或设备程序使用token和设备接入凭证的方式，以便你的应用程序和设备不会因为没有权限验证而无法正常使用IoT提供的服务。

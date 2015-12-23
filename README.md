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

>/V1/tenantportal/private/tenant

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

同时，如果你细心留意一下本次请求所获得响应的相应头，你会发现本次的HTTP响应又携带了一个名为token的响应头回来。而如果你将本次返回的token与上次进行请求所使用的token进行比对的话，你会发现这两个token是不同的。

这意味着什么呢？

因为IoT会对每次生成的token加一个访问时间长度的限制，这使得每个token有效的时间是固定的。通过调用登录服务获得的token的有效时间只有15分钟，如果你在这期间没有使用该token去访问IoT的API，那么该token就会失效。如果想要继续访问IoT的API，你必须重新调用登录服务，获得一个新的token才行。而如果在这15分钟的期限内，你使用该token去完成了一次API的访问，就像上面的例子那样，你会从IoT的响应头中获得一个新的token，新的token将再为你提供15分钟的访问权限。

接着上面的例子，在成功地创建了一个工程以后，你可以向IoT请求查看这个工程的详细信息，请求的URL如下：

    /V1/tenantportal/public/project/{id}
    
由于刚才创建的工程其id是331，所以可以按照如下格式组织URL数据，请求以GET方式发送：

    /V1/tenantportal/public/project/331
    
经过上面的解释，你一定很好奇，既然新的token能够提供新开始的15分钟的访问权限。那么旧的token是否还能用呢。这里可以应用查看工程的API来验证。按照如上的URL发送请求以后，如果你使用的是旧token来提供权限的验证，那么你很有可能收到如下的响应体：

>{  
    "result": "invalid token"  
}

    HTTP状态码将会是401（Unauthorized）

这是因为IoT会在每次一个有效的token访问的时候，获得请求头中的token，并进行解析。如果请求头中的token不能够进行解析，那么说明你使用的token是无效的。无效的原因可能是你使用的并不是IoT生成的token或者这是一个IoT生成的但是已经过期的token。如果请求头中的token可以被解析，那么IoT将通过token获得你的租户信息，并判断你的租户信息是否有效。在你的租户信息有效的情况下，IoT会根据你的租户信息为你重新创建一个基于当前时间的有15分钟有效期的token，并用新的token置换掉原token。也就是说，用来进行本次请求的token在一次请求以后就不能再次访问IoT，即使访问，也不能通过IoT的权限验证，这就保证了在一个租户的登录期间，始终只有一个token是可用的。随后，新生成的token会随着你请求的API返回的应用信息，在响应头中一起返回。

作为测试，你可以使用创建工程后返回的新token来重新访问查看工程的API。以刚才的创建工程服务生成的工程为例，我们将会得到id为331的工程的信息如下：

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

当然，正如你所能想象到的那样，在成功执行完这样的一次请求以后，响应头中还会携带一个新的token。像上面描述的那样，新的token又可以提供从当前时间开始、持续15分钟的API访问权限。

>注意：  为了使你的应用能够在租户登录以后持续访问IoT，你必须设计你的程序使之在每次的IoT请求以后，更新下次的请求头中的token。否则应用程序可能需要不停地进行登录操作来获得IoT访问的权限。

###注销

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
"mqttPermisionLevel":"project",  
"mqttPermission":["mqtt:eastsoft:*:*:connection","mqtt:eastsoft:*:*:subscription","mqtt:eastsoft:*:*:publish"],  
"roles":[7,8],  
"groupName":"haGroup",  
"clientId":"es"  
}

    别忘了请求头中带上token。
    
按上述数据的格式发送请求以后，来自IoT的响应体可能如下所示：

>{  
    "id": 179,  
    "userName": "f69101b8-ee03-4fd1-b9e4-16644e2f8645",  
    "passWord": "lPnIiaxj",  
    "status": 1,  
    "projectId": 331,  
    "type": "device",  
    "createTime": "2015-12-23 09:07:53",  
    "updateTime": null,  
    "alias": "this is a t",  
    "description": "cloud",  
    "tenantId": 20,  
    "mqttPermisionLevel": "project",  
    "mqttPermission": [  
        "mqtt:eastsoft:*:*:connection",  
        "mqtt:eastsoft:*:*:subscription",  
        "mqtt:eastsoft:*:*:publish"  
    ],  
    "roles": [  
        7,  
        8  
    ],  
    "groupName": "haGroup",  
    "clientId": "es"  
}

上述数据中，最值得注意的是userName和passWord。请务必确保你的数据库存下了IoT返回的userName和passWord，这是IoT为你的设备自动生成的随机用户名和密码。你的设备只需要带着这对用户名和密码去访问接入平台就能够进行IoT的权限验证。

进行权限验证~~connection、publish


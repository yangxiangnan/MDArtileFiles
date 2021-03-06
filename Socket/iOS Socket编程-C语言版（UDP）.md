#序言

本篇文章为总结使用`C`语言的`api`来完成`UDP`通信的基本功能，如果您对`Socket`不了解，请先阅读上一篇理论知识：

* [iOS Socket理论知识](http://www.henishuo.com/ios-socket-theory/)

如果想学习TCP的，请阅读：

* [iOS Socket之C语言版TCP](http://www.henishuo.com/ios-socket-tcp-c-version/)

如果文章中有任何您认为不正确的或者有疑问的，请联系笔者！

#1. UDP Socket编程


先讲一讲`UDP`编程，因为比`TCP`要简单多了。首先，我们需要明白`UDP`是用户数据报协议，英文名为***User Datagram Protocol***，它是面向无连接的。

**注意：**`Socket`通信一定有要服务端和客户端。

##1.1 UDP Socket客户端

客户端的工作流程：首先调用`socket`函数创建一个`Socket`，然后指定服务端的`IP`地址和端口号，就可以调用`sendto`将字符串传送给服务器端，并可以调用`recvfrom`接收服务器端返回的字符串，最后关闭该socket。

笔者这里分成了四步：

* 第一步：创建`socket`并配置`socket`，如服务端`ip`地址和端口号
* 第二步：调用`sendto`发送消息到服务器端
* 第三步：调用`recvfrom`接收来自服务器端的消息
* 第四步：调用`close`关闭`socket`

###1.1.1 客户端的代码实现：

```
- (void)udpClient {
  int clientSocketId;
  ssize_t len;
  socklen_t addrlen;
  struct sockaddr_in client_sockaddr;
  char buffer[256] = "Hello, server, how are you?";
  
  // 第一步：创建Socket
  clientSocketId = socket(AF_INET, SOCK_DGRAM, 0);
  if(clientSocketId < 0) {
    NSLog(@"creat client socket fail\n");
    return;
  }
  
  addrlen = sizeof(struct sockaddr_in);
  bzero(&client_sockaddr, addrlen);
  client_sockaddr.sin_family = AF_INET;
  client_sockaddr.sin_addr.s_addr = inet_addr("192.168.1.107");
  client_sockaddr.sin_port = htons(1024);
  
  int count = 10;
  do {
    bzero(buffer, sizeof(buffer));
    sprintf(buffer, "%s", "Hello, server, how are you?");
    
    // 第二步：发送消息到服务端
    // 注意:UDP是面向无连接的，因此不用调用connect()
    // 将字符串传送给server端
   len = sendto(clientSocketId, buffer, sizeof(buffer), 0, (struct sockaddr *)&client_sockaddr, addrlen);
    
    if (len > 0) {
      NSLog(@"发送成功");
    } else {
      NSLog(@"发送失败");
    }
    
    // 第三步：接收来自服务端返回的消息
    // 接收server端返回的字符串
    bzero(buffer, sizeof(buffer));
    len = recvfrom(clientSocketId, buffer, sizeof(buffer), 0, (struct sockaddr *)&client_sockaddr, &addrlen);
    NSLog(@"receive message from server: %s", buffer);
    
    count--;
  } while (count >= 0);
  
  // 第四步：关闭socket
  // 由于是面向无连接的，消息发出处就可以了，不用管它收不收得到，发完就可以关闭了
  close(clientSocketId);
}
```

###1.1.2 客户端的打印日志

```
2015-12-06 15:38:36.095 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.286 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.286 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.291 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.291 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.296 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.296 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.316 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.317 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.324 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.324 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.328 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.329 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.339 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.339 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.355 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.356 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.366 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.366 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.372 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
2015-12-06 15:38:36.373 iOS-Socket-C-Version-Client[9709:4234848] 发送成功
2015-12-06 15:38:36.392 iOS-Socket-C-Version-Client[9709:4234848] receive message from server: Hello, server, how are you?
```

##1.2 UDP Socket服务器端

服务器端的工作流程：首先调用`socket`函数创建一个套接字，然后调用`bind`函数将其与本机地址以及一个本地端口号绑定，接收到一个客户端时，服务器显示该客户端的IP地址，并将字串返回给客户端。

笔者这里分成了五步：

* 第一步：创建`socket`并配置`socket`
* 第二步：调用`bind`绑定服务器本机ip及端口号
* 第三步：调用`recvfrom`接收来自客户端的消息
* 第四步：调用`sendto`将接收到服务器端的信息返回给客户端
* 第四步：调用`close`关闭`socket`


###1.2.1 服务器端代码实现

```
- (void)udpServer {
  int serverSockerId = -1;
  ssize_t len = -1;
  socklen_t addrlen;
  char buff[1024];
  struct sockaddr_in ser_addr;
  
  // 第一步：创建socket
  // 注意，第二个参数是SOCK_DGRAM，因为udp是数据报格式的
  serverSockerId = socket(AF_INET, SOCK_DGRAM, 0);
  
  if(serverSockerId < 0) {
    NSLog(@"Create server socket fail");
    return;
  }
  
  addrlen = sizeof(struct sockaddr_in);
  bzero(&ser_addr, addrlen);
  
  ser_addr.sin_family = AF_INET;
  ser_addr.sin_addr.s_addr = htonl(INADDR_ANY);
  ser_addr.sin_port = htons(1024);
  
  // 第二步：绑定端口号
  if(bind(serverSockerId, (struct sockaddr *)&ser_addr, addrlen) < 0) {
    NSLog(@"server connect socket fail");
    return;
  }
  
  do {
    bzero(buff, sizeof(buff));
    
    // 第三步：接收客户端的消息
    len = recvfrom(serverSockerId, buff, sizeof(buff), 0, (struct sockaddr *)&ser_addr, &addrlen);
    // 显示client端的网络地址
    NSLog(@"receive from %s\n", inet_ntoa(ser_addr.sin_addr));
    // 显示客户端发来的字符串
    NSLog(@"recevce:%s", buff);
    
    // 第四步：将接收到的客户端发来的消息，发回客户端
    // 将字串返回给client端
    sendto(serverSockerId, buff, len, 0, (struct sockaddr *)&ser_addr, addrlen);
  } while (strcmp(buff, "exit") != 0);
  
  // 第五步：关闭socket
  close(serverSockerId);
}
```

###1.2.2 服务器端的打印日志

```
2015-12-06 15:38:36.268 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.269 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.372 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.372 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.377 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.377 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.382 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.382 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.405 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.405 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.409 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.410 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.414 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.415 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.425 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.426 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.441 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.441 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.452 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.452 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
2015-12-06 15:38:36.472 iOS-Socket-C-Version-Server[39130:2473780] receive from 192.168.1.100
2015-12-06 15:38:36.473 iOS-Socket-C-Version-Server[39130:2473780] recevce:Hello, server, how are you?
```

我们这里打印出了客户端发来的消息，由于上面实现的代码中，只发10次，所以这里只有10条。

#源代码

小伙伴们，可以到github下载了：[https://github.com/CoderJackyHuang/iOS-Socket-C-Version](https://github.com/CoderJackyHuang/iOS-Socket-C-Version)

**注意**：这里面有两个工程，一个是客户端，一个是服务器端。运行时，先运行服务器端，然后再选择客户端。另外，客户端所指定的服务器端的ip地址一定要修改成您本机对应的ip，不然使用笔者这里的ip就会失败。

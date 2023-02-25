# Chromium network stack

## 引言

Chromium的网络栈代码主要位于代码库src/net下面，负责整个浏览器的网络功能，代码主要实现了DNS, http和https，以及quic和spdy等协议。另外在src/services/network/目录下面也有网络栈的代码，这部分代码是对src/net目录下代码的一个mojo封装，使之具备mojo通信的功能。

## 初始化

Chromium会在content中的utility/services.cc中初始化NetworkService，然后在network::mojom::NetworkServiceStubDispatch::Accept()方法中会调用CreateNetworkContext。

NetworkService
NetworkContext
NetworkContextParams
创建：CreateInProcessNetworkServiceOnThread

## network模块

network模块的顶层对象是network::NetworkService，

## net模块

net模块的顶层对象是net::URLRequestContext

销毁：ShutDownNetworkService()

## HTTP(S)请求流程

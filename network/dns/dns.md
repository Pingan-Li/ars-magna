# DNS(Domain Name System)

## 引言

IP地址对于用户来说难以记忆，DNS服务的出现解决了这个问题。用户可以记忆一些有意义的自主机名称（host name），而且实现了IP地址和主机的解耦，很多商业公司提供恒定的服务器主机名，即便是在IP地址发生变化的情况下。在互联网中存在不同形式的域名解析服务，但是最普遍也是最重要的一种就是使用分布式数据库系统，即我们通常提到的DNS(域名系统)

## 名称空间

DNS使用的所有名称的集合构成了命名空间(name space)，命名空间以树作为组织形式，所以又被称为是域名数。位于最顶端的树根没有命名，它的下一层由顶级域名(TLD)组成。常见的TLD包括(gTLD, ccTLD, IDN ccTLD等)。

## DNS命名语法

DNS名称树中TLD下面的名称进一步被划分成组，成为子域名。也就是说一个域名实际上由以下几个部分：

    FQDN=Hostname.Subdomain.TLD.
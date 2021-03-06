---
深入理解 HTTPS
---

> 内部分享大纲，PPT 见文末。

今天给大家带来的分享主题是：深入理解 HTTPS。

HTTPS 的产生是为了解决 HTTP 的不安全性，它包括三个方面：

1. 机密性
2. 完整性
3. 身份验证

那 HTTPS 是如何解决的呢？其实不过是在 HTTP 之下 TCP 之上加了一个 TLS 层。

TLS 层最核心的就是加密套件，如下所示，下面就以该加密套件解读 TLS：

![](https://i.loli.net/2019/12/25/lXIRgqPwK8AYWa7.png)

其中 ECDHE 密钥交换协议用于协商出对称加密所需的密钥，也就是 AES 算法，它的密钥长度为 128 位，工作模式为 GCM。非对称加密算法 RSA 用作身份验证，SHA256 是一种 MAC 算法，用于解决完整性。

以上已经解决了机密性和完整性，但是身份验证还需要 PKI 的参与。

PKI 即公钥基础设施，服务器会把它的公钥（也就是 RSA 的公钥）给 CA 机构，CA 机构会给服务器签发一个数字证书。在客户端建立连接的时候，服务器会通过 Certificate 子消息将证书链传给客户端，告诉客户端我就是你要通信的那个人。

RSA 也能作为密钥交换算法，但是因为其性能低下，需要大量指数运算消耗 CPU，同时又不具备前向安全性，所以现在基本上看不到采用 RSA 来实现密钥交换。

为了解决运算效率以及保障前向安全性，目前都会使用 ECDHE 作为密钥交换。

ECDHE = ECC + DH，即 ECC 椭圆曲线 + DHE 密钥交换。动态的 DH 每次协商都是生成不同的 DH 参数保证了前向安全性，而且 ECC 密钥长度短，运算块，但却不失安全性。

224 位的 ECC 密钥能够达到 2048 位的 RSA 的安全强度。

对称加密，加密解密使用同一个密钥，对称加密的工作原理可以简单的看成是异或运算。但是实际情况下，明文长度肯定远大于密钥长度，这就需要对明文进行拆分，每一个分组是 16 字节，采用可以采用的分组模式有 ECB、CBC 和 CTR，ECB 不能隐藏数据特征，CBC 不能并行执行，所以通常采用 CTR 工作模式。当然，如果最后一个明文块不足 16 字节，就需要对其进行填充，填充方式一般采用 PKCS#7。

了解了密码学之后，TLS 就简单的一批了。拿 WireShark 抓一下 TLS 包，分析一手，完事～

TLS 握手过程如下：

![](https://i.loli.net/2019/12/25/zIEy8hkwqCPWAJj.png)

但是每次进行 HTTPS 连接都需要 TLS 握手吧？

肯定不是，所以有了 TLS 会话机制，可以基于 Session ID 和 Session Ticket。

TLS 在 2018 年正式发布，它移除了很多已证明不安全的套件，在第一次握手时客户端把所有的密码套件及其公钥都发给了服务端，也就不再需要 Server Key Exchange 和 Client Key Exchange，从 2 RTT 到 1RR。同时，TLS 1.3 支持基于会话恢复的 0RTT 握手。

#### 参考资料

[深入浅出 HTTPS 从原理到实战](https://pan.baidu.com/s/1oFsmzXPW5EQX3wfTvg_Y9w)

密码:bt2d

[深入理解 HTTPS PPT](https://github.com/Omooo/Android-Notes/tree/master/PPT/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%20HTTPS)
# Bitsein-crypto
# 浏览器跨端实现方案

## 隐私数据说明

1. SGX加密状态

> 依托Intel SGX平台的卓绝的加密能力和基于cosmos SDK本身的开发耦合属性，获取到的数据为加密态，对于区块的msg和event使用16进制加密。

2. 底层挑战

>The integration with the TypeScript types and methods generated by the Cosmos-SDK (and chain-specific) protobuf packages.
展开来讲，例如，Event和Log在Cosmos-SDK中具有特定含义。对于钱包的活动日志，使用这些术语会非常混乱。因此，我们会选择Incident来代表钱包中的记录保存单位项目。

3. 跨平台通信

>不同协议间的状态获取，Hermit.js相较于浏览器端的js支持和移动设备端的支持相差甚远（本质上是对于数据的获取，获取到的数据都经过加密，除了用户本身之外，谁也无法知晓，Viewing-key的存在保护了链上和链下隐私）

4. secp256k1签名和ECDH的威胁

> 出于对于其他钱包的安全考虑，我们在secp256k1签名和ECDH中不推荐使用JS实现。
> 在JS中基本上利用ES字符串来存储私钥值。（例如使用字符串interning，而且会保留在内存中）。
> JavaScript执行实际上没有针对时序攻击的保护。

5.SES锁定和CPS（内容安全保护）

# 开发情况

Bitsein crypto的开发实际上不能称之为开发工作，实际上是研发，在开发者的角度来讲，我们所进行的是前无古人的工作，从基本的数据加密的实现、对于vewing-key或permit的授权许可的配置、获取经过SGX安全飞地加密后的数据监听和传入、Dapp的联调实现都是挑战，加之上述所有的工作得保持兼顾链的性能，使之不受到链上交互的影响。

> 当前使用的开发架构是Java+Springbot+Go+flutter的开发程式栈，在9月初由于一位前端技术的电脑损坏，在此基础上，又进行了代码重构，依旧使用flutter，但为了保障，又单独写了TS（是的，没有使用JS）。

我们创造性的编写了一个新的移动端通信协议SDK，命名为BLP（Bitsein Link Protocol）-Bitsein链路协议。在该协议中，原生Android系统将使用Android's app links（PWA也是如此），在iOS系统中使用Apple's Universal Links（不建议在iOS系统中使用PWA，可能会不支持），其链接过程基本可以描述为如下的步骤：

> 1. 生成RSA密钥对
> 2. 编写连接请求URL

```ts
https://link.bitsein_crypto.com/v1.0/connect
   ?ulc-success={successCallbackUrl}
   &ulc-success={errorCallbackUrl}
   &app-permissions={permissionsManifest}
   &app-pubkey={rsaPubkey}
```

> 3. 身份验证（使用BIP39）

```js
let digest = sha256(requestURL)

let word1 = wordList[ (digest >> 00) & 0x7FF ]
let word2 = wordList[ (digest >> 11) & 0x7FF ]
let word3 = wordList[ (digest >> 22) & 0x7FF ]

showUser("{word1} {word2} {word3}")
```

> 4. 发起连接请求
> 5. 处理连接响应（Error callback\Success callback)
> 6. 发起API请求

```js
https://link.bitsein_crypto.com/v1.0/api
   ?s2r-pubkey={connectionId}
   #{encryptedApiRequest}
```

> 7. 响应API请求

```js
let account = connections[s2rPubkey]
if account:
   apiResponse = parseJson(decode(encryptedApiResponse, appPrivateKey))
```

在开发中遇到最大的挑战或瓶颈在于前后端的联调、Dapp的实现（在我们的初始测试中，小型功能的Dapp连接毫无问题，但对于合约交互复杂、逻辑复杂的交互连接是出现了问题）。不容置喙的是，基于Hermit网络开发的实验性质的Dapp在PC端是毫无压力的（使用google浏览器、Firefox浏览器、edge浏览器、Safari浏览器的插件），主要在移动端的使用问题。为了让用户能够更快进行大规模的测试实验活动，我们改进当前方案，对于开源浏览器进行修改，使之能够在手机端使用keplr等插件（包括Metamask，TP插件，目前适配了3个加密货币插件，如需要更多，请联系我们）。
> 尽可放心，Bitsein crypto的开发依旧在正常进行，flutter在优化当中、前后端联调亦是如此。

# 解决燃眉之急
>
> 当前解决方案适配于Android系统，iOS暂不支持
> 使用的解决方案是：对开源浏览器进行修改，同时对于开源插件keplr进行修改，适配Hermit网络的移动端（Android）

### 1. 修改的浏览器

* Flow（隐私保护浏览器）
* Yandex（隐私加密浏览器）
* Kiwi（通用隐私浏览器）

### 2.支持的扩展

* keplr
* Metamask
* TP

### 3.测试设备

* Xiaomi
* Redmi

* [ ] huawei

* [ ] OPPO

* vivo
* honor
* 其他机型需要测试中验证

> 特别说明

1. 上述浏览器以及扩展只能在提供的安装包或扩展包中安装；
2. 由于涉及到加密市场领域，为了防止安装恶意安装包或扩展，所有的文件都配比上Sha256值进行比对
3. 在上述提及的三款浏览器的扩展中，不得安装其他扩展，防止身份令牌被窃取。
4. 上述三款浏览器，均被修改以支持扩展keplr、Metamask、TP插件，三款浏览器根据自身设备情况安装即可，可以三个都安装，可以只安装一个浏览器。
5. 上述三款浏览器不会收集、上传用户的任何隐私信息（例如用户的钱包地址、私钥等），这些浏览器无中心化服务器控制，无后台程序。
6. 建议浏览器只用于Hermit网络，其他网络请按照您之前的使用习惯。
7. 关于Bitsein crypto，静待佳音，我们正在突破，相信不久将发布第二个测试版本。

## 操作步骤（以Flow为例）
>
> 其他的浏览器操作步骤一致，不赘述

1. 步骤1-打开浏览器-打开右上角菜单
![Alt text](image-3.png)
2. 步骤2-找到设置-点击打开设置页面
![Alt text](image-4.png)
3. 步骤3-滑到底部，找到关于Flow-应用版本（点击三次）进入开发者模式
![Alt text](image-5.png)
4. 步骤4-回到设置页面-点击扩展程序
![Alt text](image-6.png)
5. 步骤5-打开扩展程序页面-打开开发者模式（如果步骤3没有成功，这里看不到开发者模式按钮开关）
![Alt text](image-7.png)
6. 步骤6-点击+（from .zip/.crx/.user.js),选择从文件加载，将你所需要的插件加载上去即可。
![Alt text](image-8.png)
7. 步骤7-开始使用钱包插件，如果安装成功，你将在如下页面看见它（在你不需要的时候可以选择关闭插件）
![Alt text](image-9.png)

## 浏览器及钱包扩展包

1. Yandex

```js

2. Flow
3. Kiwi

# DIMP（去中心化即时通讯协议）技术白皮书 (V0.1)

草案：**2018年11月11日** (@moky)

**摘要:** 本文档引入一种新的为分布式即时通讯而设计的协议，以及一个为开发分布式即时通讯应用而设计的技术架构。该软件提供了一个账号系统（用户身份认证）以及在各账号之间通过端对端加密实现的安全通讯服务。

Copyright &copy; 2018 Albert Moky

## 0. 背景
即时通讯（IM）是目前 Internet 上最为流行的通讯方式，它允许网络上的两个人或多人使用网络实时的传递文字消息、文件、语音与视频交流。
从用户使用场景角度，目前市场上的IM工具可划分为**企业级IM工具**（如钉钉、企业微信、RTX 等）和**个人IM工具**（如 WeChat、QQ、Facebook、Whatsapp、Telegram 等）两大类。

而当前所有最流行的 IM 工具其平台架构设计都是中心化的，每个工具背后都是一套自有的账号认证系统以及消息转发系统。这将造成至少以下3个难以解决或避免的关键性问题：

1. 每个用户的账号密码以及关系链等重要资料都被绑定在特定的平台上，其他平台无法共享；
2. 每个平台需要中心化的数据库来存储用户账号密码等资料，这些中心点很容易成为黑客攻击的目标；
3. 用户完全依赖于其所在的 IM 平台处理能力，一旦出现中心故障将可能导致无法登录与通讯。

问题1的主要症结在于这种中心化机制不利于**持续创新**，容易形成**寡头垄断**市场。而一旦垄断形成，即使寡头丧失创新能力，其用户也会由于迁移成本过高而必须继续忍受其落后的通讯方式，难以享受其他开发者提供的更优质的服务。

同时由于各家平台账号难以互通，即使寡头尚未完全形成，几家头部企业的用户相互之间也无法通讯。从而导致每个用户需要同时安装多个客户端并频繁切换的不良体验。

更进一步地，由于用户需求的多样性，很难在一款 IM 工具上承载所有的用户需求特性，而同时开发和维护多款 IM 工具显然超出了一家企业的能力范围（无论这家企业多么庞大）。

问题2的主要症结在于**信息安全**。由于中心化的账号系统设计，给黑客创造了大批量窃取用户资料的有利条件。

问题3的重点是**单点故障**风险，平台需要投入巨大资源以确保其网络（尤其是中心节点）正常运行。而一旦由于某些原因导致该平台停止服务，用户除了等待平台自行修复，在有限的时间内几乎可以说是毫无选择的主动权。

在中心化 IM 技术已发展完备的今天，以上问题（或隐患）一直未能彻底消除，所以市场需要一种全新的去中心化的 IM 技术来满足更安全更丰富的通讯需求。

## 1. 去中心化 IM 工具的需求
IM 工具的竞争，是当今互联网世界最激烈的竞争领域之一，且已基本形成寡头垄断态势。为了在这个已被深耕多年的领域生存和发展，基于去中心化设计的 IM 工具必须要满足以下的要求，才能解决中心化 IM 所不能解决的那些问题，进而赢得广泛的应用。

### 去中心化的用户身份认证技术
去中心化 IM，首先要解决的就是去中心化用户身份认证技术。现有的用户身份认证系统，依靠的是简单的 ID+Password 配对技术，也即需要一个可靠的数据库去保存配对信息，因此中心化不可避免。

去中心化的用户身份认证系统，需要从算法上保证无需中心化数据库来保存配对信息，仅仅基于**共识算法**就可以实现身份认证。

### 基于端对端加密的安全通讯技术
中心化 IM 由于其服务提供者（SP）的唯一性，所有的信息安全问题都必须且只能由 SP 负责解决，用户只能选择信任 SP。而去中心化的 IM 系统中，任何人都可以成为 SP 为他人提供服务，因此我们不能完全信任任何一个 SP，从而必须在算法上保证在所有网络节点都不足信的前提之下，依然能够放心的进行通讯。

因此，基于端对端加密的通讯技术成为了必选。

### 高并发（支持亿级用户）
目前所有头部的 IM 服务提供商都拥有**亿级**以上的庞大用户群体，如国内的微信、QQ 月活跃用户已近10亿，而国外的 Facebook 更是高达20亿。因此一个可以处理极其庞大用户的高并发系统设计对于去中心化 IM 技术是至关重要的。

### 低延时
一个好的 IM 软件，需要有秒级的送达能力，才能保证**实时通讯**的良好体验。高延时的系统设计甚至不能称之为 IM 技术。

### 免费使用
一个可以免费供普通用户使用的 IM 软件或许将会在现有的 IM 领域中得到更为广泛的应用。

### 可定制、可升级
一个可根据不同人群通过定制、升级来扩展高级功能的 IM 平台，将会得到更多的开发者支持，从而衍生出更丰富的产品特性，满足更多的用户需求。

## 2. 共识
DIMP 引入了3类信息的共识机制：身份确认算法、关系维护机制、消息格式（含加密与校验算法），以保证整个系统稳定可靠地运行。

### 身份确认
首先，用户账号 ID 通过一个我称之为“元”（Meta）的数据结构来生成，该生成算法确保了用户的 ID 与其非对称密钥对（PK+SK）之间的关联关系。

ID 是一个格式为 ```name@address[/terminal]``` 的字符串，主要包含 name 和 address 两个字段，另外一个可选字段 terminal 用于标明该 ID 当前登录设备，以区分多终端登录的情形：

1. name - 账号名，非唯一；
2. address - 由 Meta 算法计算得到的地址，在当前算力下可保证全网唯一性；
3. terminal - 登录点名称（可选项），仅用于表示同一个 ID 在不同地方登录。

Meta 信息是一个数据结构，包含以下 4 个字段：

1. version - 指定 Meta 算法版本，当前为 1；
2. seed - 用于生成指纹的种子，即用户指定的账号名 **Name**；
3. key - 用户的公钥信息；
4. fingerprint - 指纹信息，由用户私钥对种子 seed 的签名信息进行 base64 编码而得。

```
/* Meta 信息实例，对应 ID 实例为 "hulk@4bejC3UratNYGoRagiw8Lj9xJrx8bq6nnN" */
{
    // 元算法版本号
    version     : 0x01,
    // 信息种子，用于生成指纹信息，同时用作 ID.name
    seed        : "hulk",
    // 用户公钥 PK
    key         : {
        // 公钥算法名称
        algorithm : "RSA",
        // 公钥数据
        data      : "-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDBUyaQPTvXgTfYC7bSAIhC3efc\nQT7HEX9PzJXQs9XeuxY4iBBBnrUPkJhOvwHrnAErBnM6tm9I45htcTeVOsi/qRbs\nXpQ6u7JuBayxgVp2vU0xUWDKLTlE9VT3F/OgT1xGuXnMO5TJnt/HjlbASToGUxBa\nrMWCrjQJX2UitMaU+wIDAQAB\n-----END PUBLIC KEY-----"
        // 其他公钥参数
    },
    // 指纹信息，用于生成 ID.address
    fingerprint : "SdxF0IU8Kq9sfD/x46uRC4W8VJ4WmVF8j0Je6ZMIURLoFju/SFEtSC41ibU7R6cgINfUpZ4QCfVpo0+rHwlXBNeyZS5vqf1+fvMuISucRWGjmFmusTAqtqN0RCDvhdkeaxuQyMJKAGlzkcm5CXeqWyijDOQOZyf2pGJlfs18e2c="
}
```

当一个节点或客户端从其他节点获得一个宣称与某 ID 对应的 Meta 信息时，可以根据共识算法自行校验。如果校验通过，则确认 Meta 信息中包含的公钥（PK）信息合法，并加入本地的数据库中。由于 Meta 算法的确定性，任何一个节点或客户端都可以判断 ID & PK 的对应关系是否合法，无需第三方机构证明。

ID 的地址生成算法参考了 BitCoin 的地址生成算法，并做了一点微小的升级，使其能包含一个人类可读的 **name** 信息，同时还增加了一个更有利于搜索账号（而不是只能复制粘贴地址）的 **number** 属性。相信以上两个扩展将会令其作为 IM 账号更加友好并更容易推广。

### 关系维护
对于“群组”（Group）等包含关系网络的信息，我们可以采用类似于区块链的共识机制，通过签名+投票的形式实现关系维护和演变。

### 消息加密与校验
DIMP 制定了统一的消息格式与加密/校验机制。每一条信息在发到网络中之前都必须事先进行加密和签名，从而确保任何中间节点不能窃听或篡改其中的内容。

## 3. 账号
DIMP 账号算法是 BitCoin 地址算法的升级版，加入了 **name** 和 **search number** 的概念。

DIMP 的账号（ID）主要包含 name 和 address 两部分，另外的 terminal 为扩展字段，为多终端登录扩展预留。其中 name 为用户指定的字符串，命名规则约定如下：

### Name

1. 长度大于 1，且小于 20 字节；
2. 应由英文字母（区分大小写）、数字0-9、下横线“_”、横线“-”、小数点“.”等组成，不包括空格以及其他特殊字符；
3. 不能包含保留字符“@”与“/”。

其中第1条和第3条是由算法本身决定的，第2条则是出于通用性的考虑所做的建议性限制，各客户端也可以酌情考虑允许用户输入其他语言文字，不过本人认为没有这个必要，建议客户端需要显示的包含其他语言的用户名字信息以 profile 方式提供。

### Address

DIMP 的账号地址由 Meta 算法生成，算法如下：

```
// 1. 用用户私钥 SK 对种子 seed 进行签名生成指纹信息
meta.version     = 0x01;
meta.seed        = "moky";
meta.key         = PK;
meta.fingerprint = sign(meta.seed, SK);

// 2. 对指纹信息进行组合哈希运算（sha256 + ripemd160）得到指纹哈希 hash
// 3. 将 Network ID 与指纹哈希 hash 拼接后再双哈希（sha256 + sha256）
//    然后取其结果前 4 个字节作为校验码
// 4. 将 Network ID、指纹哈希 hash、校验码 check_code 三者拼接后
//    再 base58 编码即得到账号地址
function btcBuildAddress(fingerprint, network) {
    hash        = ripemd160(sha256(fingerprint));
    check_code  = sha256(sha256(network + hash)).prefix(4);
    address     = base58(network + hash + check_code);
}

// 将 name 与 address 组合，即得到 ID （字符串格式：“name@address”）
ID.name    = meta.seed;
ID.address = btcBuildAddress(meta.fingerprint, network);
```

校验算法如下：

```
function isMatch(ID, meta) {
    // 1. 首先检查 Meta 信息中的 seed、key、fingerprint 与 ID.name 是否对应
    if (meta.seed != ID.name) {
        return false;
    }
    if (verify(meta.seed, meta.fingerprint, meta.key)) {
        return false;
    }
    
    // 2. 再由 Meta 算法生成其对应的地址，检查是否与 ID.address 相同
    address = btcBuildAddress(meta.fingerprint, ID.address.network);
    if (address != ID.address) {
        return false;
    }
    
    // 3. 以上全部通过，则表示匹配成功，可以接受 meta 中的 key 作为该账号的公钥
    ID.publicKey = meta.key;
    return true;
}
```

### Search Number
由于 Address 字段对人类记忆不友好，所以通常需要用直接复制 ID 字符串的方式来添加好友，无法像手机号码、QQ号码那样口头传播，因此我引入了**账户号码**的概念。

在此约定所有开发者在实现时采用 ID.address 的**校验码**（4字节）作为该账号的 **Search Number** 以方便用户口头传播（其形如 **123-456-7890**，类似于电话号码，容易记忆，但不保证绝对唯一性）。

该字段的样本空间大小为 **4,294,967,296**，因此当网络中的用户数达到了数千万级以上之后，每个账号的 Search Number 相同的概率将会超过 1%（尤其是“靓号”）。当匹配到多个用户 ID 时，需要用户通过 ID.name 等其他信息分辨真正的好友 ID。

## 4. 群（Group）
超过2人参与的聊天谓之“群”（Group）。根据人数多少和存续时间长短可以采用不同的实现方式。

确认群指令的有效性，采用 POP（权限证明）机制，即根据共识约定每个角色的权限，然后每一项操作必须由该角色成员签名方为有效。

### 4.1. Polylogue - 多人会话（临时讨论组）
对于少数人参与的群（例如少于 100 人），可以采用“临时讨论组”方式实现。

Polylogue 是一个由客户端维护的**虚拟群**，任何一个群成员邀请其他用户时，遵循向当前所有成员群发 **invite** 指令的规则；当某位成员希望退出该群，则群发 **quit** 指令；仅群创建者（群主）可以驱逐成员，通过群发 **expel** 指令实现。

每个用户需要用自己的私钥对指令进行**签名**，每个客户端接收到一条群指令后，需验证其身份与签名，验证通过后，如实记录在本地数据库中。

#### 建群操作
首先通过 ID 生成算法得到群 ID：

```
// 生成元信息
meta.version     = 0x01;
meta.seed        = "Polylogue-1234567890"; // 随机字符串
meta.key         = founder.PK;
meta.fingerprint = sign(meta.seed, founder.SK);

// 生成 ID
ID.name    = meta.seed;
ID.address = btcBuildAddress(meta.fingerprint, MKMNetwork_Polylogue);
```
然后将群 ID 发给初始成员。每位成员收到后，在本地保存为一个**临时会话**记录。
后续所有通讯都必须在 message.content.group 中夹带群 ID，以便接收方知道这是一个群消息。

#### 添加群成员
生成 **invite** 指令：

```
{
    type    : 0x88,      // DIMMessageType_Command
    sn      : 794594362,
    group   : "Polylogue-1234567890@7jA5JmnrsM46hcynsKdcns47d4fYwpbbn7",
    
    command : "invite",
    member  : "hulk@4bejC3UratNYGoRagiw8Lj9xJrx8bq6nnN"
}
```

然后加密、签名，得到 Certified Message（参照“消息”章节）。再发送给全部已有成员即可。

#### 退群
如上，```command```字段为```quit```。

#### 驱逐成员
如上，```command```字段为```expel```，且仅限群主操作。

### 4.2. Chatroom - 聊天室（固定群/大规模群聊天）
对于参与人数较多的群（例如 100 人以上），由于需要添加**管理员**角色（Administrator）以协助管理，并且需要允许转让**群主**（Owner）身份，相对比较复杂，所以我这里建议使用**区块链**技术来记录**群历史**信息。

具体实现方式是每个群一条**链**，打包区块的权限采用 **POP**（Proof Of Permission）机制，即通过共识定义群角色的权限，然后由角色成员签名打包数据。

*（DIM 项目第一阶段由于使用人数较少，暂时无需实现这部分功能，建议群聊天都采用 Polylogue 方式）。*

## 5. 消息
当一个用户需要发送信息时，客户端需要先执行以下两步操作，再将计算结果发送到 DIM 网络中：

1. 用接收方的公钥对消息体进行加密（为提高效率，可以先使用对称密码 PW 对消息体进行加密，然后用接收方公钥对 PW 进行加密）得到密文；
2. 用发送方的私钥对密文进行签名（先求取密文的摘要，再对摘要进行签名）。

对应地，信息的接收也需要两个对应的步骤：

1. 用发送方的公钥对密文和签名进行校验（先求得密文的摘要，再对摘要进行校验）；
2. 用接收方的私钥对密文进行解密（先解密得到对称密码 PW，再用 PW 对密文进行解密还原消息体）。

通过以上算法，既能确保信息不被任何中间节点窃取，也能防止第三方篡改，从而实现安全可靠的去中心化通讯。

### 消息头（Envelope）
每一个消息都包含3个字段作为消息头：

1. sender - 发送方 ID
2. receiver - 接收方 ID
3. time - 发送时间

### 消息内容（Content）
每一份消息内容都包含2个公共字段以区分不同的消息：

1. type - 消息类型（text、file、image、audio、video、webpage、command 等）
2. sn - 消息序列号（由发送方客户端随机生成的数字，以唯一标识具体某个信息）

下面列举几个不同类型的消息格式：

#### 0x01 - 文本消息

```
{
    type : 0x01, // DIMMessageType_Text
    sn   : 1234,
    
    text : "Hey guy!"
}
```

#### 0x10 - 文件消息

```
{
    type : 0x10, // DIMMessageType_File
    sn   : 1234,
    
    URL      : "http://...", // encrypt & upload to CDN
    filename : "dir.zip"
}
```

#### 0x12 - 图片消息

```
{
    type : 0x12, // DIMMessageType_Image
    sn   : 1234,
    
    URL      : "http://...",    // encrypt & upload to CDN
    snapshot : "BASE64_ENCODE", // base64(smallImage)
    filename : "photo.png"
}
```

#### 0x14 - 语音消息

```
{
    type : 0x14, // DIMMessageType_Audio
    sn   : 1234,
    
    URL  : "http://...", // encrypt & upload to CDN
    text : "ASR_TEXT"    // Automatic Speech Recognition
}
```

#### 0x16 - 视频消息

```
{
    type : 0x16, // DIMMessageType_Video
    sn   : 1234,
    
    URL      : "http://...",   // encrypt & upload to CDN
    snapshot : "BASE64_ENCODE" // base64(smallImage)
}
```

#### 0x20 - 网页消息

```
{
    type : 0x20, // DIMMessageType_Page
    sn   : 1234,
    
    URL   : "http://...",   // Web Page URL
    icon  : "BASE64_ENCODE" // base64(icon)
    title : "...",
    desc  : "..."
}
```

#### 0x37 - 引用回复

```
{
    type : 0x37, // DIMMessageType_Quote
    sn   : 5678,
    
    quote : 1234, // referenced serial number of previous message
    text  : "I like it!"
}
```

#### 0x88 - 系统命令

```
{
    type : 0x88, // DIMMessageType_Command
    sn   : 1234,
    
    command : "...", // command name
    params  : ...    // extra parameters
}
```

#### 0xFF - 转发消息
此类消息通常用于 DIM 网络环境极度不可信、消息发送者希望隐藏消息路径的使用场景。其中```forward```字段包含的是需要转发的实际消息包（已加密+已签名）。实施过程如下：

1. 发送方先创建一个包含实际消息内容的**待发送信息包**（加密+签名）；
2. 发送方将整个实际消息包进行二次打包，生成一个发给提供转发服务的 Station 的消息（receiver 为 **Station ID**，消息类型为 **DIMMessageType_Forward**）；
2. 该 Station 收到后，先验证并用自己的私钥解密，判断消息类型是否为 DIMMessageType_Forward；
3. 如果是，则读取```forward```的信息头，获取**实际接收方 ID**，并将```forward```重新打包，生成一个由 Station 发给实际接收方的消息包并发送到 DIM 网络中（sender 为 Station ID，receiver 为实际接收方 ID，消息类型为 DIMMessageType_Forward）；
5. 实际 receiver 收到数据包后，解开并判断消息类型，如果是 **DIMMessageType_Forward** 则对```forward``内容**递归执行解包操作**，直到获得最终的真实消息内容。

```
{
    type : 0xFF, // DIMMessageType_Forward
    sn   : 5678,
    
    forward : { // top-secret message
        sender   : "moki@4LrJHfGgDD6Ui3rWbPtftFabmN8damzRsi",
        receiver : "hulk@4bejC3UratNYGoRagiw8Lj9xJrx8bq6nnN",
        time     : 1542075610,
        
        data      : "BASE64_ENCODE", // secret message content
        key       : "BASE64_ENCODE", // encrypt(PW, receiver.PK)
        signature : "BASE64_ENCODE"  // sign(hash(data), sender.SK)
    }
}
```

### 原始信息（Instant Message）
发送方发出的信息（打包处理前）、接收方收到后（解包处理后）的信息，格式如下：

1. sender - 发送方 ID
2. receiver - 接收方 ID
3. time - 发送时间
4. content - 消息内容（明文）

### 加密信息（Secure Message）
客户端发出信息前，先对原始信息进行加密，得到格式如下（content 字段替换成了 data）：

1. sender - 发送方 ID
2. receiver - 接收方 ID
3. time - 发送时间
4. data - 加密数据（用一个随机密码对 content 进行对称加密）
5. key - 对称密码信息（用接收方公钥加密，因此只有接收方私钥能够解密）

加密算法如下：

```
string = json(content);            // 1. 先将 content 序列化为一个字符串
PW     = random();                 // 2. 生成随机密码（对称加密密钥）
data   = encrypt(string, PW);      // 3. 对序列化字符串进行加密并替换
key    = encrypt(PW, receiver.PK); // 4. 对随机密码进行非对称加密
```

### 签名信息（Certified Message）
对原始信息加密后，再用发送方私钥进行签名，得到格式如下：

1. sender - 发送方 ID
2. receiver - 接收方 ID
3. time - 发送时间
4. data - 加密数据
5. key - 对称密码信息
6. signature - 签名信息

签名算法如下：

```
digest    = sha256(sha256(data));    // 1. 先计算 data 的摘要信息
signature = sign(digest, sender.SK); // 2. 再对摘要信息进行签名
```

## 6. 扩展
## 7. 结论
祝帝企鹅20周岁生日快乐！🎂
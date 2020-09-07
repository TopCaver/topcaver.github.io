---
title: Google密码检测：Google不知道你的用户名密码，但它能告诉你的用户名密码已经泄漏了

tags: Google PasswordCheckup 密码 同态加密 PSI 不经意传输
--- 

译自Google Security Blog，[《Protect your accounts from data breaches with Password Checkup 》](https://security.googleblog.com/2019/02/protect-your-accounts-from-data.html)  

> February 5, 2019
> Posted by Jennifer Pullman, Kurt Thomas, and Elie Bursztein, Security and Anti-abuse research
> Update (Feb 6): We have updated the post to clarify a protocol used in the design is centered around private set intersection.

文章简明扼要的介绍了Google的密码保护是如何设计的，特别是关于隐私保护集合交集(Private Set Intersection, PSI) 应用协议在检查密码泄漏数据的细节。

**翻译作者：**
[@TopCaver](https://github.com/TopCaver)

**特别声明：**
限于译者的英语水平和技术能力，翻译中必然存在错误，请及时阅读上述链接原文。

<!--more-->

[TOC]

-----
Google通过[预防、检测和缓解](https://www.usenix.org/node/208154)等一系列措施，保护你的账户安全。因此，我们会重置那些受到[第三方数据泄漏](https://security.googleblog.com/2014/09/cleaning-up-after-password-dumps.html)影响等账户的密码。这个策略仅在过去的两年中，就已经帮助我们保护超过了1.1亿用户。如果没有这些安全措施，用户遭受账户挟持的[风险将扩大10倍](https://security.googleblog.com/2017/11/new-research-understanding-root-cause.html)。现在我们希望保护你所有的Web账号，而不仅仅只是Google账号。这就是[Password Checkup](https://chrome.google.com/webstore/detail/password-checkup/pncabnpcffmalkkjpajodfhijclecjno)这个Chrome扩展的作用（这个扩展现在已经合入Chrome，是Chrome的一个功能）。无论何时，你登录一个网站，如果你使用的用户名和密码是已经泄露的，Password Checkup将会弹出告警。Password Checkup和斯坦福大学的密码学家共同设计，确保Google从不获取你的用户名和密码，并且确保已经泄露的数据不会造成更大的危害。自从Password Checkup早期实验开始，我们就一直分享技术细节，透明的阐述我们是如何保护您的数据安全的。

![Image](/illustration/password-checkup-extension-how-to.png)

## 关键设计准则
我们设计Password Checkup的三个关键准则：

-  **告警可以被处理，而非仅提示:**  我们认为告警应该提供简明准确的安全建议。对于不安全的账户，这意味着重置密码。数据泄露也会包含其他信息，比如个人电话或者是邮件地址，但是这些信息泄露没有办法重新保护它们。因此，我们只专注于提示您不安全的用户名和密码。


-  **隐私是设计的核心: ** 您的用户名和密码是非常敏感的数据。我们设计Password Checkup采用隐私保护技术，确保您的个人信息不会披露给Google。同时，我们也防范攻击者滥用Password Checkup从而获取已经泄漏的用户名和密码。最后， 扩展报告的统计信息都是匿名的。这些指标包括显示不安全凭据的查询次数，告警后是否修改了密码，web需要的用于改进站点兼容性的统计。

-  **避免重复告警: ** Password Checkup被设计成仅当账户所有的信息都落入攻击者手中时，才产生告警。我们不会为您设置过期的密码，或者单纯的弱密码（例如 123456）。只有当用户名和密码同时出现泄漏的时候，才会产生告警，因为这时风险巨大。


## 解决方案
总体来说，Password Checkup需要向Google请求用户名和密码的泄漏状态，但是又不能向Google提供查询的信息。同时，我们也要确认在这个过程中，也没有不安全的用户名和密码再次发生泄漏，也不能进行暴力猜解。Password Checkup通过使用多轮HASH，k-anonymity和隐私数据集合求交（PSI：rivate set intersection）
我们的设计，在隐私、计算量和网络延迟之间取得了平衡。尽管[单方隐私信息检索（single-party private information retrieval，PIR）](https://crypto.stanford.edu/~dabo/courses/cs355_fall07/pir.pdf) 和 [1/N不经意传输（1-out-of-N oblivious transfer）](https://en.wikipedia.org/wiki/Oblivious_transfer) 实现了一些我们的需求，但是40亿记录的数据库所涉及的通信开销非常棘手。另外[多方PIR（k-party PIR）](https://crypto.stanford.edu/~dabo/courses/cs355_fall07/pir.pdf) 和硬件安全区提供有效的替代方案，但是它需要的user trust in schemes在实践中还没有被广泛使用。多方PIR还存在共谋风险；硬件安全区计算也有[硬件漏洞](https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-van_bulck.pdf)和侧信道攻击的风险。

## 方案概览
这是Password Checkup如何保护你的账户，并且满足我们对安全和隐私的要求。

![Image](/illustration/password_checkup_revised.png)

### 方案说明（此部分非google原文，根据上图的大意解释）

1. 在第一步中，google会发现已经泄漏的用户名和密码。这些用户名和密码，通过Argon2 Hash算法取Hash，HASH的前16位作为数据库分区索引明文保存，全部HASH使用Google的密钥加密。
2. 然后当你在登录页面输入用户名和密码之后，Password Checkup将使用Argon2 Hash计算用户名密码，并且使用只有你知道的密钥加密HASH。
3. Password Check将根据前16bit的HASH，从google拉取一个数据分区。使用用户密钥加密过的数据，将发送给Google。Google将用Google的密钥加密，然后发还给用户。此时用户会得到一个双重密钥加密的密文。用户使用自己的密钥解密，得到一个只有Google密钥加密的数据。
4. Password Check在本地环境搜索Google加密过的数据库分区，以及Google加密过的用户名密码，如果有匹配的记录，说明这一用户名和密码已经泄漏，您应该立刻修改密码。

## 保护你的账户
[Password Checkup](https://chrome.google.com/webstore/detail/password-checkup/pncabnpcffmalkkjpajodfhijclecjno)现在已经是一个Chrome的扩展。由于这是第一个版本，因此我们会在未来几个月继续对其进行完善，包括改善网站兼容性以及用户名和密码字段的识别。
（目前已经是Chrome的一部分了）。


## 致谢
This post reflects the work of a large group of Google engineers, research scientists, and others including: Niti Arora, Jacob Barrett, Borbala Benko, Alan Butler, Abhi Chaudhuri, Oxana Comanescu, Sunny Consolvo, Michael Dedrick, Kyler Emig, Mihaela Ion, Ilona Gaweda, Luca Invernizzi, Jozef Janovský, Yu Jiang, Patrick Gage Kelly, Nirdhar Khazanie, Guemmy Kim, Ben Kreuter, Valentina Lapteva, Maija Marincenko, Grzegorz Milka, Angelika Moscicki, Julia Nalven, Yuan Niu, Sarvar Patel, Tadek Pietraszek, Ganbayar Puntsagdash, Ananth Raghunathan, Juri Ranieri, Mark Risher, Masaru Sato, Karn Seth, Juho Snellman, Eduardo Tejada, Tu Tsao, Andy Wen, Kevin Yeo, Moti Yung, and Ali Zand.

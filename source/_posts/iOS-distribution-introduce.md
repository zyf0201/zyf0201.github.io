---
title: iOS分发介绍
date: 2021-10-21 14:56:01
tags: iOS
---

> 随着互联网技术的普及，越来越多的公司通过移动app来开展线上应用，同时随之而来就是设备的管理问题。苹果公司有很多终端产品，包括iPhone、Mac、iWatch、iPad等，同时还有很多app供用户下载使用，普通用户主要通过 App Store下载应用，除了App Store以外，苹果公司对于企业客户还专门推出了Apple商务管理计划，这篇文章会进行简单的介绍。

# Apple分发简介

目前Apple主流的四种分发方式：

- [App Store](https://developer.apple.com/cn/app-store/)

> 作为用户探索和下载 app 的好去处，App Store 不仅安全可靠、值得信赖；它更提供了巨大的商机，让开发者能在 175 个地区提供面向 iPhone、iPad、Mac、Apple TV 及 Apple Watch 的 app 和服务。查看文章、指南及其他资源，从而帮助您设计出精彩绝伦的 app，触及更多用户，并不断发展您的业务。

普通消费者主要通过App Store下载应用，或者也可以通过testflight将人员加入到测试名单进行测试分发，中国区apple id可以下载本区的的应用。

- Mac App Store

> 在 Mac App Store 中，客户能够简单容易地发现、购买和下载您的 app，并能轻松保持它们为最新状态。Mac App Store 会按照用户在 Mac 上喜欢做的事来归类整理各个 app，并提供深度报道、精选集和视频，以此出色展示您的 app，提升其曝光度。

适用于mac用户，或者可以直接下载安装包进行安装

- 分发自定App

> 与商务和教育机构客户接洽，根据他们所在组织的独特需求，为其设计和构建自定 app。通过 Apple 商务管理(简称ABM)和 Apple 校园教务管理，您不但能以安全私密的方式向特定的合作伙伴、客户和特许经营者进行分发，而且还能向内部员工分发专属 app。

这类自定App主要是企业内部app，比如企业内部特定的办公软件，不对公众开放.针对企业内部的应用苹果主推的Apple商务管理分发(ABM)，客户使用ABM分发首先需要申请苹果企业账号，然后申请ABM证书。企业用户可以通过以兑换码的方式下载应用，具体操作请参考[在 Apple 商务管理中选择和购买内容](https://support.apple.com/zh-cn/guide/apple-business-manager/asmc21817890/web)。企业管理内部应用的实践可以参考：[企业管理苹果设备的正确姿势 — ABM](https://blogs.vmware.com/china/2019/10/08/%E4%BC%81%E4%B8%9A%E9%87%87%E8%B4%AD%E8%8B%B9%E6%9E%9C%E8%AE%BE%E5%A4%87%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF-abm/)

Apple商务管理(ABM)介绍：
组织可以注册为 Apple 商务用户，以使用 Apple 商务管理来购买和分发内容，以及自动完成设备部署。您指定的组织可以在 Apple 商务管理的“内容”部分中看到和购买您的 app，并能通过移动设备管理对其进行无缝分发。或者，组织可以选择向授权用户提供兑换码，供他们从 App Store 下载相应的 app。

Apple 校园教务管理(ASM)介绍：
根据教育机构的个别需求，以非公开的方式向对方提供专为其量身定制的 app。您在 App Store Connect 中指定的组织将能够看到该 app 并在 Apple 校园教务管理中进行批量购买。通过 Apple 校园教务管理服务，教育机构可以购买内容、配置自动设备注册，并为学生和教职人员创建帐户。教育机构也可以使用这一功能来分发专属 app，供内部使用。

- Safari浏览器扩展

> 使用 Safari 扩展增强和自定义 Mac、iPhone 和 iPad 上的 Web 浏览体验。使用强大的原生 API 和框架，以及熟悉的 Web 技术（例如 HTML、CSS 和 JavaScript），您可以轻松地在 Xcode 中创建 Safari 扩展，并将它们分发到 App Store 的扩展类别中，或者让它们经过公证以便在 Mac 之外分发应用商店。 Xcode 12 及更高版本支持流行的 WebExtension API，并包含一个移植工具，可以轻松将您的扩展程序带到 Safari。

# Apple分发方式介绍

- App Store
通过 App Store Connect，您可以向 App Store 提交 App，审核通过后，然后上架App Store后供用户下载

- TestFlight
通过 App Store Connect，您可以向 App Store 提交 App，然后在testflight添加测试人员进行测试，测试人员需要通过testflight下载对应的应用

- In-house
使用In-house证书打包的iOS可以直接安装，但是目前官方已经关闭了In-house证书的申请途径

- 超级签名
- ABM
Apple 商务管理是一个面向 IT 管理员的基于网页的简单门户，可提供一种快速、简便的方法来部署您所在组织的 Apple 设备（指直接从 Apple 购买或从加入该计划的 Apple 授权经销商或运营商处购买的设备）。您可以在移动设备管理 (MDM) 解决方案中自动注册设备，而无需在用户拿到设备前实际接触或设置设备。

- ASM

以下是各个分发方式的区别：
{% asset_img ios-distribute.png 分发方式区别 %}


# 参考文档

[iOS应用发布方式盘点+苹果商务详解](https://www.jianshu.com/p/c8361a83a338)
[什么是 Apple 商务管理？](https://support.apple.com/zh-cn/guide/apple-business-manager/apdd344cdd9d/1/web/1)
[企业管理苹果设备的正确姿势 — ABM](https://blogs.vmware.com/china/2019/10/08/%E4%BC%81%E4%B8%9A%E9%87%87%E8%B4%AD%E8%8B%B9%E6%9E%9C%E8%AE%BE%E5%A4%87%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF-abm/)
[什么是MDM](https://www.manageengine.cn/mobile-device-management/what-is-mdm.html)
[iOS MDM详解](https://juejin.cn/post/6844903969160986632)


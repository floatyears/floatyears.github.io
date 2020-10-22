---
title: iOS提审踩坑记
date: 2020-10-22
categories: Program
tags: [Program]
---

最近这段时间，因为某些不可抗力，iOS提审失败，被逼做马甲包。

#### 1、送审被拒历史：
>App Store Connect
>
>Dear Developer,
>
>We identified one or more issues with a recent delivery for your app, "xxxx" . Please correct the following issues, then upload again.
>
>ITMS-90809: Deprecated API Usage - New apps that use UIWebView are no longer accepted. Instead, use WKWebView for improved security and reliability. Learn more (https://developer.apple.com/>documentation/uikit/uiwebview).
>
>Though you are not required to fix the following issues, we wanted to make you aware of them:
>
>ITMS-90683: Missing Purpose String in Info.plist - Your app's code references one or more APIs that access sensitive user data. The app's Info.plist file should contain a NSLocationWhenInUseUsageDescription key with a user-facing purpose string explaining clearly and completely why your app needs the data. Starting Spring 2019, all apps submitted to the App Store that access user data are required to include a purpose string. If you're using external libraries or SDKs, they may reference APIs that require a purpose string. While your app might not use these APIs, a purpose string is still required. You can contact the developer of the library or SDK and request they release a version of their code that doesn't contain the APIs. Learn more (https://developer.apple.com/documentation/uikit/core_app/protecting_the_user_s_privacy).
>
>Best regards,
>
>The App Store Team

废弃API的引用，这个是SDK中引用的，升级SDK之后解决了此问题。邮件提示获取定位权限需要详细的说明，因为Unity在导出工程的iPhoneSensor.mm文件中使用了定位相关的API，如果确定游戏内没有使用定位功能，可以手动删掉这些API的引用（Unity最新的LTS版本对此进行了修正，不过在项目中后期升级Unity太过麻烦，所以选择手动删除相关API）。同时如果用到了这个权限，那么对此权限的使用说明要更加清晰，比如：
>iOS申请权限应说明用途.
>反例 : "应用需要访问你的同意  才能访问相机"
>正例 : "我们需要相机权限，才能保存您的游戏头像"

这次拒审之后重头戏就来了，连续几次4.3，花了很大的精力来处理这个问题，庆幸的是最终过审了。

>4. 3 Design: Spam
>Guideline 4.3 - Design
>
>
>`We noticed that your app provides the same feature set as other apps submitted to the App Store; it simply varies in content or language, which is considered a form of spam.`
>
>The next submission of this app may require a longer review time, and this app will not be eligible for an expedited review until this issue is resolved.
>
>Next Steps
>
>- Review the Design section of the App Store Review Guidelines.
>- Ensure your app is compliant with all sections of the App Store Review Guidelines and the Terms & Conditions of the Apple Developer Program. 
>- Once your app is fully compliant, resubmit your app for review.
>
>When creating multiple apps where content is the only varying element, you should offer a single app to deliver differing content to customers. If you would like to offer this content for purchase, it would be appropriate to use the in-app purchase API.
>
>Alternatively, you may consider creating a web app, which looks and behaves similar to a native app when the customer adds it to their Home screen. Refer to the Configuring Web Applications section of the Safari Web Content Guide for more information.
>
>Submitting apps designed to mislead or harm customers or evade the review process may result in the termination of your Apple Developer Program account. Review the Terms & Conditions of the Apple Developer Program to learn more about our policies regarding termination.

>Guideline 4.3 - Design
>
>
>`This app duplicates the content and functionality of other apps submitted by you or another developer to the App Store, which is considered a form of spam.`
>
>`Specifically, this app appears to be identical to another app previously submitted under a terminated Apple Developer Program account.`
>
>Apps that simply duplicate content or functionality create clutter, diminish the overall experience for the end user, and reduce the ability of developers to market their apps.
>
>The next submission of this app may require a longer review time, and this app will not be eligible for an expedited review until this issue is resolved.
>
>Next Steps
>
>- Review the Design section of the App Store Review Guidelines.
>- Ensure your app is compliant with all sections of the App Store Review Guidelines and the Terms & Conditions of the Apple Developer Program. 
>- Once your app is fully compliant, resubmit your app for review.
>
>Submitting apps designed to mislead or harm customers or evade the review process may result in the termination of your Apple Developer Program account. Review the Terms & Conditions of the Apple Developer Program to learn more about our policies regarding termination.

两次都是4.3，不过描述还是有很大差别的，可以只看前面一两行，后面都是格式回复。这两次提包，第一次是在机审被拒，第二次是在人工审核被拒。第二次有游戏登陆记录，所以确认是到了人工审核。人工审核被拒，应该是相同的游戏名，UI基本一样，直接就被拒审了。到这里已经毫无办法，常规手段肯定会被4.3，只有朝马甲包的方向尝试了。最气的是这个项目是第一次提审，也不知道跟谁撞车了，估计跟今年版号趋严有关，苹果大力打击马甲包，AppStore的审核标准可能提高了很多。

#### 2、混淆代码
修改了Unity的一个混淆工具的相关代码，把随机字母的形式改为了随机单词的形式，避免随机字母这种过于直白的混淆被拒审。万一AppStore的审核机制会分析代码的语义，发现类名是全大写或者全小写，而且完全不能组成一个单词，违背了一般的驼峰式或者下划线式的命名习惯，这个问题不得不考虑。
 
#### 3、混淆资源
资源名和路径都要混淆，并且资源计算的md5值要不同，资源的size也需要不同。因为Unity在AssetBundle文件头部存储了当前文件内所有资源的positon和size，所以要改变AB包的md5值和大小比较简单，只需要在AB包文件的尾部Append一部分随机长度的byte[]数组就可以了。

#### 4、隔离开发环境和打包环境
网上很多说用不同的机器来打包，因为可操作性比较差，所以想了另外一个方式。创建一个新的用户，新用户可以较大程度上隔离之前的环境。删除这个用户内不相关的钥匙串证书，只保留打包需要的，AppleID也不要登陆。
 
#### 5、iOS预审核
对比了WeTest和QuickSDK的iOS预审核功能，QuickSDK的功能相对更加完善，并且说明更加精准。
QuickSDK主要提到了几个点：
a. 打包的机器用户名要更加通用，比如admin和jenkins等无特征性用户。
b. 预审核提示在代码中发现了dlopen和dlsym。这个问题很奇怪，是在il2cpp导出文件中发现的dlopen和dlsym，最终定位到了MacProxy.cs这个文件，在Mono.Net命名空间中。如果导出一个空的Unity工程，不会发现这个文件被导出。然后在空工程中引入一个第三方zip库，在导出工程中出现了这个文件的il2cpp导出，猜测是这个库导致MacProxy.cs文件被导出。尝试在项目中删除zip库，导出后依然存在MacProxy.cs文件。那项目中不止一处会导致导出工程中包含这个引用，所以用删减代码的方式来去掉MacProxy.cs的导出是不可行的。看了一下MacProxy.cs这个文件，主要是mac下网络代理相关的功能。游戏内基本没有使用到相关的API，可能是某些关联的依赖导致这个文件被il2cpp导出，所以直接注释掉MacProxy.cs导出的cpp文件中的接口调用，同时修改里面的dlopen和dlsym字符串文本，避免在二进制搜索中找到这个字符串。

上面的组合拳一套下来，基本上也很难检测到相似的代码和资源了。不过苹果粑粑这种逼良为娼的行为，着实让人掉头发。
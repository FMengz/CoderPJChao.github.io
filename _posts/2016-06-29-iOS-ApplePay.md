---
layout:     post
title:      "iOS-APP集成Apple Pay指南"
subtitle:   "每天进步一点点"
date:       2016-06-29
author:     "Pegu"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - iOS
    - 每天进步一点点
    - 前端开发
---



>版权声明：本文为作者原创文章， 如果有相同的或者相似的，那将是我的荣幸。

### Apple Pay是什么？
Apple Pay目前在国内上线已有一段时间，这意味着消费者可通过 `苹果手机、苹果手表`等智能设备来进行支付，它的功能类似一个“卡包”，讲实体银行卡虚拟到手机里，用户可以绑定`储蓄卡或信用卡`实现刷卡支付。

---
###为什么使用Apple Pay

**从理论上看：**微信支付也好、支付宝也好，均属于“结算平台”，他们会透过银联，从你的银行里提出现金代为托管；直至你把金钱放回银行之前，都只能在微信/支付宝上使用。但 Apple Pay 只是一张电子信用卡，你的钱仍然在银行里，付款时直接从银联向银行提取。而从技上术上，微信支付也好、支付宝也好，在付款时都要透过相机或条码机，读取一次性的二维码；而 Apple Pay 则透过近场通信 (NFC) 的方式，读取 iPhone 上的 Token 令牌。

**从实际上看：**目前微信支付及支付宝的优势在于低入场门槛：商店只要有台智能手机，就毋须为移动支付购入/租用新设备，而消费者也不需要很高端的旗舰级设备。而 Apple Pay 优势是良好的用户体验，使用 Apple Pay 付费，既安全、又快捷、隐私度也比较高。

---
###设备支持有哪些要求？
Apple Pay需要支持`NFC`功能，目前只限于iPhone 6s、iPhone 6s Plus、iPhone 6、iPhone 6 Plus和Apple Watch这几款设备使用。同时，用户需讲手机操作系统版本升级到`iOS 9.2`以上，Apple Watch则需要`Watch OS 2.1`版本以上。

---
###Apple Pay如何使用？
在`iPhone`上，先打开系统自带的`Wallet`应用，后点右上角的`➕`符号，这时你有两个选择，可以用摄像头拍卡就能识别，也可以手动输入，或者通过iTunes绑定，至于`Apple Watch`怎么使用，应该操作差不多，具体情况本人未去实践，还望谅解。

---
###Apple Pay环境配置

- 配置Bundle ID
![14671699608022.jpg](http://upload-images.jianshu.io/upload_images/1628418-f6b2e3c0397a1001.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 添加商户的ID
![14671702234549.jpg](http://upload-images.jianshu.io/upload_images/1628418-6455d68fbbe679ec.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
- 把工程中对应的Bundle ID添加进去
![14671703281837.jpg](http://upload-images.jianshu.io/upload_images/1628418-f4414773db7b7aa7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 点击创建好的商户ID
![14671705166905.jpg](http://upload-images.jianshu.io/upload_images/1628418-e8bf3abc959c4113.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 编辑商户ID
![14671711439975.jpg](http://upload-images.jianshu.io/upload_images/1628418-170da8ae1019f459.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 添加CSR文件
![14671714784874.jpg](http://upload-images.jianshu.io/upload_images/1628418-ca519e9048761690.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 下载配置好的商户ID
![14671715087391.jpg](http://upload-images.jianshu.io/upload_images/1628418-f1f11784f2f03016.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![14671715713657.jpg](http://upload-images.jianshu.io/upload_images/1628418-a6671b1c8a78f14c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 创建对应的App IDs
![14671716587260.jpg](http://upload-images.jianshu.io/upload_images/1628418-2f5ba9a6162d8283.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 勾选Apple Pay
![14671717034194.jpg](http://upload-images.jianshu.io/upload_images/1628418-a30994b9ee5c9211.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 配置支付环境
![14671720852032.jpg](http://upload-images.jianshu.io/upload_images/1628418-374d448ab749a6cf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- Clean一下工程，如果工程自动添加此文件则以上步骤正确
![14671720005926.jpg](http://upload-images.jianshu.io/upload_images/1628418-f35657a44fdb3594.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
###App项目内部集成方式
**Apple Pay**使用了**PassKit**框架，所以需要导入相应头文件

```objc
#import <PassKit/PassKit.h>
```

接收**Apple Pay**处理信息的回调，需要遵守协议，实现相应的代理方法

```objc
@interface ViewController ()<PKPaymentAuthorizationViewControllerDelegate>
@end
```
为了方便测试，触发支付操作时间写在`touchBegan:`方法中

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // code...
}
```

具体代码实现如下：

- 判断设备是否支持Apple Pay快捷支付功能

```objc
if (![PKPaymentAuthorizationViewController canMakePayments]) {
        // 提示用户该设备不支持Apple Pay快捷支付功能
        // code...
        return;
    }
```

- 判断设备是否绑定过可支付的银行卡

```objc
	/**
     *  若没有可用银行卡，则跳转到设置银行卡界面
     *  PKPaymentNetworkVisa                Visa国际卡
     *  PKPaymentNetworkChinaUnionPay       中国银联
     *  PKPaymentNetworkDiscover            Discover(美国流行的信用卡)
     */
    if (![PKPaymentAuthorizationViewController canMakePaymentsUsingNetworks:@[PKPaymentNetworkVisa, PKPaymentNetworkChinaUnionPay, PKPaymentNetworkDiscover]]) {
        
        // 进入设置银行卡界面
        [[[PKPassLibrary alloc] init] openPaymentSetup];
        
    }

```

- 设置商品参数

```objc
	// 创建商品
    NSDecimalNumber *firstAmount = [NSDecimalNumber decimalNumberWithString:@"1.11"];
    NSDecimalNumber *secondAmount = [NSDecimalNumber decimalNumberWithString:@"2.22"];
    NSDecimalNumber *thirdAmount = [NSDecimalNumber decimalNumberWithString:@"3.33"];
    
    NSDecimalNumber *amountSum = [NSDecimalNumber zero];
    amountSum = [amountSum decimalNumberByAdding:firstAmount];
    amountSum = [amountSum decimalNumberByAdding:secondAmount];
    amountSum = [amountSum decimalNumberByAdding:thirdAmount];
    
    /**
     *  @param label        商品名称（英文名称默认全部显示大写）
     *  @param amount       商品价格 - NSDecimalNumber类型
     */
    PKPaymentSummaryItem *firstItem = [PKPaymentSummaryItem summaryItemWithLabel:@"FirstItem" amount:firstAmount];
    PKPaymentSummaryItem *secondItem = [PKPaymentSummaryItem summaryItemWithLabel:@"SecondItem" amount:secondAmount];
    PKPaymentSummaryItem *thirdItem = [PKPaymentSummaryItem summaryItemWithLabel:@"ThirdtItem" amount:thirdAmount];
    
    PKPaymentSummaryItem *itemsSum = [PKPaymentSummaryItem summaryItemWithLabel:@"PJChao" amount:amountSum];
```

- 创建支付请求（基本配置）

```objc
	PKPaymentRequest *request = [[PKPaymentRequest alloc] init];
    
    // 设置商户ID（merchant IDs）
    request.merchantIdentifier = @"merchant.com.zpj.ApplePayTest";
    
    // 设置国家代码(中国大陆)
    request.countryCode = @"CN";
    
    // 设置支付货币(人民币)
    request.currencyCode = @"CNY";
    
    // 设置商户的支付标准(3DS支付方式必须支持，其他方式可选)
    request.merchantCapabilities = PKMerchantCapability3DS;
    request.paymentSummaryItems = @[firstItem, secondItem, thirdItem, itemsSum];
    
    /**
     *  以上参数都是必须的
     *  以下参数不是必须的
     */
     
    // 设置收据内容
    request.requiredBillingAddressFields = PKAddressFieldAll;
    
    // 设置送货内容
    request.requiredShippingAddressFields = PKAddressFieldAll;
    
    // 设置送货方式
    PKShippingMethod *method = [PKShippingMethod summaryItemWithLabel:@"阿敏" amount:[NSDecimalNumber decimalNumberWithString:@"10.00"]];
    method.identifier = @"阿敏物流";
    method.detail = @"12小时到达";
    
    request.shippingMethods = @[method];
```

- 显示支付界面

```objc
	PKPaymentAuthorizationViewController *paymentVC = [[PKPaymentAuthorizationViewController alloc] initWithPaymentRequest:request];
    paymentVC.delegate = self;
    
    if (paymentVC == nil) return;
    
    [self presentViewController:paymentVC animated:YES completion:nil];
```

- 代理方法的实现

```objc
#pragma mark - <PKPaymentAuthorizationViewControllerDelegate>
- (void)paymentAuthorizationViewController:(PKPaymentAuthorizationViewController *)controller
                       didAuthorizePayment:(PKPayment *)payment
                                completion:(void (^)(PKPaymentAuthorizationStatus status))completion
{
    /**
     *  在这里支付信息应发送给服务器/第三方的SDK（银联SDK/易宝支付SDK/易智付SDK等）
     *  再根据服务器返回的支付成功与否进行不同处理
     *  这里直接返回支付成功
     */
	completion(PKPaymentAuthorizationStatusSuccess);
}

- (void)paymentAuthorizationViewControllerDidFinish:(PKPaymentAuthorizationViewController *)controller
{
    // 点击支付/取消按钮隐藏界面
    [controller dismissViewControllerAnimated:YES completion:nil];
}

```

###附上测试结果
![14671851652964.jpg](http://upload-images.jianshu.io/upload_images/1628418-2f32a91c15544167.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![test.gif](http://upload-images.jianshu.io/upload_images/1628418-99b3c630f75b57f3.gif?imageMogr2/auto-orient/strip)
![14671854207186.jpg](http://upload-images.jianshu.io/upload_images/1628418-d526e36f6983757d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

终于写完了，如果有什么问题，还望大神给予指点，Thank you。



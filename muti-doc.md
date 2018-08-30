#  iOS SDK接入文档

## 资源说明

• 所有SDK需要的资源都在SDK.zip包里

• 辅助测试库在SDK文件夹下 

### 通用资源

> - channel_config.json  ——配置文件（后台下载，必须引入；测试配置文件在oc资源demo中）[下载方法](http://access.hoolai.com/doc/2017/06/02/%E5%85%B6%E4%BB%96/#配置文件下载-channel-config-json)
> - AccessSDK.framework ——接入主库（Access_iOS.zip中，必须引入）
> - access.bundle ——接入主要资源（Access_iOS.zip中，必须引入）
> - demo_pay.a —— 辅助测试库（sdk目录下，2.11之后的版本引入）
> - security.data —— 证书文件(security目录下)（sdk提供）

### web配置说明
- 需要配置sdk url,每个游戏都不同，由sdk分配url
![](https://raw.githubusercontent.com/xyjwsj/static-resource/master/setting-url.png)

### 模块资源说明

#### •支付模块

   ºApp Store支付(libaccess_appstore.a)

>-![](https://raw.githubusercontent.com/xyjwsj/static-resource/master/appstore.jpg)

## Access SDK接入环境搭建

### 系统framework引入

- 请在游戏工程中添加以下Framework
```objective-c
Security.framework
CoreText.framwork
AdSupport.framwork
Foundation.framework
SystemConfiguration.framework
QuartzCore.framework
CoreTelephony.framework
CoreGraphics.framework
CoreLocation.framework
AssetsLibrary.framework
JavaScriptCore.framework
StoreKit.framework
libsqlite3.tbd
libz.tbd
libstdc++.6.tbd
```

### 添加Access SDK库文件

#### 引入通用资源到工程中

  通用资源引入方法 :将通用资源拖到工程面板中，在弹出选择框里按照图1-1进行选择（推荐勾选Copy items if needed)

![](http://webcdn.hulai.com/group1/M00/00/45/CgIJK1qyDR-AGk_JAABvRp7z-t4569.jpg)
图1-1

#### 引入模块资源

模块资源引入方法: 同引入通用资源

#### 引入配置文件

配置文件引入方法: 同引入通用资源

#### 项目参数配置


• 点击项目target，选择Build Setting tab页，搜索Other Linker Flags,双击后点击加号，添加为-ObjC即可，已经添加过不用再添加。效果如图1-2所示

![](http://cdn.hoolaiimg.com/group1/M00/00/2B/CgIJKlib0zWAJbQ9AACLPknOBOY277.png)




• 在Info.plist中配置Privacy - Photo Library Additions Usage Description项，类型选择String，值为：将要访问您的相册
![](http://webcdn.hulai.com/group1/M00/00/3F/CgIJK1oWmleAbqMfAAAuH5weKO0644.png)





• 在Info.plist中配置Privacy - Photo Library Usage Description项，类型选择String，值为：将要保存截图到相册
 ![](http://cdn.hoolaiimg.com/group1/M00/00/2B/CgIJKlidJ_eAR28kAABUEgaZ7Zs236.png)




• 在Info.plist中配置http网络访问的支持，配置App Transport Security Settings项，类型为Dictoionary，然后配置子项Allow Arbitrary Loads，类型为Boolean，值为Yes
 ![](http://cdn.hoolaiimg.com/group1/M00/00/2C/CgIJK1ib1O-AI--AAAA9MTKOcfI043.png)

## Access SDK基础功能

在配置好接入环境，导入SDK库和配置文件后，您即可开始介入SDK各功能

<font color=#DC143C>SDK初始化</font>
```objective-c
导入SDK头文件

 #import <AccessSDK/AccessSDK.h>
 #import <AccessSDK/UserExtDataKeys.h>
```

<font color=#DC143C>SDK初始化 delegate需要 InitCallback,ReportCallback,PayCallback,UserCallback 这四种回调中按需接入,具体方法可以见后续说明</font>
```objective-c
@interface SDKDelegate : NSObject<InitCallback,ReportCallback,PayCallback,UserCallback>

//在这里声明方法,在下面要做实现,比如初始化成功回调方法的声明
- (void)onInitSucceeded:(NSString *) result;

@end

@implementation SDKDelegate

// SDK 初始化成功的回调方法实现
- (void)onInitSucceeded:(NSString *)result {
    NSLog(@"onInitSucceeded---->%@",result);
    [rootView setIsInit:YES];
    // 目前仍处于未登陆状态
    [rootView loginStatusCallback:NO];
    [[AccessSDK sharedInstance] login];
}

@end

SDKDelegate* handler = nil;

@interface AppDelegate ()

@end

@implementation AppDelegate

@synthesize window;

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	// Override point for customization after application launch
	self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
	rootView = [[ViewController alloc] init]；
	rootView.title = @"测试接口";
	self.navController = [[UINavigationController alloc] init];
	[self.navController pushViewController:rootView animated:NO];
	self.window.backgroundColor = [UIColor whiteColor];
	[self.window addSubview:self.navController.view];
	[self.window setRootViewController:self.navController];
	[self.window makeKeyAndVisible];

	//接入代码开始
	handler = [[SDKDelegate alloc] init];
	[[AccessSDK sharedInstance] checkUpdate:@“3"]；//根据需求接入更新
	[AccessSDK sharedInstance].payDelegate = handler;
	[AccessSDK sharedInstance].userDelegate = handler;
	[AccessSDK sharedInstance].initDelegate = handler;
	[AccessSDK sharedInstance].reportDelegate = handler;
	[[AccessSDK sharedInstance] initSDK];
	[super application:application didFinishLaunchingWithOptions:launchOptions];
	//接入代码结束
	return YES;
}
```




<font color=#DC143C>生命周期调用需要注意调用super 父类方法</font>
```objective-c
- (void)applicationWillResignActive:(UIApplication *)application {
	// Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
	// Use this method to pause ongoing tasks, disable timers, and throttle down OpenGL ES frame rates. Games should use this method to pause the game.
}

- (void)applicationDidEnterBackground:(UIApplication *)application {
	// Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
	// If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
	[super applicationDidEnterBackground:application];
}

- (void)applicationWillEnterForeground:(UIApplication *)application {
	// Called as part of the transition from the background to the inactive state; here you can undo many of the changes made on entering the background.
	[super applicationWillEnterForeground:application];
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
	// Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
}

- (void)applicationWillTerminate:(UIApplication *)application {
	// Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
}

- (NSUInteger)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
	return UIInterfaceOrientationMaskAll;
}

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
	[super application:application openURL:url sourceApplication:sourceApplication annotation:annotation];
	return YES;
}

-(void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {
	[super application:application didReceiveLocalNotification:notification];
}

- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings {
	[super application:application didRegisterUserNotificationSettings:notificationSettings];
}

- (void)application:(UIApplication *)application handleActionWithIdentifier:(NSString *)identifier forRemoteNotification:(NSDictionary *)userInfo completionHandler:(void (^)())completionHandler{
	[super application:application handleActionWithIdentifier:identifier forRemoteNotification:userInfo completionHandler:^{
	}];
}

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
	[super application:application didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}


- (void)application:(UIApplication *)app didFailToRegisterForRemoteNotificationsWithError:(NSError *)err {
	[super application:app didFailToRegisterForRemoteNotificationsWithError:err];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
	[super application:application didReceiveRemoteNotification:userInfo];
}
```


<font color=#DC143C> SDK更新接口(选接)</font>
```objective-c
- [[AccessSDK sharedInstance] checkUpdate:@"游戏当前版本号"];
```

注:需要在调用初始化接口之前传入，此接口需要配合hoolai更新版本一起使用


<font color=#DC143C> SDK登录接口(必接)</font>
```objective-c
- [[AccessSDK sharedInstance] login];//此接口会触发登录回调

```
注：再未登出的情况下不得二次调用登录接口，以免游戏出现多次弹框


<font color=#DC143C>SDK登出接口(必接)</font>
```objective-c
-（void）logout；
[[AccessSDK sharedInstance] logout];
```

注:登出成功后会触发登出回调接口

```

<font color=#DC143C>支付接口(必接)</font>
```objective-c
中文版海外版都能用的支付方法
/**
 *  支付
 *
 *  @param itemName     商品名，不能为空
 *  @param amount       支付金额（单位：分）
 *  @param callbackInfo 透传参数，不能为空
 */
- (void)payName:(NSString*)itemName amount:(int)amount callbackInfo:(NSString*)callbackInfo;
[[AccessSDK sharedInstance] payName:@”xxxx” amount:x callbackInfo:xxxxx];

此方法中文版海外版都可以用

此接口会触发支付回调

扩展信息，回调时原样返回,因为各渠道回调参数限制不一致。请不要对该字段urlencode(会导致很多不兼容的情况)。回调参数暂时只支持长度50

callBackInfo中不要使用这些符号：“|”、“=”、“+”、“/”
```

```objective-c
海外版独有的方法,调用方式如上代码,推荐海外版使用这个方法 
- (void)payWithProduct:(ACProductInfo *)payObj;

 ACProductInfo *product = [[ACProductInfo alloc] init];
	product.itemName = @"test";
	product.amount = 998;
	product.itemId = @"tester";
	product.callbackUrl = payAmount;
	product.callbackInfo = @"testCallbackInfo";
 [[AccessSDK sharedInstance] payWithProduct:product];
```


<font color=#DC143C>选服接口(选接)</font>
```objective-c
-（BOOL）selectServer:(NSString*)selectServerId;返回值确定是否选服成功
 [[AccessSDK sharedInstance] selectServer:@”1”]；//此接口会触发选服回调
```


<font color=#DC143C>切换账号接口(选接)</font>
```objective-c
-（void）switchAccount；
 [[AccessSDK sharedInstance] switchAccount]; //此接口会触发登出回调接口

```

注:切换帐号操作SDK内部做了登出操作，并进行了登出回调，游戏需在登出回调中对游戏数据进行处理。接着触发登录 等同于如下操作

```objective-c
[[AccessSDK sharedInstance] logout];
 [[AccessSDK sharedInstance] login];
```


<font color=#DC143C>用户信息上报接口(必接,中文版海外版通用)</font>
```objective-c
-（void）setUserExtData:(NSMutableDictionary*)userExtData;//调用：

   [userData setObject:XXX forKey:ACTION_TYPE];//进入游戏(必填)

  XXX为：ACTION_ENTER_SERVER（进入游戏） / ACTION_LEVEL_UP（角色升级） / ACTION_CREATE_ROLE(创建角色)

   进入服务器的报送点，必须在进入游戏的第一时间点报送，不能有间隔（比如中间不能有开场动画）

   [userData setObject:@"Test_roleId" forKey:ROLE_ID];//必须是游戏里的角色ID，必传，请不要使用SDK返回的UID

   [userData setObject:@"Test_roleName" forKey:ROLE_NAME];//必须是游戏里角色的昵称，必传，没有的时候 可以传role_id

   [userData setObject:@"Test_level" forKey:ROLE_LEVEL];//角色等级,必传,没有等级概念可以传0 

   [userData setObject:@"Test_zoneId" forKey:ZONE_ID];

   [userData setObject:@"Test_zoneName" forKey:ZONE_NAME];//服务器名称(必填)

   [userData setObject:@"Test_balance" forKey:BALANCE];//代币余额(选填)

   [userData setObject:@"0" forKey:VIP];//vip等级(选填,只能传数字 @"0" @0 这两种格式都可以)

   [userData setObject:@"Test_partyname" forKey:PARTYNAME];//公会(选填)

   [userData setObject:@"Test_appversion" forKey:APP_VERSION];//版本号(选填)

   [userData setObject:@"Test_appResVersion" forKey:APP_RES_VERSION];//资源版本号(选填)

  [[AccessSDK sharedInstance] setUserExtData:userExtData];

```

*注:服务器id,必传,如果有删档测试，请更换新的zone_id或者通知SDK和BI也进行删档，否则会引起数据统计不准确
userExtData中的key需要使用UserExtDataKeys中定义的key*

<font color=#DC143C>BI数据报送(此版本不支持接入)</font>
```objective-c
/**
*  BI数据报送(不支持)
*
*  @param metric   报送指标类型，不能为空
*  @param jsonData 报送指标json串，不能为空
*/
- (void)sendBIData:(NSString*)metric jsonData:(NSString*)jsonData;
```

```objective-c

/**
 *  上报用户信息至SDK
 *
 *  @param userExtData 用户信息，不能为空
 */
- (void)setUserExtData:(NSMutableDictionary*)userExtData;
```

*注意：点击付费以及支付成功的事件报送，SDK内部已做相应的报送，外部不需要处理*

### 实现SDK相关回调

```objective-c
º相关回调都放在这个头文件里 

@required表示协议必须实现的方法 

@optional表示协议中选择实现的方法



#pragma mark - 初始化

@protocol InitCallback <NSObject>

@required


- (void)onInitSucceeded:(NSString*)result;

- (void)onInitFailed:(NSString*)result;

@optional

@end



#pragma mark - 支付

@protocol PayCallback <NSObject>

@required

- (void)onPaySuccess:(NSString*)param;

- (void)onPayFail:(NSString*)param;

- (void)onPayCancle:(NSString*)description;

@optional

/**
 * 获取支付产品的回调
 */
- (void)onProductInfo:(NSArray<PayProduct *>*)products;

@end


#pragma mark - 报送

typedef enum _STORE_PROMOTION_SETTING {
	sUPDATE_PROMOTION_ORDER = 0,
	sUPDATE_PROMOTION_VISIBILITY_HIDE,
	sUPDATE_PROMOTION_VISIBILITY_SHOW,
} STORE_PROMOTION_SETTING;


@protocol ReportCallback <NSObject>

@optional

- (void)onSendBIResult:(NSString*)msg;

- (BOOL)onMaintenance:(NSString*)msg;

@end



#pragma mark - 用户

@protocol UserCallback <NSObject>

@required

- (void)onLoginSucceeded:(ACUserLoginResponse*)userInfo;

- (void)onLoginFailed:(ACReturnValue*)returnValue;

- (void)onLogout:(NSString*)msg;

@optional

/**
 *  绑定手机回调
 *
 *  @param msg 信息
 */
- (void)onBindPhone:(NSString*)msg;


/**
 *  游戏退出回调
 *
 *  @param msg 描述
 */
- (void)onExit:(NSString*)msg;

/**
 *  获取服务器列表成功回调
 *
 *  @param serverInfos 服务器列表信息
 */
- (void)onGetServerListSuccess:(ServerInfos*)serverInfos;

/**
 *  获取服务器列表失败回调
 *
 *  @param msg 失败信息
 */
- (void)onGetServerListFail:(NSString*)msg;

/**
 *  选服成功回调
 *
 *  @param serverId 选服id
 */
- (void)onSelectServerSuccess:(NSString*)serverId;

/**
 *  选服失败回调
 *
 *  @param serverId 选服id
 */
- (void)onSelectServerFail:(NSString*)serverId;


@end
```

- 举例 在APPDelegate.h声明要实现的回调即可

```objective-c
@interface SDKDelegate : NSObject<InitCallback,ReportCallback,PayCallback,UserCallback>

//InitCallback 初始化相关回调 
- (void)onInitSucceeded:(NSString *) result;
- (void)onInitFailed:(NSString *) result;

//支付相关 PayCallback
- (void)onPaySuccess:(NSString *)param;
- (void)onPayFail:(NSString *)param;

//UserCallback 用户相关回调 
- (void)onLoginSucceeded:(ACUserLoginResponse *)userLoginResponse;
- (void)onLoginFailed:(ACReturnValue *)returnValue;
- (void)onLogout:(NSString *)msg;
- (void)onGetServerListSuccess:(ServerInfos *)serverInfos;
- (void)onSelectServerSuccess:(NSString *)serverId;
- (void)onBindPhone:(NSString *)msg;

//ReportCallback 报送相关回调
- (void)onSendBIResult:(NSString*)msg;
- (BOOL)onMaintenance:(NSString*)msg;

@end
```

- 举例 在APPDelegate.m实现的回调方法,处理游戏内逻辑


```objective-c
@interface SDKDelegate : NSObject<InitCallback,ReportCallback,PayCallback,UserCallback>

//InitCallback 初始化相关回调 
- (void)onInitSucceeded:(NSString *) result;
- (void)onInitFailed:(NSString *) result;

//支付相关 PayCallback
- (void)onPaySuccess:(NSString *)param;
- (void)onPayFail:(NSString *)param;

//UserCallback 用户相关回调 
- (void)onLoginSucceeded:(ACUserLoginResponse *)userLoginResponse;
- (void)onLoginFailed:(ACReturnValue *)returnValue;
- (void)onLogout:(NSString *)msg;
- (void)onGetServerListSuccess:(ServerInfos *)serverInfos;
- (void)onSelectServerSuccess:(NSString *)serverId;
- (void)onBindPhone:(NSString *)msg;

//ReportCallback 报送相关回调
- (void)onSendBIResult:(NSString*)msg;
- (BOOL)onMaintenance:(NSString*)msg;

@end
```

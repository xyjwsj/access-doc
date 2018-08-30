#  iOS SDK接入文档

## 资源说明

• 所有SDK需要的资源都在SDK.zip包里

• 辅助测试库在SDK文件夹下 

### 通用资源

> - channel_config.json  ——配置文件（后台下载，必须引入；测试配置文件在oc资源demo中）[下载方法](http://access.hoolai.com/doc/2017/06/02/%E5%85%B6%E4%BB%96/#配置文件下载-channel-config-json)
> - AccessSDK.framework ——接入主库（Access_iOS.zip中的sdk目录中，必须引入）
> - access.bundle ——接入主要资源（Access_iOS.zip中的sdk目录中，必须引入）
> - demo_pay.a —— 辅助测试库（sdk目录下，2.11之后的版本引入）
> - security.data —— 证书文件（sdk提供）

### web配置说明
- 需要配置sdk url,每个游戏都不同，由sdk分配url
![](https://raw.githubusercontent.com/xyjwsj/static-resource/master/setting-url.png)

### 模块资源说明

#### •支付模块

   ºApp Store支付(libaccess_appstore.a)

>-![](https://raw.githubusercontent.com/xyjwsj/static-resource/master/appstore.jpg)

## SDK接入环境搭建

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

### 添加SDK库文件

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

## Hoolai SDK基础功能

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


<font color=#DC143C>Facebook登录(海外版选接)</font>
```objective-c
  注意：这个模块需要你在facebook开发者中心注册你的应用（https://www.facebook.com），下代码添加进你应用的 info.plist文件中
  
        <key>CFBundleURLTypes</key>
        <array>
            <dict>
                <key>CFBundleURLSchemes</key>
                <array>
                    <string>fb{yourAppId}</string>
                </array>
            </dict>
        </array>
        <key>FacebookAppID</key>
        <string>{yourAppId}</string>
        <key>FacebookDisplayName</key>
        <string>{yourAppDisplayName}</string>
        <key>LSApplicationQueriesSchemes</key>
        <array>
            <string>fbapi</string>
            <string>fb-messenger-api</string>
            <string>fbauth2</string>
            <string>fbshareextension</string>
        </array>


```

<font color=#DC143C>Facebook分享(选接)</font>
```objective-c
* 注意：这个模块需要依赖 FaceBook 登录模块的项目配置，即上面 FaceBook 登录中 info.plist 中的配置，参见上面配置

/*
 * Facebook 图片分享
 * 照片大小必须小于 12MB
 * 用户需要安装版本 7.0 或以上的原生 iOS 版 Facebook 应用
 */
- (void)facebook2ShareImageWithImagePath:(NSString *)imagePath success:(void (^)(NSDictionary *result))success failure:(void (^)(NSError *error))failure;

/*
 * Facebook 链接分享
 * link：表示需要分享的链接（必传）
 * contentTitle：表示链接中的内容的标题（可不传）
 * imageURL：在帖子中显示的缩略图的网址（可不传）
 * contentDescription：内容的描述，通常为 2-4 个句子（可不传）
 */
- (void)facebook2ShareLink:(NSString *)link contentTitle:(NSString *)contentTitle imageURL:(NSString *)imageURL contentDescription:(NSString *)contentDescription success:(void (^)(NSDictionary *result))success failure:(void (^)(NSError *error))failure;

/*
 * Facebook 视频分享
 * 视频大小必须小于 12MB
 * 分享内容的用户应安装版本 26.0 或以上的 iOS 版 Facebook 客户端
 * 视频网址 url 必须是资产网址，否则无法分享。例如，您可以从 UIImagePickerController 获取视频资产网址，[info objectForKey:UIImagePickerControllerReferenceURL]
 */
- (void)facebook2ShareVideo:(NSURL *)url success:(void (^)(NSDictionary *result))success failure:(void (^)(NSError *error))failure;


- (void)share2Facebook:(UIButton *)sender {

	//分享图片

	NSString *imagePath = @"assets-library://asset/asset.JPG?id=987945B6-4D87-4DFD-AD4A-660218A673FA&ext=JPG";

	[[AccessSDK sharedInstance] facebook2ShareImageWithImagePath:imagePath success:^(NSDictionary *result) {
		NSLog(@"图片分享成功");
		NSLog(@"%@",result);

	} failure:^(NSError *error) {
		NSLog(@"图片分享失败");
		NSLog(@"%@",error);
	}];


//      分享链接
	[[AccessSDK sharedInstance] facebook2ShareLink:@"https://www.baidu.com/" contentTitle:nil imageURL:nil contentDescription:nil success:^(NSDictionary *result) {
		NSLog(@"链接分享成功");
		NSLog(@"%@",result);

	} failure:^(NSError *error) {
		NSLog(@"链接分享失败");
		NSLog(@"%@",error);
	}];


//    分享视频
//视频大小必须小于12m

	NSString *videoPath = @"assets-library://asset/asset.MOV?id=F54C49F5-D5F9-4645-BF91-F2A2C03E7B20&ext=MOV";

	[[AccessSDK sharedInstance] facebook2ShareVideo:[NSURL URLWithString:videoPath] success:^(NSDictionary *result) {
		NSLog(@"分享视频成功");
	} failure:^(NSError *error) {
		NSLog(@"分享视频失败");
	}];
}


```

<font color=#DC143C>获取产品信息(海外版)</font>
```objective-c
注: itemIds: 商品的唯一ID集合(必传,苹果后台配置商品id)

/*
 * 获取产品信息接口
 *
 */
- (void)getProductInfo:(NSArray<NSString *>*)itemIds;


- (void)getProductInfo {
	[[AccessSDK sharedInstance] getProductInfo:@[@"smash_weekly_en",@"smash_monthly_en",@"smash_yearly_en"]];
}

/**
 * 获取支付产品的回调接口
 */
- (void)onProductInfo:(NSArray<PayProduct *>*)products;

- (void)onProductInfo:(NSArray<PayProduct *> *)products {
	NSLog(@"%@",products);
}

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

   [userData setObject:XXX forKey:ACTION];//进入游戏(必填)

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

<font color=#DC143C>BI数据报送(选接)</font>
```objective-c
/**
*  BI数据报送
*
*  @param metric   报送指标类型，不能为空
*  @param jsonData 报送指标json串，不能为空
*/
- (void)sendBIData:(NSString*)metric jsonData:(NSString*)jsonData;
```

<font color=#DC143C>海外版Adjust和Facebook数据报送</font>
>对于数据报送，在事件报送之前需要在项目中添加ad_event.json文件，具体配置规则如下：

```objective-c
对于Adjust 报送，需要在文件中配组如下信息，键名即下面所示：

ADJUST_ENVIRONMENT_TYPE （注意：该选项用于决定Adjust是在何种模式运行，如果是在开发阶段，该模式为 sandbox，如果是在线上发布环境，请将此模式设置为 production）

具体配置，参考如下：
{
"adToken": {
"ADJUST_ENVIRONMENT_TYPE":"sandbox"
}
}
```

```objective-c
对于Adjust, 以及Facebook 数据报送
请注意：打开应用、升级、普通注册、快速注册、登录、创角、进入服务器、点击支付、支付成功，已经全部迁移至由服务器报送,外部不需要处理，
对于 完成新手引导，解锁成就以及其他的报送事件，可使用如下接口完成报送

例如：
- (void)customInfo:(id)sender {
    NSString *customKey = @"";
    NSMutableDictionary *temp = [[NSMutableDictionary alloc] init];
    [temp setValue:ACTION_CUSTOM forKey:ACTION];
    [temp setValue:@24 forKey:ZONE_ID];
    [temp setValue:customKey forKey:CUSTOM_KEY];
    [temp setValue:@"月光宝盒" forKey:ZONE_NAME];
    [temp setValue:@12 forKey:SERVER_ID];
    [temp setValue:@"逐鹿之战" forKey:SERVER_NAME];
    [temp setValue:@348 forKey:ROLE_ID];
    [temp setValue:@"乱七八糟" forKey:ROLE_NAME];
    [temp setValue:@20 forKey:ROLE_LEVEL];
    [temp setValue:@1 forKey:VIP];
    [[AccessSDK sharedInstance] setUserExtData:temp];
}

注意：其中，ACTION 必须对应 ACTION_CUSTOM，customKey为在海外管理后台配置的需要报送的事件名（比如完成新手引导，或者解锁成就以及其他事件），且不为空，如果为空，则无法报送, customKey所对应的值需要在海外管理后台进行配置
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

## 附录icon&LanchImage尺寸
IOS 10,11

|	Asset	|	iPhone 6s plus and iPhone 7plus and iPhone 8 Plus 5.5inch(@3x)	|	iPhone 6s and iPhone 7 and iPhone 8  4.7inch(@2x)	|	iPhone X 5.8inch(@3x)	|	iPad pro 9.7inch (@2x)	|	iPad pro 12.9inch(@2x)	|	iPad pro 10.5inch(@2x)	|	iPhone SE 4.7inch(@2x)	|
| ------------ | ------------ |------------ |------------ |
|	App icon	|	180 x 180	|	120 x 120	|	180 x 180	|	167 x 167	|	167 x 167	|	167 x 167	|	120 x 120	|
|	App icon for the App Store	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|
|	Launch image	|	1242 x 2208 (portrait)/2208 x 1242(landscape)	|	750 x 1334 (portrait)/1334 x 750(landscape)	|	1125 x 2436 (portrait)/2436 x 1125 (landscape)	|		1536 x 2048 (portrait)/2048 x 1536 (landscape)	|	2048 x 2732 (portrait)/2732 x 2048 (landscape)	|	1668 x 2224 (portrait)/2224 x 1668 (landscape)	|	640 x 1136 (portrait)/1136 x 640 (landscape)	|
|	Spotlight search results icon	|	120 x 120	|	80 x 80	|	120 x 120	|	80 x 80	|	80 x 80	|	80 x 80	|	80 x 80	|
|	Settings icon	|	87 x 87	|	58 x 58	|	87 x 87	|	58 x 58	|	58 x 58	|	58 x 58	|	58 x 58	|
|	Notification icon size	|	60 x 60	|	40 x 40	|	60 x 60	|	40 x 40	|	40 x 40	|	40 x 40	|	40 x 40	|

IOS7,8,9

|	Asset	|	iPhone 6 Plus (@3x)	|	iPhone 6 and iPhone 5 (@2x)	|	iPhone 4s (@2x)	|	iPad and iPad mini (@2x)	|	iPad 2 and iPad mini (@1x)	|
| ------------ | ------------ |------------ |------------ |
|	App icon	|	180 x 180	|	120 x 120	|	120 x 120	|	152 x 152	|	76 x 76	|
|	App icon for the App Store	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|	1024 x 1024	|
|	Launch image	|	1242 x 2208 (portrait)/2208 x 1242(landscape)	|	For iPhone 6, 750 x 1334/For iPhone 5, 640 x 1136	|	640 x 960	|	1536 x 2048 (portrait)/2048 x 1536 (landscape)	|	768 x 1024 (portrait)/1024 x 768 (landscape)	|
|	Spotlight search results icon	|	120 x 120	|	80 x 80	|	80 x 80	|	80 x 80	|	40 x 40	|
|	Settings icon	|	87 x 87	|	58 x 58	|	58 x 58	|	58 x 58	|	29 x 29	|

IOS 5,6

|	Description	|	Size for iPhone 5 and iPod touch (5th generation)	|	Size for high-resolution iPhone and iPod touch	|	Size for iPhone and iPod touch	|	Size for high-resolution iPad	|	Size for iPad	|
| ------------ | ------------ |
|	App icon	|	114 x 114	|	114 x 114	|	57 x 57	|	144 x 144	|	72 x 72	|
|	App icon for the App Store	|	1024 x 1024 (recommended)	|	1024 x 1024 (recommended)	|	512 x 512	|	1024 x 1024 (recommended)	|	512 x 512	|
|	Launch image	|	640 x 1136	|	640 x 960	|	320 x 480	|	1536 x 2008 (portrait)/2048 x 1496 (landscape)	|	768 x 1004 (portrait)/1024 x 748 (landscape)	|
|	Small icon for Spotlight search results and Settings	|	58 x 58	|	58 x 58	|	29 x 29	|	100 x 100 (Spotlight search results)/58 x 58 (Settings)	|	50 x 50 (Spotlight search results)/29 x 29 (Settings)	|

- DESC

|||||
| ------------ | ------------ | ------------ | ------------ |
| iPhone Portrait| iOS 8-Retina HD 5.5| （1242×2208）| @3x|
| iPhone Portrait| iOS 8-Retina HD 4.7| （750×1334）| @2x|
| iPhone Portrait| iOS 7,8-2x| （640×960）| @2x|
| iPhone Portrait| iOS 7,8-Retina 4| （640×1136）| @2x|
| iPhone Portrait| iOS 5,6-1x| （320×480）| @1x|
| iPhone Portrait| iOS 5,6-2x| （640×960）| @2x|
| iPhone Portrait| iOS 5,6-Retina4| （640×1136）| @2x|

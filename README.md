*Share集成指南*

*1．导入SDK*
    将ShareSDK文件夹到项目文件夹中夹并拖入项目中。然后打开项目设置中的Build Phases页，在“Link Binary With Libraries”一栏中，点击左下角的“＋”号；在弹出窗口里面分别将SystemConfiguration.framework、QuartzCore.framework、MessageUI.framework以及libicucore.dylib加入到项目中。如图所示：
!http://www.sharesdk.cn/media/attached/t1.png(image)!

*2. 配置URL Scheme*
    打开*-Info.plist（*代表你的工程名字）。在配置文件中新增一项URL types
（如果存在可以不创建），展开URL types – URL Schemes，在URL Schemes下新增一项用于新浪微博授权的Scheme（如果不添加则会导致新浪微博的SSO授权方式无法返回应用）。其填写格式为：sinaweibosso.2279784657，其中后面的数字段为你在新浪微博申请的AppKey。如下图所示：
!http://www.sharesdk.cn/media/attached/t2.png(image)!
    另外，如果需要使用微信或者QQ平台接口也需要在此项下面添加相应的Scheme。

*3．嵌入代码*
    打开*AppDelegate.h(*代表你的工程名字)。导入头文件ShareSDK.h。实现代码如下：
<pre><code>#import <ShareSDK/ShareSDK.h>
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{

    [AGShareSDK registerApp:@"520520test"];
    //省略代码
    ⋯⋯
}</code></pre>

   然后，在处理请求URL的委托方法中加入ShareSDK的处理方法，如下：
<pre><code>- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
    return [ShareSDK handleOpenURL:url wxDelegate:self];
}

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    return  [ShareSDK handleOpenURL:url wxDelegate:self];
}</code></pre>
   此处理方法已包括微信和QQ的回复处理，如果使用ShareSDK内置提示功能可以不需要再实现消息的微信和QQ的回复消息捕获。

*4．配置社区应用信息*
   打开main.m文件，导入头文件ShareConfig.h（P.S. 此头文件必须在此文件下导入，否则会导致编译不通过）。如下：
<pre><code>#import <UIKit/UIKit.h>
#import <ShareSDK/ShareConfig.h>
#import "AGAppDelegate.h"

int main(int argc, char *argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AGAppDelegate class]));
    }
}</code></pre>
   打开ShareSDKConfig.h文件，根据需求设置各个平台的App相关信息（每个平台的App都需要到相应平台上进行应用登记后来取的相关信息）。


*5. 用户授权*
   要想操作相关平台提供的方法必须要先取得用户授权。ShareSDK提供了单独的授权接口来实现用户的登录授权。其代码如下：
<pre><code>[ShareSDK authWithType:ShareTypeSinaWeibo result:^(SSAuthState state, id<ICMErrorInfo> error) {
        if (state == SSAuthStateSuccess)
        {
            NSLog(@"成功");
        }
        else if (state == SSAuthStateFail)
        {
            NSLog(@"失败");
        }
}];</code></pre>

   第一个参数为需要进行授权的平台类型。第二个参数则是授权返回结果的处理方法。需要注意的是ShareSDK中提供的各种需要进行授权的接口（包括分享、获取用户信息等）都已经在内部进行授权检测判断，不需要开发者单独进行授权。
授权界面效果如下所示：
!http://www.sharesdk.cn/media/attached/t3.png(image)!

   如果需要取消授权可以调用ShareSDK的cancelAuthWithType接口，如下：
<pre><code>[ShareSDK cancelAuthWithType:ShareTypeSinaWeibo];</code></pre>

*6．分享内容*
在分享内容部分ShareSDK提供了三种接口来满足不同需求的分享功能实现。

6.1 菜单方式分享：
使用此方式进行分享会首先弹出菜单供用户选择分享的目标平台，然后再显示内容编辑界面供用户进行分享内容编辑，最后进行分享。调用该方式的接口如下：
<pre><code>[ShareSDK showShareActionSheet:self 
  		        shareList:nil
                                  content:[ShareSDK publishContent:content
                                                    defaultContent:@""
                                                             image:image
                                                      imageQuality:0.8
                                                         mediaType:SSPublishContentMediaTypeNews
                                                             title:@"ShareSDK"
                                                               url:url
                                                      musicFileUrl:nil
                                                           extInfo:nil
                                                          fileData:nil]
                           statusBarTips:YES
                         oneKeyShareList:[NSArray defaultOneKeyShareList]
                          shareViewStyle:ShareViewStyleDefault
                          shareViewTitle:@"内容分享"
                                  result:^(ShareType type, SSPublishContentState state, id<ISSStatusInfo> statusInfo, id<ICMErrorInfo> error, BOOL end) {
                                if (state == SSPublishContentStateSuccess)
                                {
                                    NSLog(@"成功!");
                                }
                                else if(state == SSPublishContentStateFail)
                                {
                                    NSLog(@"失败!");
                                }
}];
</code></pre>
    第一个参数用于指定菜单显示在哪个视图上。
    第二个参数则用于指定菜单弹出的分享平台列表，传入nil则表示显示所有分享平台，如果开发者需要自己定制显示列表则通过此参数进行控制。ShareSDK提供了getShareListWithType方法来进行分享列表的构造。
    第三个参数为实现了ISSPublishContent协议的对象，需要调用了ShareSDK的publishContent来构造对象。需要特别说明一下的是content参数对象除了基础的图片和文字字段外还包括了微信和QQ这两个平台的其他分享类型参数字段（包括News、Music、Video等，通过mediaType来指定分享出去的类型），如果不需要使用微信和QQ的分享功能，可以在构造分享内容对象时把参数值设置为nil或者默认值即可。
    第四个参数为是否在状态栏上显示提示信息（包括成功和错误提示）。
    第五个参数为一键分享的平台显示列表，用于控制允许一键分享的目标平台。如果不需要一键分享可以把参数设置为nil，或者是想使用默认列表可以调用[NSArray defaultOneKeyShareList]方法。此参数仅在shareViewStyle为ShareViewStyleDefault时生效。
    第六个参数用于指定分享内容编辑视图的样式，ShareSDK提供了三种方式，具体显示样式可以参照下面提供的应用截图。
    第七个参数为分享内容的编辑视图标题。
    第八个参数用于处理分享返回后续的处理方法。
    菜单分享方式的效果如下图所示：
!http://www.sharesdk.cn/media/attached/t4.png(image)!
!http://www.sharesdk.cn/media/attached/t5.png(image)!

6.2 分享内容视图方式
    使用此方式直接弹出内容分享编辑框，用户编辑完内容后点击发布直接分享出去。其调用代码如下：
<pre><code>[ShareSDK shareContentWithType:ShareTypeSinaWeibo
                                  content:[ShareSDK publishContent:CONTENT
                                                    defaultContent:@""
                                                             image:[UIImage imageNamed:IMAGE_NAME]
                                                      imageQuality:0.8
                                                         mediaType:SSPublishContentMediaTypeText]
                     containerController:self
                           statusBarTips:YES
                         oneKeyShareList:[NSArray defaultOneKeyShareList]
                          shareViewStyle:ShareViewStyleDefault
                          shareViewTitle:@"内容分享"
                                  result:nil];
</code></pre>

    第一个参数为分享的目标平台类型。其他参数与菜单方式中的content参数含义相同。其显示效果如上面编辑分享内容视图截图。

6.3 直接分享方式
    此方式不需要通过任何分享界面直接发送到指定的平台。调用代码如下：
<pre><code>[ShareSDK shareContentWithShareList:[ShareSDK getShareListWithType:ShareTypeSinaWeibo,nil]
                                       content:[ShareSDK publishContent:CONTENT
                                                         defaultContent:@""
                                                                  image:[UIImage imageNamed:IMAGE_NAME]
                                                           imageQuality:0.8
                                                              mediaType:SSPublishContentMediaTypeText]
                                statusBarTips:YES
                                       result:nil];
</code></pre>

    其中第一个参数为分享的目标平台类型数组，使用ShareSDK的getShareListWithType方法进行数组列表构造。其它参数与上面所说的分享接口同名参数含义相同。


6.4 离线分享内容提交。
    对于分享失败或者尚未进行分享的内容可以通过调用ShareSDK的flushOfflineShare方法进行离线内容的重新提交。其调用方式如下：
<pre><code>[ShareSDK flushOfflineShare:YES result:nil];</code></pre>
    第一个参数为是否在状态栏显示分享提示。
    第二个参数则为分享返回的处理方法。

*7. 获取当前用户信息*
    获取当前授权用户信息，其调用方式如下：
<pre><code>[ShareSDK getUserInfoWithType:ShareTypeSinaWeibo
                                  result:^(BOOL result, id<ISSUserInfo> userInfo, id<ICMErrorInfo> error) {
                               if (result)
                               {
                                   NSLog(@"成功");
                               }
                               else
                               {
                                   NSLog(@"失败");
                               }
                           }];
</code></pre>
    其中第一个参数为平台类型，用于指定获取哪个平台的授权用户信息。如果指定平台的用户尚未进行授权则弹出授权界面。第二个参数为授权返回的处理方法。


*8. 关注用户*
    关注指定用户（此接口目前仅支持新浪微博和腾讯微博），其调用方式如下：
<pre><code>[ShareSDK followUserWithName:@"ShareSDK"
                              shareType:ShareTypeSinaWeibo
                                 result:^(BOOL result, id<ISSUserInfo> userInfo, id<ICMErrorInfo> error) {
                              if (result)
                              {
                                  NSLog(@"成功");
                              }
                              else
                              {
                                  NSLog(@"失败");
                              }
}];
</code></pre>
    其中第一个参数为关注用户的昵称，第二个参数为平台类型，用于指定获取哪个平台的授权用户信息。如果指定平台的用户尚未进行授权则弹出授权界面。第二个参数为授权返回的处理方法。

*9. 获取关注列表*
    获取当前授权用户的关注列表（此接口目前仅支持新浪微博和腾讯微博），其调用方式如下：
<pre><code>[ShareSDK getFriendsWithType:ShareTypeSinaWeibo
                                 pageNo:1
                                 result:^(BOOL result, NSArray *users, BOOL hasNext, id<ICMErrorInfo> error) {
                              if (result)
                              {
                                  NSLog(@"成功");
                              }
                              else
                              {
                                  NSLog(@"失败");
                              }
}];
</code></pre>
    其中第一个参数为平台类型，用于指定获取哪个平台的授权用户关注列表，如果指定平台的用户尚未进行授权则弹出授权界面。第二个参数为取关注列表的第几页数据，页码从1开始。第三个参数为取关注列表返回的处理方法。
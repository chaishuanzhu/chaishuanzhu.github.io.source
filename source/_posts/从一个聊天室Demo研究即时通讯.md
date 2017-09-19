---
title: 从一个聊天室Demo研究即时通讯
date: YYYY-MM-DD hh:mm:ss
tags: 即时通讯                   #多标签时以[tag1,tag2]格式填写
categories: iOS              #多类别时以[category1,category2]格式填写
---

## 从一个聊天室Demo研究即时通讯

* 聊天室Demo：http://www.workerman.net/workerman-chat
* iOS 参考文章：http://www.jianshu.com/p/2dbb360886a8

传输协议 websocket 传输格式 json 
具体业务根据json传输的内容来实现。

### 先从最直观的页面入口开始分析

先启动项目 

![屏幕快照 2017-08-26 下午10.22.36.png](http://upload-images.jianshu.io/upload_images/1386124-b2ccdff474a48a92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开页面

![屏幕快照 2017-08-26 下午10.24.58.png](http://upload-images.jianshu.io/upload_images/1386124-4d1ae3b139984981.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开项目源文件，找到对应的入口文件

![屏幕快照 2017-08-26 下午10.28.10.png](http://upload-images.jianshu.io/upload_images/1386124-c306a37f70192c5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分析接口：
连接服务器：
```
ws://127.0.0.1:7272
```
登陆：(用户名：222，聊天室ID：1)
```
{"type":"login","client_name":"222","room_id":"1"}  返回数据：{"type":"login","client_id":xxx,"client_name":"xxx","client_list":"[...]","time":"xxx"}
```
发送消息：(to_client_id发送给谁，all表示聊天室所有人，content消息内容)
```
{"type":"say","to_client_id":"all","to_client_name":"","content":"66666"}
```
接收消息：
```
{"type":"say","from_client_id":xxx,"to_client_id":"all/client_id","content":"xxx","time":"xxx"}
```
退出：
```
{"type":"logout","client_id":xxx,"time":"xxx"}
```
测试（[Advanced REST client](https://advancedrestclient.com/)）：


![屏幕快照 2017-08-26 下午10.46.45.png](http://upload-images.jianshu.io/upload_images/1386124-99c78bd9f80fb005.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

服务器地址：http://www.chaisz.xyz/webchat
socket接口：ws://www.chaisz.xyz:7272

一切OK，接下来准备研究一下iOS

ios项目地址：https://github.com/chaishuanzhu/xchat-ios


* ios应用进入后台代码会停止执行。此时想要收到消息只能通过远程推送。
* 自己搭建推送服务器（PHP）参考文章：http://blog.csdn.net/lin1109221208/article/details/51719873  （需要有开发者账号制作推送证书）

### 服务端代码

```
<?php

// set_time_limit(0);

// sleep(5);

// 这里是我们上面得到的deviceToken，直接复制过来（记得去掉空格）
$deviceToken = 'ed8c1c155a6daf448f610ae749eec794be551f8d1b145448711eea973583f820';


// Put your private key's passphrase here:
$passphrase = '123456';


// Put your alert message here:
$message = 'My first push test!';


////////////////////////////////////////////////////////////////////////////////


$ctx = stream_context_create();

stream_context_set_option($ctx, 'ssl', 'allow_self_signed', true);

stream_context_set_option($ctx, 'ssl', 'verify_peer', false);

stream_context_set_option($ctx, 'ssl', 'local_cert', '/Applications/MAMP/htdocs/Push-dev/ck.pem');

stream_context_set_option($ctx, 'ssl', 'passphrase', $passphrase);


// Open a connection to the APNS server

//这个为正是的发布地址

 // $fp = stream_socket_client('ssl://gateway.push.apple.com:2195', $err, $errstr, 60, STREAM_CLIENT_CONNECT, $ctx);

//这个是沙盒测试地址，发布到appstore后记得修改哦

 $fp = stream_socket_client('ssl://gateway.sandbox.push.apple.com:2195', $err,$errstr, 60, STREAM_CLIENT_CONNECT|STREAM_CLIENT_PERSISTENT, $ctx);

//$fp=stream_socket_client("udp://127.0.0.1:1113", $err, $errstr, 60, STREAM_CLIENT_CONNECT, $ctx);

if (!$fp)

exit("Failed to connect: $err $errstr" . PHP_EOL);


echo 'Connected to APNS' . PHP_EOL;


// Create the payload body

$body['aps'] = array(

'alert' => $message,

'sound' => 'default'

);


// Encode the payload as JSON

$payload = json_encode($body);


// Build the binary notification

$msg = chr(0) . pack('n', 32) . pack('H*', $deviceToken) . pack('n', strlen($payload)) . $payload;


// Send it to the server

$result = fwrite($fp, $msg, strlen($msg));


if (!$result)

echo 'Message not delivered' . PHP_EOL;

else

echo 'Message successfully delivered' . PHP_EOL;


// Close the connection to the server

fclose($fp);

?>
```

### ios端代码
```
#import "AppDelegate.h"
#import <UserNotifications/UserNotifications.h>

@interface AppDelegate ()<UNUserNotificationCenterDelegate>

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    
    // 注册
    UNUserNotificationCenter* center = [UNUserNotificationCenter currentNotificationCenter];
    [center requestAuthorizationWithOptions:UNAuthorizationOptionBadge | UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionCarPlay completionHandler:^(BOOL granted, NSError * _Nullable error) {
        if (!error) {  
            [[UIApplication sharedApplication]registerForRemoteNotifications];
        }
    }];
    center.delegate = self;
    return YES;
}


- (void)applicationWillTerminate:(UIApplication *)application {
    // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
}


- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
    // 把此处输出的 “deviceToken” 去掉空格 配置到 php 代码中，就可以向这台设备发送通知了。
    NSLog(@"regisger success:%@", deviceToken);
}

- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(nonnull NSError *)error{
    
    NSLog(@"%@",error);
}

@end
```

测试结果：

![屏幕快照 2017-09-10 下午11.55.33.png](http://upload-images.jianshu.io/upload_images/1386124-3ccdfd3df1642c78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



未完待续！

### GetWayworker与ThinkPHP结合使用做一个聊天应用后台。

#### 创建项目
 在命令行下面，切换到你的web根目录下面并执行下面的命令：
```
composer create-project topthink/think xchat  --prefer-dist
```

![屏幕快照 2017-09-13 下午11.23.59.png](http://upload-images.jianshu.io/upload_images/1386124-5b2affe994654637.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


到应用根目录安装GetWayworker
```
composer require workerman/gateway-worker
```

![屏幕快照 2017-09-13 下午11.29.24.png](http://upload-images.jianshu.io/upload_images/1386124-411b35af30c945bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装GatewayClient
```
composer require workerman/gatewayclient
```


![屏幕快照 2017-09-13 下午11.32.06.png](http://upload-images.jianshu.io/upload_images/1386124-35eeb66ead3d0f9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后下载一个聊天室Demo：[http://www.workerman.net/workerman-chat](http://www.workerman.net/workerman-chat)

把里面的文件放到应用中。完成后的项目目录如下。




![屏幕快照 2017-09-13 下午11.40.32.png](http://upload-images.jianshu.io/upload_images/1386124-1136740af5c39dde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-09-13 下午11.42.01.png](http://upload-images.jianshu.io/upload_images/1386124-1a5f88692bdae02e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改start.php 

![屏幕快照 2017-09-13 下午11.46.16.png](http://upload-images.jianshu.io/upload_images/1386124-95a0bc500d76e33b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


修改Events.php (参考：http://doc2.workerman.net/326107)


![屏幕快照 2017-09-13 下午11.47.43.png](http://upload-images.jianshu.io/upload_images/1386124-01c2809824e418fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


启动应用
```
php start.php start
```
![屏幕快照 2017-09-13 下午11.51.12.png](http://upload-images.jianshu.io/upload_images/1386124-2485a5fe772d3c5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

测试链接：

![屏幕快照 2017-09-13 下午11.55.04.png](http://upload-images.jianshu.io/upload_images/1386124-32e46abbdba5bd5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新建bind.php控制器测试：
```
<?php
namespace app\index\controller;
//加载GatewayClient。安装GatewayClient参见本页面底部介绍
require_once ROOT_PATH.'vendor/workerman/gatewayclient/Gateway.php';
// GatewayClient 3.0.0版本开始要使用命名空间
use GatewayClient\Gateway;
// 设置GatewayWorker服务的Register服务ip和端口，请根据实际情况改成实际值
class Bind
{

  public function index() {
    echo '66666';
  }
    public function bind () {

        Gateway::$registerAddress = '127.0.0.1:1236';

        // 假设用户已经登录，用户uid和群组id在session中
        $uid      = '123456';
        $group_id = '1';
        $client_id = '7f00000108fe00000001';
        // client_id与uid绑定
        Gateway::bindUid($client_id, $uid);
        // 加入某个群组（可调用多次加入多个群组）
        Gateway::joinGroup($client_id, $group_id);
    }

    public function send_message($uid = '123456', $message = '25652652', $group = '1') {
        // //加载GatewayClient。安装GatewayClient参见本页面底部介绍
        // require_once '/your/path/GatewayClient/Gateway.php';
        // // GatewayClient 3.0.0版本开始要使用命名空间
        // use GatewayClient\Gateway;
        // 设置GatewayWorker服务的Register服务ip和端口，请根据实际情况改成实际值
        Gateway::$registerAddress = '127.0.0.1:1236';

        // 向任意uid的网站页面发送数据
        Gateway::sendToUid($uid, $message);
        // 向任意群组的网站页面发送数据
        Gateway::sendToGroup($group, $message);

        $this -> post('http://www.chaisz.xyz/xchat/index.php/push/push/ios',array('msg' => '54131321'));
    }

    function post($url, $post_data){
          //初始化
          $curl = curl_init();
          //设置抓取的url
          curl_setopt($curl, CURLOPT_URL, $url);
          //设置头文件的信息作为数据流输出
          curl_setopt($curl, CURLOPT_HEADER, 1);
          //设置获取的信息以文件流的形式返回，而不是直接输出。
          curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
          //设置post方式提交
          curl_setopt($curl, CURLOPT_POST, 1);
          //设置post数据
          curl_setopt($curl, CURLOPT_POSTFIELDS, $post_data);
          //执行命令
          $data = curl_exec($curl);
          //关闭URL请求
          curl_close($curl);
          //显示获得的数据
          // print_r($data);
    }
}

```


![屏幕快照 2017-09-13 下午11.57.41.png](http://upload-images.jianshu.io/upload_images/1386124-0d6d0c7497714724.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


测试：

![屏幕快照 2017-09-14 上午12.01.02.png](http://upload-images.jianshu.io/upload_images/1386124-5e752d86a52f9176.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![IMG_0208.PNG](http://upload-images.jianshu.io/upload_images/1386124-94b1c00d8145bb4a.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

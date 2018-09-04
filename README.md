# Unity For PaySDK 快速集成文档

## 下载并导入PaySDK

下载[Unity-For-PaySDK](https://github.com/MobClub/PaySDK-for-Unity3D.git), clone整个repo可能会很慢，浪费您的时间，最好加上depth=1参数。

```
git clone -b master --depth=1 https://github.com/MobClub/PaySDK-for-Unity3D.git
```

导入PaySDK.unitypackage到您的项目中。注意该操作可能会覆盖您原来已经存在的文件！


## 配置应用信息



#### iOS配置及注意事项(Android开发者可忽略)


1.配置AppKey和AppSecret，如下图

![image.png](https://upload-images.jianshu.io/upload_images/2121032-b0e45abe5bee6702.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


2.预配置Scheme 找到Unity3D-For-PaySDK - Assets - Plugins - iOS - PaySDK - Editor - PaySDK.mobpds,对其中的CFBundleURLSchemes进行设定,将其设置为您的 URI Scheme (注意不带'://') 

```
{
    "folders":     ["/SDK"],
    "buildSettings": { 
        "OTHER_LDFLAGS" : ["-lz","-ObjC","-lsqlite3"]
    }
    "URLSchemes" : {
    	"CFBundleURLSchemes" : ["wxc551587ec5a55ceb","ap2017073107971452","upmoba6b6c6d6"]
    }
}
```

3. 添加系统PassKit.frame， 并添加白名单，名单如下
![image.png](https://upload-images.jianshu.io/upload_images/2121032-3c406eeac72020a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


添加代码(伪代码)

1. 创建订单

```
PaySDKOrder order = new PaySDKOrder ();
order.orderId = DateTime.Now.ToFileTime().ToString();
order.amount = Convert.ToInt64(amount);
order.subject = "支付测试";
order.userId = "1234567890";
order.nickName = "nickName";
order.body = "body";
order.des = "des";
order.metadata = "{\n\t\"meta\": \"meta\"\n}";
PaySDK.payWithOrder (order, PaySDKChannel.PaySDKChannelWechat, this);

``` 

2. 发起支付

```
PaySDK.payWithOrder (order, PaySDKChannel.PaySDKChannelAlipay, this);

```


3. 支付回调

```
//支付结束 回调结果
	public void onPayEnd (PaySDKStatus status, string ticketId, long errorCode, string errorDes) {

		Debug.Log ("Status:" + status + "  ticketId:" + ticketId + "  errorCode:" + errorCode + "errorDes:" + errorDes);
		if (status == PaySDKStatus.PaySDKStatusCancel) {
			result = "Pay Cancel";
		} else if (status == PaySDKStatus.PaySDKStatusSuccess) {
			result = "Pay Success";
		} else if (status == PaySDKStatus.PaySDKStatusFail) {
			result = "Pay Fail Error:" + errorCode + "  Des:" + errorDes;
		} else {
			result = "Pay Result Unknown  Error:" + errorCode + "  Des:" + errorDes;
		}
	} 
```




#### Android配置及注意事项(iOS开发者可忽略)

1. 配置AndroidManifest

增加权限

```
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

设置AppKey和AppSecret(在Application标签里)

```
        <meta-data android:name="Mob-AppKey" android:value="您的AppKey"/>
        <meta-data android:name="Mob-AppSecret" android:value="您的AppSecret"/>
```

初始化，设置Application为MobApplication。

```
    <application
        android:name="com.mob.MobApplication" />
```

如果您不太熟悉如何配置AndroidManifest文件，可查看我们的Demo里的配置，Unity3D-For-PaySDK/Assets/Plugins/Android/AndroidManifest.xml。或者参考[Unity 文档](https://docs.unity3d.com/Manual/android-manifest.html)

2. PaySDK的是java语言开发的（PaySDK.jar), 这里在jar的基础上做了桥接， 桥接代码是开源的，您可以在Android-Unity-Bridge/UnityBridge目录下看到。UnityBridge项目最终生成PaySDK-Unity.jar和libpaysdk_bridge.so两个文件。涉及本地代码部分PaySDK.unitypackage里面只有armeabi、armeabi-v7a、x86平台的, 除此之外您还需要其它平台，可以自行编译。

## 添加代码（伪代码）

#### 1. 创建订单

```
            PaySDKOrder order = new PaySDKOrder ();
            order.orderId = "订单ID";
            order.amount = 支付金额;
            order.subject = "支付subject";
            order.body = "支付主休";
            order.des = "支付描述信息";
            order.metadata = "元数据";
```

#### 2. 创建监听器对象

我这里让当前Scene继承PaySDKHandler接口，在支付的scene里直接使用this就可以了

```
    public bool onWillPay (string ticketId) {
        return false;
    }

    public void onPayEnd (PaySDKStatus status, string ticketId, long errorCode, string errorDes) {
    } 
```

#### 3. 发起支付

调用payWithOrder即可, 第二个参数是枚举类型，表示支付渠道(支付宝、微信等)

```
PaySDK.payWithOrder (order, PaySDKChannel.PaySDKChannelAlipay, this);
```

    



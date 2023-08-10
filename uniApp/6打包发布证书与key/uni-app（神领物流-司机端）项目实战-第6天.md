## uni-app（神领物流）项目实战 - 第6天



**学习目标：**

- 知道如何生成 Android 证书

- 知道如何配置 App 端地图服务平台

- 知道如何实现实人认证的功能

- 知道如何实现一键登录的功能

- 知道如何实现消息推送的功能

  

### 一、自定义调试基座

在使用 HBuilderX 运行到 App 端时，官方提供的 Android 包（标准基座）来对项目进行打包，标准基座提供了日常开发的一系列功能，能够满足大部分日常的业务开发，但是当涉及到一些特殊需求时时就需要自定义调试基座，如消息推送、一键登录、人脸识别等。

生成自定义基座时及**正式发布 App 应用**时需要使用自有证书，然后再由 HBuilderX 提供的云打包服务生成安卓 APK 安装程序。

#### 1.1 安卓证书

安卓证书是一种数字签名技术，安卓系统要求安装到系统上的 App 必须要使用证书进行签名。签名相当于是 App 的身份证书，该证书能够证明 App 的归属者及合法性。

安卓证书的生成规则是由 Google 公司规定的，由开发者自助生成且免费。

##### 1.1.1 JRE 环境

1. 安装 JRE 环境

![](./assets/image-1.png)

![](./assets/image-2.png)

![](./assets/image-3.png)

2. 配置环境变量

![](./assets/image-4.png)

![](./assets/image-5.png)

![](./assets/image-6.png)

![](./assets/image-7.png)

注：大家在安装 JRE 时默认安装到了 `C:\Program Files\Java\jre-1.8\bin` 目录当中，因此环境变量添加的路径也是该目录对应的路径。

3. 验证 JRE 是否安装成功

```bash
# 打开命令行窗口
java -version
```

![](./assets/image-8.png)

##### 1.1.2 生成证书

HBuilderX 给出了[证书生成](https://ask.dcloud.net.cn/article/35777)的步骤说明：

```bash
# 打开命令行工具
keytool -genkey -alias 证书别名 -keyalg RSA -keysize 2048 -validity 36500 -keystore 证书名字.keystore
```

- `testalias` 是证书别名，可修改为自己想设置的字符，建议使用英文字母和数字
- `test.keystore` 是证书文件名称，可修改为自己想设置的文件名称，也可以指定完整文件路径
- `36500` 是证书的有效期，表示100年有效期，单位天，建议时间设置长一点，避免证书过期

回车后会提示：

![](./assets/image-9.png)

**！！！大家务必记住证书别名和密码！！！**

创建完证书后给出了一个警告，复制警告中的代码到命令行中执行

```bash
# 自已复制自已提示出来的警告代码
keytool -importkeystore -srckeystore test.keystore -destkeystore test.keystore -deststoretype pkcs12
```

![](./assets/image-10.png)

此时在执行命令的目录中就生成了一个 `xxxxx.keystore` 文件，即安卓证书了，同时还有一个 `xxxxx.keystore.old` 的备份文件，这个文件执行警告代码之前的证书文件，我们将这两个文件放在一起保管就可以了。

##### 1.1.3 云端证书

云端证书是由 DCloud 平台提供的生成证书的服务，登录到 [DCloud 开者中心](https://dev.dcloud.net.cn/)平台

![](./assets/image-18.png)



#### 1.2 云打包

**生成好的证书即可以用来打正式的 APK 包，也可以用于自定义调试基座。**

##### 1.2.1 自定义基座

![](./assets/image-11.png)

![](./assets/image-12.png)

##### 1.2.2 正式打包

![](./assets/image-13.png)

正式包与自定义调试基座的区别在于，自定义调试基座用于开发环境，正式包用于上线到安卓应用商店，如360、小米、华为应用商店等。

**注意：正式包与自定义调试基座的包名要一致！**



#### 1.3 地图服务

在 uni-app 中使用地图服务时，需要注意兼容性：

| 地图服务商 | App        | H5   | 小程序 |
| ---------- | ---------- | ---- | ------ |
| 高德       | √          | √    |        |
| Google     | 仅nvue页面 | √    |        |
| 腾讯       |            | √    | √      |

H5 和小程序平台咱们选择的是[腾讯地图](https://lbs.qq.com/)服务平台，App 咱们选择[高德地图](https://lbs.amap.com/)服务平台，自行注册高德地图开发者账号并创建应用。

##### 1.3.1 创建应用Key

![](./assets/image-14.png)

- `PackageName` 是在自定义调试基座和正式包时定义的包名
- `SHA1` 值的获取方式如下所示

```bash
# 找到你保管证书的目录中，打开命令行工具
keytool -list -v -keystore 你的证名名.keystore 
```

![](./assets/image-15.png)

##### 1.3.2 配置神领物流

![](./assets/image-16.png)

把在高德地图创建的 Key 填写到图示位置，IOS 的 key 可以省略也可以随便填写一个，然后重新打包自定义基座，打包后运行 App 项目，验证上报异常位置时是否可以打开地图。

注意：没有配置 key 的情况下打包自定义基座运行，打开地图时会报如下错误：

![](./assets/image-17.png)



### 二、uniCloud 开发

uniCloud 是 DCloud 联合阿里云、腾讯云，为开发者提供的基于 serverless 模式和 js 编程的云开发平台。提供了许多高效实用的业务解决方案，如消息推送、一键登录、实人认证等。

**注：在下列的功能开发过程中所应用到的证书即可以是 DCloud 平台的云端证书，也可以是本地自定义生成的证书，无论选择哪一种切忌不要混用！作为新手建议大家就使用 DCloud 平台的云端证书。**

#### 2.1 消息推送

消息推送是服务端管理后台主动向安装了 App 的用户发送消息功能，分为离线推送和在线推送两种。

##### 2.1.1 应用信息

完善平台信息，填写 MD5、SHA1、SHA256 信息。

![](./assets/image-21.png)

![](./assets/image-22.png)

IOS 平台信息也需要完善，只需要填写一个包名就行

![](./assets/image-23.png)



然后找到 uniPush 选择 2.0（支持全端推送）

![](./assets/image-20.png)

##### 2.1.2 离线推送

离线推送即厂商推送设置，这种方式不需要在 App 中添加代码（不引入 SDK）即可实现消息推送的功能。这种方式的弊端是要为不同品品牌的手机分别设置参数。

以华为手机为例，首先注册成为[华为开发者账号](https://developer.huawei.com/consumer/cn/agconnect)，需要完成身份认证，华为手机用户可以使用自身的华为账号登录。

个推服务平台[文档说明](https://docs.getui.com/getui/mobile/vendor/vendor_open/)

##### 2.1.3 配置神领物流

![](./assets/image-24.png)

配置完成后重新打包。

##### 2.1.4 发送消息

![](./assets/image-25.png)

没有安卓机的同学可以使用 WeTest 平台提供的[云手机](https://wetest.qq.com/products/cloud-phone)进行测试，注意选择华为品牌的手机。



#### 2.2 一键登录

一键登录是通过运营商（中国移动、中国电信、中国联通）来获取用户手机号，进而实现用户登录的功能。

##### 2.2.1 开通服务

参见[官方文档](https://ask.dcloud.net.cn/article/37965)



##### 2.2.2 配置神领物流

![](./assets/image-26.png)

配置好后重新打包，自定义基座和正式包均可，开发阶段推荐使用自定义基座。

##### 2.2.3 客户端（App）调用

在 uni-app 中通过调用 API `uni.login` 即可唤起一键登录请求授权

```javascript
uni.login({
  provider: 'univerify',
  univerifyStyle: {
    fullScreen: true,
  },
})
```

- `provider`: 指定登录服务类型，`univerify` 表示是一键登录
- `univerifystyle` 自定义授权界面的样式

1. 调用 `uni.login`

接下来我们将登录的功能整合进神领物流项目中，又由于一键登录仅能用于 App 端，所以咱们需要用到条件编译处理登录页面的执行逻辑：

```vue
<!-- pages/login/index.vue -->
<script setup>
  import { ref, reactive, computed } from 'vue'
	
  // 省略中间部分代码...

  // 切换登录类型
  function changeLoginType() {
    // #ifdef APP-PLUS
    uni.login({
      provider: 'univerify',
      univerifyStyle: {
        fullScreen: true,
      },
    })
    // #endif

    // #ifndef APP-PLUS
    tabIndex.value = Math.abs(tabIndex.value - 1)
    // #endif
  }
</script>
```

上述代码仅会在 App 端才会调用 `wx.login` 唤起一键登录授权。

2. 自定授权界面的样式

uni-app 允许开发者对授权界面有限制的定制，即对 `univerifyStyle` 属性进行配置，详细说明[参见文档](https://uniapp.dcloud.net.cn/univerify.html#%E5%AE%A2%E6%88%B7%E7%AB%AF-%E8%AF%B7%E6%B1%82%E7%99%BB%E5%BD%95%E6%8E%88%E6%9D%83)

```javascript
const { authResult } = await uni.login({
  provider: 'univerify',
  univerifyStyle: {
    fullScreen: true,
    icon: {
      path: 'static/logo.png', // 自定义显示在授权框中的logo，仅支持本地图片 默认显示App logo
    },
    authButton: {
      normalColor: '#ef4f3f', // 授权按钮正常状态背景颜色 默认值：#3479f5
      highlightColor: '#ef4f3f', // 授权按钮按下状态背景颜色 默认值：#2861c5（仅ios支持）
    },
    privacyTerms: {
      defaultCheckBoxState: false,
      uncheckedImage: 'static/images/uncheckedImage.png',
      checkedImage: 'static/images/checkedImage.png',
      termsColor: '#ef4f3f',
    },
  },
})
```



##### 2.2.4 云函数

在使用云函数之前需要开通 uniCloud 服务空间。

云函数即在云空间（服务器）上的函数，云函数中封装的逻辑其实就是后端程序的代码，比如可以执行操作数据库的操作等。

在前面的步骤中只是获得了用户的授权，并未真正拿到用户的手机号，需要服务端直接或间接的调用中国移动、中国联通、中国电信的接口才可拿到用户的手机号码，我们将这部分逻辑封装进云函数当中。

1. 创建云函数

![](./assets/image-27.png)

![](./assets/image-28.png)

默认生成的代码为：

```javascript
'use strict';
exports.main = async (event, context) => {
	//event为客户端上传的参数
	console.log('event : ', event)
	
	//返回数据给客户端
	return event
}
```

2. 调用云函数

在 uni-app 中通过 uni.callFunction 专门调用云端的函数：

```javascript
uniCloud.callFunction({
  name: '云函数名（即文件名）',
  data: {/* 给云函数传的实参 */},
  success: (result) => {
    // result 是云函数 return 的返回值
  }
})
```

**注：在 App 端调试云函数时要求手机必须与电脑处于相同的网络下。**

![](./assets/image-29.png)

在对云函数进行调试时需要安装 HBuilderX 的插件，点击命令行窗口右上角的【开启断点调试】就会自动对插件进行下载安装了。

3.  调用 `uniCloud.getPhoneNumber` 获取用户手机号

```javascript
// 在云函数中获取
'use strict'
exports.main = async (event, context) => {
  // event里包含着客户端提交的参数
  const res = await uniCloud.getPhoneNumber({
    appid: context.APPID, // 替换成自己开通一键登录的应用的DCloud appid
    provider: 'univerify',
    apiKey: 'fc7aa3fb2672f947a501f2392a22501a', // 在开发者中心开通服务并获取apiKey
    apiSecret: 'b4d9fafeb3c3ed12ff6cc97b9f9a1817', // 在开发者中心开通服务并获取apiSecret
    access_token: event.access_token,
    openid: event.openid,
  })

  console.log(res) // res里包含手机号
  // 执行用户信息入库等操作，正常情况下不要把完整手机号返回给前端
  // 在云函数中也可以调用后端接口将手机号存入后端数据中或云数据库中
  return {
    code: 200,
    message: '获取手机号成功',
  }
}
```

![](./assets/image-30.png)

如上图所示，在获取用户手机号时需要为云函数指定依赖 `uni-cloud-verify`，添加至云函数的 `package.json` 中，另外就是这获取用户手机号码是需要付费的，充值付费后才可以使用。

添加了依赖和充值后手机号码便成功获取到了

![](./assets/image-31.png)



#### 2.3 实人认证

实人认证是通过视频方式来验证用户身份的技术，uniCloud 提供了简单方便的实现方式。

##### 2.3.1 开通服务

参见[官方文档](https://uniapp.dcloud.net.cn/uniCloud/frv/service.html)



##### 2.3.2 配置神领物流

![](./assets/image-32.png)



##### 2.3.3 新增页面

为了让【实人认证】的功能更符合实际的业务，我们为神领物流新增加一个页面：

```vue
<!-- subpkg_user/verify/index.vue -->
<script setup></script>
<template>
  <uni-forms class="verify-form">
    <uni-forms-item name="account">
      <input
        type="text"
        placeholder="请输入姓名"
        class="uni-input-input"
        placeholder-style="color: #818181"
      />
    </uni-forms-item>
    <uni-forms-item name="password">
      <input
        type="text"
        placeholder="请输入身份证号"
        class="uni-input-input"
        placeholder-style="color: #818181"
      />
    </uni-forms-item>
    <button class="submit-button">认证</button>
  </uni-forms>
</template>

<style lang="scss" scoped>
  .verify-form {
    padding: 120rpx 66rpx 66rpx;
  }

  .uni-forms-item {
    height: 100rpx;
    margin-bottom: 20 !important;
    border-bottom: 2rpx solid #eee;
    box-sizing: border-box;
  }

  ::v-deep .uni-forms-item__content {
    display: flex;
    align-items: center;
  }

  ::v-deep input {
    width: 100%;
    font-size: $uni-font-size-base;
    color: $uni-main-color;
  }

  ::v-deep .uni-forms-item__error {
    width: 100%;
    padding-top: 10rpx;
    border-top: 2rpx solid $uni-primary;
    color: $uni-primary;
    font-size: $uni-font-size-small;
    transition: none;
  }

  .submit-button {
    height: 100rpx;
    line-height: 100rpx;
    /* #ifdef APP */
    padding-top: 4rpx;
    /* #endif */
    margin-top: 80rpx;
    border: none;
    color: #fff;
    background-color: $uni-primary;
    border-radius: 100rpx;
    font-size: $uni-font-size-big;
  }

  button[disabled] {
    background-color: #fadcd9;
    color: #fff;
  }
</style>
```

新建的页面还必须要到 `pages.json` 中进行配置，配置在 `subpkg_user` 这个分包下：

```json
{
  "pages": [],
  "globalStyle": {},
  "tabBar": {},
  "subPackages": [{
      "root": "subpkg_task",
      "pages": []
    },
    {
      "root": "subpkg_message",
      "pages": []
    },
    {
      "root": "subpkg_user",
      "pages": [
        ...
        
        {
          "path": "verify/index",
          "style": {
            "navigationBarTitleText": "实人认证"
          }
        }
        
        ...
      ]
    }
  ],
  "uniIdRouter": {}
}
```

页面创建好之后到 `pages/my/index` 页面中添加链接跳转：

```vue
<template>
  <view class="page-container">
    <view class="user-profile">
      ...
    </view>
    <view class="month-overview">
      ...
    </view>
    <view class="entry-list">
      <uni-list :border="false">
        ...
        <!-- #ifdef APP-PLUS -->
        <uni-list-item
          to="/subpkg_user/verify/index"
          showArrow
          title="实人认证"
        />
        <!-- #endif -->
      </uni-list>
    </view>
  </view>
</template>
```

##### 2.3.4 客户端（App）调用

1. 获取设备信息

```vue
<script setup>
  function onFormSubmit() {
    // 1. 获取设备信息
    const metaInfo = uni.getFacialRecognitionMetaInfo()
    console.log(metaInfo)
  }
</script>
<template>
  <uni-forms class="verify-form">
		...
    <button @click="onFormSubmit" class="submit-button">认证</button>
  </uni-forms>
</template>
```

2. 创建云函数获取 `certifyId`

![](./assets/image-33.png)

![](./assets/image-34.png)

![](./assets/image-35.png)

创建好云函数之后，添加如下代码：

```javascript
'use strict'
exports.main = async (event, context) => {
  // 云函数获取实人认证实例
  const frvManager = uniCloud.getFacialRecognitionVerifyManager({
    requestId: context.requestId,
  })

  // 云函数提交姓名、身份证号以获取认证服务的certifyId
  const result = await frvManager.getCertifyId({
    realName: event.realName,
    idCard: event.idCard,
    metaInfo: event.metaInfo,
  })

  //返回数据给客户端
  return result
}
```

在调用云函数时需要 App 客户端传入 3 个参数：

- `realName` 验证用户的真实姓名
- `idCard` 验证用户的身份证号
- `metaInfo` 设备信息

3.  客户端 App 调用云函数

```vue
<script setup>
  import { ref } from 'vue'
  
  const realName = ref('用户姓名')
  const idCard = ref('用户身份证号')
  
  function onFormSubmit() {
    // 1. 获取设备信息
    const metaInfo = uni.getFacialRecognitionMetaInfo()
    // 2. 调用云函数
    uniCloud.callFunction({
      name: 'uni-verify',
      data: { metaInfo, realName: realName.value, idCard: idCard.value },
      success() {},
      fail() {}
    })
  }
</script>
<template>
  <uni-forms class="verify-form">
		...
    <button @click="onFormSubmit" class="submit-button">认证</button>
  </uni-forms>
</template>
```

4. 开始验证

```vue
<script setup>
  import { ref } from 'vue'
  
  const realName = ref('用户姓名')
  const idCard = ref('用户身份证号')
  
  function onFormSubmit() {
    // 1. 获取设备信息
    const metaInfo = uni.getFacialRecognitionMetaInfo()
    console.log(metaInfo)
    // 2. 调用云函数
    uniCloud.callFunction({
      name: 'uni-verify',
      data: { metaInfo, realName, idCard },
      success({ result }) {
        // 3. 客户端调起sdk刷脸认证
        uni.startFacialRecognitionVerify({
          certifyId: result.certifyId,
          success() {
            uni.utils.toast('认证成功!')
          },
          fail() {
            uni.utils.toast('认证失败!')
          },
        })
      },
      fail() {}
    })
  }
</script>
<template>
  <uni-forms class="verify-form">
		...
    <button @click="onFormSubmit" class="submit-button">认证</button>
  </uni-forms>
</template>
```






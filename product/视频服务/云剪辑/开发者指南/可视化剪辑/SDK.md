# 云剪辑前端 sdk

## 概述

云剪辑 SDK 主要用于协助用户完成云剪辑 WEB 端开发，合成自己的轨道编辑数据等。_本文假定用户已经做好相关准备,应用服务已经就绪，如果没有请移步[接入文档](后台文档地址)查阅详情_。

> sdk 主要包含 3 块：

1. [协商配置和初始化组件,生命周期管理](#开始配置)。
2. [组件通信协议](#与组件通信)。
3. [用户事件监听抛出](#事件管理)。

## 安装

> 使用我们的 CDN 引入 sdk，`<script src="https://imgcache.qq.com/qcloud/cme/sdk_v1.js"></script>`。

## 开始配置

配置主要内容是身份校验和 云剪（CME） 组件应该出现位置以及相关操作。

## 定制页面

页面可以做*基础定制*，目前只能我们来协助操作，后续会开放自助。
*基础定制*主要提供以下能力:

- 设置页面元素隐藏和显示。
- 定制页面元素呈现形态（仅支持替换 Logo 图标,修改部分文案的小幅修改）。

以下介绍初始化过程：

```js
/* @init  初始化方法,没调用该方法之前,调用 send 方法通信无效。
 *    @param el {{HTMLELement}}  html元素,做为父容器承载 CME 组件。
 *    @param options {{ object|hash }} 参数。
 *         @param signature {{ string }} ，该项目签名.具体参考 签名算法。
 *         @param onload {{ funcion }}  iframe 加载成功回调, 加载成功后可以调用 send 方法使用具体功能。
 *         @param onerror {{ function}}  iframe 加载失败的回调。
 *    @return 返回一个 cme 组件实例。
 **/
let cme = CME.init(document.getElementById("Container"), {
  signature: "your_signature",
});

/**
 * 页面初始化异常时，会抛出Error事件。
 **/
cme.on("Error", (data) => {
  console.log("Error", data);
});

/**
 * 页面组件完全准备好以后，会抛出：Editor:Ready 事件，
 * 该事件触发以后，所有命令字会正常响应，触发之前，部分命令字无法响应。
 **/
cme.on("Editor:Ready", () => {
  console.log("desc", "iframe Ready");
  /**
   *
   * @send 发送命令子方法,
   *    @param cmd {{string}} 字符串,参考 cme 命令字列表。
   *    @param param {{object}} 命令入参。
   *    @param callback {{function}} 完成命令的回调函数。
   *
   **/
  cme.send("syncFusionData", {}, (data) => {
    console.log("sync done", data);
  });
});

let clickHandler = () => {
  console.log("Click");
};
/**
 *
 * @on 监听方法,监听来自 CME 的用户事件。
 *    @param eventName {{string}} 字符串,参考 cme 自定义用户事件。
 *    @param callback {{function}} 回调函数。
 *
 */
cme.on("Editor:MoreResourceBtn:click", clickHandler);

/**
 * @off 取消监听方法,监听来自 CME 的事件,如果什么都不传则会清理所有监听事件。
 *    @param eventName {{string}} 字符串,参考 cme 自定义用户事件。
 *    @param callback {{function}} 对应回调函数。
 *
 */
cme.off("Editor:MoreResourceBtn:click", clickHandler);

/**
 * @destroy 清理所有监听事件、临时变量并将对象销毁。
 *  注意，此时cme对象引用会变为空。
 */
cme.destroy();
```

## 事件监听

> 以上初始化例子里已经体现云剪辑前端的所有的数据交换方式.发送命令字，监听事件，取消监听等。
> 事件主要有三种类型:

1. 异常事件[查看](#Error)。
2. 生命周期事件[查看](#生命周期事件)。
3. 用户交互事件[查看](#用户交互事件)。

## 与组件通信

> 云剪组件用 iframe 的方式嵌入到应用，采用 postMessage 方式与父容器通信，_所以通信必须在 iframe 加载成功以后可用（init 的 onload 方法执行以后），否则无效或报错_。

> 云剪组件会对特定命令字和参数响应，具体命令字和功能参考[命令字列表](#命令字列表)。

## 附录

### Error

| 事件名 | 数据结构                                                 |
| ------ | -------------------------------------------------------- |
| Error  | {error:1000,msg:'NotSupportBrowser'}，更多错误码见下表。 |

|错误码| 意义|
|1000| 不支持的浏览器，_目前只支持 Chrome 60 版本以上的浏览器_ 。|

### 生命周期事件

| 事件名            | 数据结构 | 说明                                 |
| ----------------- | -------- | ------------------------------------ |
| Editor:Ready      | null     | 编辑页已成功加载，等待数据交换。     |
| ProjectList:Ready | null     | 项目列表页已成功加载，等待数据交换。 |

### 用户交互事件

用户事件命名规则,可以使用控制台查看元素，元素上标有`data-ui`就会响应`click`,该事件抛出名字为:`${data-ui}:Click`，`data-ui` 以组件/页面，组件的方式命名,参考`Editor:ExportBtn:Click`可知道是编辑页导出按钮点击事件,具体抛出事件如下:

| 事件名                         | 数据结构 | 说明                                              |
| ------------------------------ | -------- | ------------------------------------------------- |
| Header:BackBtn:Click           | null     | 用户点击顶部返回按钮。                            |
| Editor:ExportBtn:Click         | null     | 用户点击导出按钮 [编辑页](#参考页面地址)。        |
| Header:Logo:Click              | null     | 用户点击 Logo。                                   |
| Header:Goto:Click              | null     | 用户点击顶部跳转按钮。                            |
| Assets:CreateClassBtn:Click    | null     | 用户点击创建分类按钮[素材库](#参考页面地址)。     |
| Editor:MoreResourceBtn:Click   | null     | 用户点击更多资源按钮[编辑页](#参考页面地址)。     |
| Editor:UploadBtn:Click         | null     | 用户点击上传按钮 [编辑页](#参考页面地址)。        |
| Project:CreateProjectBtn:Click | null     | 用户点击创建项目按钮[项目列表页](#参考页面地址)。 |

### 可配置组件

目前可用组件,使用控制台查看元素，元素上标有`data-ui`就表示该组件名称，配置时可用，具体有如下:

| 组件名                 | 描述                                  |
| ---------------------- | ------------------------------------- |
| Header:BackBtn         | 顶部返回按钮。                        |
| Editor:ExportBtn       | 导出按钮[编辑页](#参考页面地址)。     |
| Header:Logo            | 顶部 Logo。                           |
| Editor:MoreResourceBtn | 更多资源按钮[编辑页](#参考页面地址)。 |
| Editor:UploadBtn       | 上传按钮 [编辑页](#参考页面地址)。    |

### 命令字列表

| 可用范围                | 命令字           | 参数                                                                                           | 功能                                                                                                                                                                   |
| ----------------------- | ---------------- | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [编辑页](#参考页面地址) | refreshResources | mids: 素材 id 列表数组(可选)                                                                   | 更新素材,如果不什么都不传,会将素材刷新到为第一页（20 条数据）,如果传入素材 ID，会去尝试将刷新传入素材 ID 到顶部。                                                      |
| [编辑页](#参考页面地址) | syncFusionData   | 无                                                                                             | 提交编辑数据，如果在外部触发导出项目事件，需要调用一次该接口。                                                                                                         |
| [编辑页](#参考页面地址) | showUpload       | sign: 上传签名                                                                                 | 开启上传面板，**在云剪内部上传会使用云剪的即传即用功能，即浏览器支持度较好的媒体格式，如 .h264 的 mp4 视频和 webm，可边上传边编辑， 不用等待完全上传和解码的全过程**。 |
| 所有页面                | routeGoto        | name: 路由名目前支持`Editor`,`MyAsset`,`AssetMarket`,projectId: 项目id，必须指定跳转具体项目Id | 跳转指定页面                                                                                                                                                           |


### 参考页面地址

可以到 [云剪辑简介](./yj_intro.md) 里查看页面效果.

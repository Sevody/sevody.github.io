---
title: 房多多 Hydrofoil 跨端容器
date: 2021-07-10 14:22:30
tags: hybrid
---

Hydrofoil 跨端容器是房多多大前端团队为了实现同时使用 Web、Flutter、Native 三种技术而开发的底层框架。通过引入 Hydrofoil 跨端容器，一个 App 能很简单的实现 Web 和 Flutter 跟 Native 间的通信，并通过各端预定义的 SDK，业务端(由 Web 或 Flutter 实现)很容易获得 Native 端的底层能力。

<!--more-->

## 1. 背景

随着 iOS 和安卓两个平台开发范式的稳定和组件功能的沉淀，出于开发效率和动态化的需求，降低试错成本，跨端运行的混合开发模式一直在发展，从 Webview 容器 到 React Native 再到 Flutter，大前端的概念也渐渐兴起。

### 1.1 Webview 容器

Webview 容器技术是指将 Web 技术与 Native 技术结合起来，基于 Web 技术来实现页面和业务功能，将操作系统底层的 API 通过封装的**桥协议（JSBridge）**暴露给 JavaScript 调用，Web 编写的代码可以在原生提供的 Webview 容器里运行起来。这种模式的好处是能复用已有的 Web 页面，极大的降低开发成本，更新也不需要发版。但是原生的 Webview 组件渲染性能不高，对于交互比较复杂的场景，用户体验难以保证。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled.png)

### 1.2 React Native

2013 年，Facebook 推出了 React 前端框架，它是一个为数据提供渲染为 HTML 视图的开源 JavaScript 库。React 采用了**虚拟 DOM** 的方式来分离数据逻辑和实际渲染，在这种渲染模式启发下，2015 年，Facebook 又推出了 React Native。React Native 和 Webview 容器不同之处在于把实际渲染放到了原生来实现。通过事先定义的 DSL （React 的 JSX 语法），数据逻辑变动后，会生成一个虚拟 DOM 树，然后由 Native 端解析和映射并渲染成平台的组件。这种方案对于体验会有一定的提升，不过在一些极端的场景（比如无限滚动列表）还是有一定的性能问题。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%201.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%201.png)

### 1.3 Flutter

2018 年 12 月初，Google 推出了 Flutter 1.0 稳定版，它是通过 Dart 语言构建的一套跨平台开发组件。与 Webview 容器、React Native 不同的是，它没有使用 Webview 或者系统平台自带的原生控件，而是有一套自己的 Widget，所有组件都是基于自建的 Skia 渲染引擎自绘，性能上能与 Native 平台的 View 相媲美。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%202.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%202.png)

房多多的前端混合开发历程也经历了 Webview 容器阶段、尝试过 React Native，并处在 Flutter 实际使用阶段，在不同阶段都遇到了一些需要解决的问题。而 Webview 和 Flutter 两种模式在 App 中同时存在，也导致了跨端调用出现了一定的混乱和重复开发。正如混合开发是为了实现 `Write Once，Run Both（iOS & Android）` ， 为了践行`Add One, Run Both（Webview & Flutter）` ，Hydrofoil 跨端容器应运而生。

## 2. Hydrofoil 容器

### 2.1 什么是 Hydrofoil 容器

> Hydrofoil 即水翼船。这种船底部装上了类似翅膀的部件，使船体行驶时可以离开水面，快速航行。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%203.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%203.png)

Hydrofoil 容器是为了实现 Web、Flutter、Native 三端间的相互调用开发出来的底层框架。通过引入 Hydrofoil 跨端容器，一个 App 能很简单的实现 Web 和 Flutter 跟 Native 间的通信，并通过各端预定义的 SDK，业务端（由 Web 或 Flutter 实现）很容易获得 Native 端的底层能力。

### 2.2 为什么需要 Hydrofoil 容器

前面提到，我们房多多前端团队在实践混合开发的时候，遇到了一些问题，其中最突出的问题是平台内部各个 App 用的混合技术栈不统一，有的 App 用到了 Flutter，有的没有用，有的用了不同版本的 Webview 容器等等。而如果使用一个统一的跨端容器，不但有利于当前 App 技术栈的统一，降低维护成本，也能让新 App 快速接入混合开发技术，并高度复用已有代码。比如我们最新开发的**房云 App**，因为需要在短周期内快速上线，我们接入了 Hydrofoil 容器后，能直接复用已有的 Flutter、Web 代码，甚至因为容器打通了 Native、Web、Flutter 三端，能在保证用户体验的基础上，只使用 Web、Flutter 开发新的业务页面，实现了 App 的快速上线，正如加上了翅膀的水翼船（Hydrofoil）一般，快速航行（上线）。

## 3. 架构设计

### 3.1 整体架构

Hydrofoil 跨端容器以 Hydrofoil Core 为核心，实现了底层的消息分发和响应回调，并实现了路由的统一调度。Native SDK 提供了底层能力的实现，并通过 Web SDK 和 Flutter SDK 为业务层提供可调用的 API。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/hydrofoil_architecture.jpg](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/hydrofoil_architecture.jpg)

### 3.2 Hydrofoil Core

Hydrofoil Core 提供了混合开发的**_运行环境_**、**_路由调度_**和**_底层通信_**。它主要包含三个部分：**FddWebview**、**FddFlutter**、**调度中心**。

- **FddWebview** 模块提供了房多多内部定制的 Webview 容器，来由它来加载 Web 页面，并实现与 Native 端的通信，是实现 Web 与 Native 混合开发技术的核心。
- **FddFlutter** 模块提供了 Flutter 容器，它内置了 Flutter SDK，并且封装了通信等相关模块，使得 App 接入 Hydrofoil 容器后，能直接使用 Flutter 技术进行开发。
- **调度中心**模块主要负责统一 Webview 和 Flutter 容器的路由管理和 API 调用，是与外部 SDK 的通信的统一出入口。

### 3.3 SDK

各端都有对应的 SDK 实现：

- **Native SDK** 负责提供操作系统底层能力，比如网络、设备信息等。也可以对一些第三方功能如上传、IM 等进行一定程度的收敛。
- **Web SDK** 作为 Webview 容器和业务方的 Web 代码之间通信的胶水层，并为 Web 端提供对应的语言化接口
- **Flutter SDK** 主要是对 Flutter 可以调用的底层系统能力的统一收敛，为 Flutter 端提供可直接调用的 API 接口。

### 3.4 扩展性

前端近些年发展迅速，跨平台方案也在大量涌现，例如市占较高的 Weex、RN、UniApp、mPaaS 等。由于 Hydrofoil 将各平台容器以及路由和通信细节封装起来，业务侧各技术栈的交互均可通过提供的自定义协议进行，做了很好的解耦，故需要引入新的跨平台方案时，只需在 Hydrofoil 内进行容器通信及 SDK 的扩展即可，业务侧几乎无需做任何变更即可进行跨平台的调度。

## 4. 模块设计

### 4.1 Hydrofoil Core 模块

Hydrofoil Core 对外提供了 5 个最基本的方法：`init`、`open` 、`close`、`call`、`register`。其中 `init` 方法主要是对容器做一些初始化工作，比如注入支持的协议、配置 ua 标识等等；`open` 和 `close` 方法负责路由调度，打开或者关闭指定的路由；`call` 和 `register` 方法负责三端 SDK 方法的注册和调用。

上面的 5 个基本方法，Hydrofoil Core 通过在不同维度封装不同的逻辑模块来实现：

- 容器顶层是 **HyWebviewContainer** 和 **HyFlutterContainer**，是 Webview 容器和 Flutter 容器的载体。`init` 方法提供的参数会落到 **HyWebviewContainer** 和 **HyFlutterContainer** 内部执行具体的初始化流程。
- 路由调度统一收敛到了 **HyRouterComponent**，根据 `open` 方法传入的**协议类型（protocol）HyRouterComponent** 会决定具体调用哪种类型的 controller/activity 进行加载、渲染。
- **HyHandlerComponent** 模块统一管理各个容器的 API 调用和底层通信，`call`、`register` 被调用后，由 **HyHandlerComponent** 把消息传递到容器各自的 Channel。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%204.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%204.png)

### 4.2 SDK 模块

为了给业务端提供底层能力，Hydrofoil 容器会提供对应功能的 API 给业务方调用。但是 API 的开发和实现不是一蹴而就的，会随着业务需求的变化而更迭版本，各端的 API 可能会有不同程度的变化。为了避免命名的冲突和保证功能的唯一性，我们使用唯一的 **id** 来作为三端 API 共同的标识。比如，打开指定路由的 id 标识是 1001、关闭路由的标识是 1002。

但是这样的方式会带来一些问题：业务方每次使用容器 API 都需要去翻阅文档，看看对应的 id 代表什么功能，代码可读性太差。为了解决这个问题，Web SDK 和 Flutter SDK 会收敛这些 id，并提供更语义化的 API 供业务方调用。而在三端的 SDK 内部，会维护一份类似这样的表格对 API 进行管理：

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/web_protocol.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/web_protocol.png)

当业务方调用 `router.open` 方法时，Web SDK 会发送一个 id 为 **1001** 消息给 Native SDK，Native SDK 检测到 **1001** 这个 id 的消息后，调用 `Hy.open` 方法打开消息里指定的路由页面。而当需要新增 API 时，只需三端约定一个统一的 id，然后在表格后添加对应的方法注释。

```jsx
// router.js 路由模块
const router = {
  open: [
    {
      handler(options) {
				// 把三端约定的id传入Bridge
        return this.invoke(1001, options);
      },
    },
  ],
	// ...
};

// bridge.js
class Bridge {
	// ...
	const init = () => {
		// 动态解析模块方法
	  const ext = Extension(Bridge);
	  ext.gen('event', alias.event);
	  ext.gen('router', alias.router);
		// ...
	};

	// 收敛底层调用入口
	async invoke(type, data = {}, additional = {}) {
		// ...
	  try {
			// 获取桥实例
	    const bridge = await connect();
			// 通过 Promise 返回给调用方
	    return await new Promise((resolve) => bridge.callHandler('jsInvokeNative', dataJson, (result) => resolve(result)));
	  } catch (e) {
	    // ...
	  }
	}
	// ...
}

// 使用方
// demo.js
import jsbridge from '@fdd/hydrofoil-jsbridge';

// 通过调用 webSDK 提供的语义化方法，打开新路由
jsbridge.router.open({
  url: 'xxxxx',
});
```

## 5. 通信实现

Hydrofoil 容器内部有 Hydrofoil Core 模块和各端对应的 SDK 模块，而各个模块可能运行在不同的代码引擎中，它们间方法的调用，需要使用特定的通信方式来实现。

### 5.1 Web 与 Native 通信

实际上，Web 和 Native 之间有 2 种方式实现通信：

1. 无论是 iOS 还是 Android，提供的 Webview 容器是可以拦截一切 Web 端发起的请求的，无论是标准协议（如 http://、https:// 等）还是私有协议（如 weixin:// ）。基于这个原理，Web 采用私有协议模拟发起 URL 请求，Native 解析这类 URL 并定制相应的处理函数，这就实现了 Web 与 Native 的通信。
2. 在 Native 的开发中，开发者可以给 Webview 容器注入全局变量并挂载在 `window` 对象上，这样前端的 JavaScript 代码就可以通过 `window` 上全局对象方法来调用事先注入的函数，实现通信。

而在 Hydrofoil 容器，为了实现 Web 和 Native 的双向调用，同时使用了上面的两种方式，底层的 iOS 和 Android 都使用了一套统一的 **WebViewJavascriptBridge** 通信实现。

首先，Web 端定义了 4 个方法和 3 个数据结构：

```jsx
// WebViewJavascriptBridge_JS.m
window.WebViewJavascriptBridge = {
  registerHandler: registerHandler, // 注册供 Native 调用的 JS 方法
  callHandler: callHandler, // 调用 Native 端的方法
  _fetchQueue: _fetchQueue, // 供 Native 调用，取出 sendMessageQueue 的数据
  _handleMessageFromObjC: _handleMessageFromObjC, // 供 Native 调用，把通信消息传给 Web
};

// ...

var sendMessageQueue = []; // 保存要传给 Native 的通信消息
var messageHandlers = {}; // 保存注册的 JS 方法
var responseCallbacks = {}; // 保存回调方法
```

同时，Native 端也定义了 2 个对应的方法和 2 个数据结构：

```objectivec
// WKWebViewJavascriptBridge.m
@interface WKWebViewJavascriptBridge : NSObject<WKNavigationDelegate, WebViewJavascriptBridgeBaseDelegate>
// 注册供 JS 调用的 Native 方法
- (void)registerHandler:(NSString*)handlerName handler:(WVJBHandler)handler;
// 调用 JS 端的方法
- (void)callHandler:(NSString*)handlerName data:(id)data responseCallback:(WVJBResponseCallback)responseCallback;
@end

// WebViewJavascriptBridgeBase.m
- (id)init {
    if (self = [super init]) {
        self.messageHandlers = [NSMutableDictionary dictionary]; // 保存注册的 Native 方法
        self.responseCallbacks = [NSMutableDictionary dictionary]; // 保存回调方法
    }
    // ...
}
```

以 Web 调用 Native 定义的 `getUser` 方法来获取 App 的用户信息来进行登录操作为例，具体的调用逻辑如下图：

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%205.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%205.png)

实际上，Web 传递消息给 Native 的时候，只是通过加载地址为 https://wvjb_queue_message 的 iframe 来告知 Native 有新的消息，具体的信息存到了 **sendMessageQueue** 队列里，每个调用的回调都是通过唯一的 id 来识别的（callbackId 和 responseId），以此来保证回调消息的顺序。

### 5.2 Flutter 和 Native 通信

对于页面路由，我们使用了 **FlutterBoost** 来提供底层的路由映射能力，而对于 Flutter 与 Native 间的事件通知和方法调用，使用了 **Method Channel** 进行通信，并在此基础上做了一定程度的封装。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%206.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%206.png)

```dart
// fdd_channel.dart
class FddChannel {
  // 初始化MethodChannel，通过该channel统一管理所有的事件通知&方法调用
  final MethodChannel _methodChannel = MethodChannel("fdd_channel");

  // 管理事件监听
  final Map<String, List<EventListener>> _eventListeners = Map();

  // 管理方法调用
  final Set<MethodHandler> _methodHandlers = Set();

  FddChannel() {
    _methodChannel.setMethodCallHandler((MethodCall call) {
			// 遍历methodHandler集合，处理native端到flutter端的方法调用
			// ...
    });
  }

  void sendEvent(String name, Map arguments) {
    // flutter端通过methodChannel向native发送事件
		// ...
  }

  VoidCallback addEventListener(String name, EventListener listener) {
		// 添加事件监听，用于处理native端发送到flutter端的事件
		// ...
  }

  // ... ...

  Future<T> invokeMethod<T>(String method, [dynamic arguments]) async {
    // flutter端通过methodChannel调用native端的方法
		// ...
  }

  VoidCallback addMethodHandler(MethodHandler handler) {
		// 处理native端到flutter端的方法调用
		// ...
  }
}
```

## 6. 集成

Hydrofoil 容器的目标是让业务 App 通过简单的接入流程后，快速获得 Webview 容器和 Flutter 的混合开发支持，同时保持较高可维护性，这在 iOS 和 Android 两端的工程上都面临一定的挑战。

### 6.1 iOS 端集成

为了将 Flutter 相关的能力直接集成在容器内部，需要将 **Flutter SDK** 和 **FlutterBoost** 通过依赖的方式引入到容器中。官方 iOS [混编方案](https://flutter.dev/docs/development/add-to-app) 是使用 CocoaPods 和 Flutter SDK 集成，实现了无缝开发，一旦配置好可以在 Flutter 工程内开发、无缝同步到 Native 侧。但是这个方案存在下面几个问题：

- 耦合严重，需要修改原有 Native 工程的配置、添加特定脚本去编译 Flutter
- 会修改原有 Pod 的 xcconfig 配置
- 所有成员均需要成功配置好 Flutter 环境才能编译成功

所以我们在官方的混编方案上做了工程上的优化，通过一个自动化脚本，将它们的产物 **\*.framework** 引入到 Hydrofoil 中。

实际上，iOS 原生工程依赖了以下 3 部分 Flutter module 生成的 framework：

- Flutter.framework、flutter_boost.framework 被包含在 Hydrofoil 中，接入后一般不用动，除非是 Flutter 和 FlutterBoost 版本更新了；
- App.framework：我们编写的 dart 代码，每次修改 Flutter module 里的代码都需要更新原生工程里的 App.framework；
- FlutterPluginRegistrant.framework 和 Flutter 第三方插件 framework（排除 flutter_boost.framework）：每次对 Flutter module 里的 pubspce.yaml 文件里的依赖进行更新、增加、删除操作都需要将原生工程里本地库（如 FlutterThirdPartyFrameworks）同步更新后，执行 `pod install`。

![https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%207.png](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/%E6%88%BF%E5%A4%9A%E5%A4%9A%20Hydrofoil%20%E8%B7%A8%E7%AB%AF%E5%AE%B9%E5%99%A8%208ab9d3b7660642b58faf4bf71e64dabd/Untitled%207.png)

这样一来，我们在原生工程中通过 CocoaPods 引入 Hydrofoil 这个私有库，使用自动化脚本将 Flutter 的业务代码编译成 App.framework 并拷贝到原生工程的目录中，即可编译运行。

### 6.2 Android 端集成

在 Android 工程中直接引用 Flutter 工程会导致编译速度非常慢，而且必须安装了 Flutter 环境才能正常运行，为了解决这些问题，我们将 Flutter 工程打成 **AAR** 包，然后在 Android 工程中引用。而生成 AAR 包有两种方案：

1. **生成一个 AAR：**
   - FlutterBoost 通信模块作为 Flutter 部件发布到 pub 私库
   - Flutter 业务模块直接引入 1 的 pub 私库产物
   - 开发完毕 Flutter 业务模块打包成 Native 需要的 AAR，Android 需要初始化 FlutterBoost，接收消息驱动路由分发
2. **生成两个 AAR：**
   - FlutterBoost 通信模块包装成独立的 AAR
   - Flutter 业务代码打包成独立的 AAR
   - Native 端要初始化 FlutterBoost 的渠道

经过评估，在方案二中，通信模块可以作为基础部件下沉到项目底层，业务层 AAR 作为业务 module 存在，业务 module 变更不会干涉到通信模块，完全解耦，所以我们采用了**生成两个 AAR** 的方案。

要实现方案二，首先要抽离通信层，我们通过利用 build.gradle 配置文件对项目代码进行编译打包，避免资源文件冗余。关键流程分三步：

- 新建 Flutter 插件工程 flutter_route，在 dart 模块中添加 FlutterBoost 依赖并封装 Flutter 端的通信逻辑, 在 Android 模块中封装 Native 端通信逻辑，通过 build.gradle，将 Android 模块的代码，FlutterBoost 代码以及 Flutter SDK 的代码打包，生成 flutter_route_native.aar
- 新建 Flutter 工程（业务模块）， 添加 Flutter 插件 flutter_route 的依赖，实现 Flutter 相关业务代码，通过 build.gradle, 移除 FlutterBoost 代码以及 Flutter SDK 的代码，将 Flutter 业务代码以及其他依赖打包生成 flutter_module.aar
- 在 native 工程中，引入 flutter_route_native.aar，添加通信层依赖； 引入 flutter_module.aar，可以将 Flutter 相关业务添加到 Native 工程中； 通过调用 flutter_route_native.aar 中的 API，native 可以实现和 flutter 业务之间的通信

实际上，上面的工程逻辑，我们全部收敛到了 Hydrofoil 容器内，对环境依赖的安装和使用流程做了闭环，实现了开箱即用。

## 7. 落地结果

目前房多多平台的房云 App、多多新房 App、多多卖房 App、房多多 App 都已经接入了 Hydrofoil 跨端容器，特别是房云 App 项目大量使用了 Hydrofoil 容器提供的混合开发模式，不同场景应用不同的跨端技术，保障用户体验同时提高了开发效率，为 App 的快速上线提供了有力支持。

## 8. 总结与展望

混合开发带来的开发效率的提升是毋庸置疑的，而随着业务的发展，工程的复杂度不断上升，维护成本上升等问题也为我们带来一定的挑战。通过构建一个统一的跨端容器，打造大前端底座，充分利用跨端技术赋能业务，为我们解决面临的动态化、多端复用、维护成本等问题。未来，我们还会进一步完善跨端容器的生态建设，优化开发体验，基于大前端融合，探索更多跨端技术的可能性。

和原生开发相比，React Native 最明显的不足就是页面的渲染速度，比如页面加载慢，渲染的效率低等。对于这些问题，都是开发中常见的问题，也是使用React Native 开发跨平台应用时必须优化的点，由此引入一个问题，React Native的性能优化究竟应该如何做？

相信对于这个问题，大多数人第一眼看到后都是很懵逼的。因为大多数人除了业务开发之外，对于React Native原理性的东西都了解甚少。其实，经过我们多年的经验，一个未经优化的 React Native 应用，从大体上讲可以分为 3 个瓶颈：\
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f93352989f7414fbb1d54ea290ca54f~tplv-k3u1fbpfcp-zoom-1.image" />

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4684cda5394c42b8bd9293a00ac6dfb8~tplv-k3u1fbpfcp-zoom-1.image" />

当然，RN的性能优化包括JavaScript 侧和原生容器的优化。不过，我们今天我们主要站从客户端角度进行优化。

## 一、React Native 环境预创建

在 最新的React Native 架构中，Turbo Module (新架构下的通信方式)是按需加载，而旧框架则是在初始化的时候把Native Modules一股脑的加载进来，同时 Hermes 引擎放弃了 JIT，在启动速度方面也有明显提升。如果对React Native新架构感兴趣的同学，可以参考：React Native新架构。

抛开这两个版本在框架方面的优化，在启动速度方面，我们还能做些什么呢？首先，我们看一下React Native 环境预创建。在混合工程中，React Native 环境与加载页面的关系如下图。\
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/772d669fe9d14936ad1ec28817546634~tplv-k3u1fbpfcp-zoom-1.image" />

从上图中可以看到，在混合应用中，独立的 React Native 载体页都有自己独立的执行环境。Native 域包括 React View、Native Modules；JavaScript 域泽包括 JavaScript 引擎、JS Modules、业务代码；中间通信使用的是Bridge/JSI（新版本）。

当然，业内也有多个页面复用一个引擎的优化。但是多页面复用一个引擎存在一些问题，比如 JavaScript 上下文隔离、多页面渲染错乱、JavaScript 引擎不可逆异常等。而且复用的性能不稳定，考虑到投入产出比、维护成本等方面，通常在混合开发中，采用的是一个载体页一个引擎。

通常，一个 React Native 页面从加载渲染到展示大致分为以下几步：【React Native 环境初始化】 -> 【下载/加载 bundle】 -> 【执行JavaScript 代码】。

环境初始化这一步主要包含的工作包括：创建 JavaScript 引擎、Bridge、加载 Native Modules（旧版）。根据我们的测试，初始化这一步在 Android 环境中是特别耗时的。所以，我们想到的第一个优化点就是提前将 React Native 环境创建好，流程如下。\
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1050334146c44953a1f3d4190d3a3416~tplv-k3u1fbpfcp-zoom-1.image" />

涉及的代码如下：\
RNFactory.java

```
public class RNFactory {
    // 单例
    private static class Holder {
        private static RNFactory INSTANCE = new RNFactory();
    }

    public static RNFactory getInstance() {
        return Holder.INSTANCE;
    }

    private RNFactory() {
    }

    private RNEnv mRNEnv;
    
    //App启动时调用init方法，提前创建RN所需的环境
    public void init(Context context) {
        mRNEnv = new RNEnv(context);
    }
    
    //获取RN环境对象
    public RNEnv getRNEnv(Context context) {
        RNEnv rnEnv = mRNEnv;
        mRNEnv = createRNEnv(context);
        return rnEnv;
    }
}
```

RNEnv.java

```
public class RNEnv {
   private ReactInstanceManager mReactInstanceManager;
   private ReactContext mReactContext;
   
   public RNEnv(Context context) {
       // 构建 ReactInstanceManager
       buildReactInstanceManager(context);
       // 其他初始化
       ...
   }
   
   private void buildReactInstanceManager(Context context) {
      // ...
      mReactInstanceManager = ...
   }
   
   public void startLoadBundle(ReactRootView reactRootView, String moduleName, String bundleid) {
      // ...
   }
}
```

在做预创建时，我们需要注意线程同步问题。在混合应用中，React Native 由应用级变成页面级使用，所以在线程安全这方面有不少的问题，预创建时会并发创建多个 React Native 环境，而 React Native 环境内部构建存在异步处理，一些全局的变量，如 ViewManagersPropertyCache。

```
class ViewManagersPropertyCache {
    private static final Map<Class, Map<String, ViewManagersPropertyCache.PropSetter>> CLASS_PROPS_CACHE;
    private static final Map<String, ViewManagersPropertyCache.PropSetter> EMPTY_PROPS_MAP;

    ViewManagersPropertyCache() {
    }
    ...
}
```

内部的 CLASS_PROPS_CACHE、EMPTY_PROPS_MAP 都是非线程安全的数据结构，并发时可能会存在 Map 扩容转换问题 ，又比如 DynmicFromMap 也有此问题。\
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bc7579957bd4c608165e1187d994c72~tplv-k3u1fbpfcp-zoom-1.image" />

## 二、异步更新

原先我们进入 React Native 载体页后需要先下载最新的 JavaScript 代码包版本，若有更新，就要下载最新的包并加载。在这个过程中，我们会经历两次网络请求，即获取是否有更新，如果有下载热更新的bundle包。如果用户网络比较差，下载bundle包就会很慢，最终等待时间也会较长。

所以我们针对部分特殊的页面，采取了异步更新的策略。异步更新策略的主要思路为在进入页面之前选择性地提前下载 JavaScript 代码包，进入载体页后再看 JavaScript 代码包是否有缓存，如果有，我们就优先加载缓存并渲染；然后再异步检测是否有最新版本的 JavaScript 代码包，如果有，下载到本地并进行缓存，再等下次进入载体页时生效。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40f4e43f7468451ca54503dbe21e96af~tplv-k3u1fbpfcp-zoom-1.image" />

上图展示了我们打开一个RN页面所需要经历的一些流程。流程图中可以看出，我们从进入载体页到渲染页面，需要两次网络请求，不管网速快还是慢，这个流程算是比较漫长的，但在进行异步更新后，我们的流程就会变成下图这样
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f049868bca846a0b80c2d1aaad6bf3e~tplv-k3u1fbpfcp-zoom-1.image" />

在业务页面中，我们可以对 JavaScript 代码包进行提前下载并缓存，在用户跳转到 React Native 页面后，检测是否有缓存的 JavaScript 代码包，如果有我们就直接渲染页面。这样就不需要等待版本号检测网络接口以及下载最新包的网络接口，也不依赖于用户的网络情况，减少了用户等待时间。

在渲染页面的同时，我们通过异步检测 JavaScript 代码包的版本，若有新版本就进行更新并缓存，下次生效。当然，业务也可以选择更新完最新包之后，提示用户有新版本页面，以及是否选择刷新并加载最新页面。

## 三、接口预缓存

在经过React Native 环境初始化、bundle 加载流程进行优化后，我们的 React Native 页面基本就可以达到秒开级别了。不过，React Native 页面加载后，进入 JavaScript 业务执行区间，大部分业务都不可避免地会进行网络交互，请求服务器数据进行渲染，这部分其实也有很大的优化空间。

首先，我们来看下具备热更新能力的 React Native 加载流程。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/481c8616d85d4bfeab95ca18c9f3d0d5~tplv-k3u1fbpfcp-zoom-1.image" />

可以看到，整个流程是从 React Native 环境初始化到热更新 ，再到 JavaScript 业务代码执行，最后到业务界面展示。链路比较长，而且每一个步骤都依赖前一个步骤的结果。特别是热更新流程，最长可涉及两次网络调用，分别是检测是否需要更新与下载最新 bundle 文件。

针对这种场景，我们想到一个优化点，在等待网络返回的过程中，Native 能不能把闲置的 CPU 资源利用起来呢？

在纯客户端开发中，我们经常使用接口数据缓存策略来提升用户体验，在最新数据返回前，先使用缓存数据进行页面渲染。那么在 React Native 中，我们也可以参考这一思路，对整个流程进行优化。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93aeddd1e96c488c8f9cd54862af3ba0~tplv-k3u1fbpfcp-zoom-1.image" />

下面我们来看一下具体如何实现。首先，当我们打开载体页时，解析对应 bundle 缓存中的预请求接口配置数据，发起请求缓存数据，并在请求成功之后缓存请求。

```
public class RNApiPreloadUtils {
    public static void preloadData(String bundleId) {
       //根据bundle id解析对应的预请求接口配置，可存在多个接口
       List<PrefetchBean> prefetchBeans = parsePrefetchBeans(bundleId);
       //请求接口，成功后缓存到本地存储
       requestDatas(prefetchBeans);
    }
    
    public static String prefetchData(String url) {
       //从本地缓存中，根据url获取对应的接口数据
    }
}
```

然后，根据 url 获取对应的缓存数据。

```
public class PreFetchBusinessModule extends ReactContextBaseJavaModule 
    implements ReactModuleWithSpec, TurboModule {
    public PreFetchBusinessModule(ReactApplicationContext reactContext) {
       super(reactContext.real());
    }

    @ReactMethod
    public void prefetchData(String url, Callback callback) {
        String data = RNApiPreloadUtils.prefetchData(url);
        // 回传数据给 JS
        WritableMap resultMap = new WritableNativeMap();
        map.putInt("code", 1);
        map.putString("data", data);
        callback.invoke(resultMap);
    }
}
```

接下来，就可以在JavaScript端调用上面的方法了，调用的代码如下：

```
NativeModules.PreFetchBusinessModule.prefetchData(url, (result)=>{
    //获取到结果后，判断是否为空，不为空解析数据后渲染页面
    console.info(result);
  }
);
```

## 四、拆包

React Native 页面的 JavaScript 代码包是热更新平台根据版本号进行下发的，每次有业务改动，我们都需要通过网络请求更新代码包。不过，只要 React Native 官方版本没有发生变化，JavaScript 代码包中 React Native 源码相关的部分是不会发生变化的，所以我们不需要在每次业务包更新的时候都进行下发，在工程中内置一份就好了。

因此，我们在对JavaScript 代码进行打包的时候，需要讲包拆分成两个部分： 一个是Common 部分，也就是 React Native 源码部分；另一个是业务代码部分，也就是我们需要动态下载的部分。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26b8ded63f68480c8bbd04f371206419~tplv-k3u1fbpfcp-zoom-1.image" />

经过上面的拆分后，Common 包内置到工程中（至少为几百 kb 的大小），业务代码包进行动态下载。然后我们利用 JSContext 环境，在进入载体页后在环境中先加载 Common 包，再加载业务代码包就可以完整的渲染出 React Native 页面，下面是iOS原生部分的加载逻辑。

```
//载体页
- (void)loadSourceForBridge:(RCTBridge *)bridge
                 onProgress:(RCTSourceLoadProgressBlock)onProgress
                 onComplete:(RCTSourceLoadBlock)loadCallback{
    if (!bridge.bundleURL) return;
    //加载新资源
    //开始加载bundle，先执行common bundle
    [RCTJavaScriptLoader loadCommonBundleOnComplete:^(NSError *error, RCTSource *source){
        loadCallback(error,newSource);
    }];
}

//common执行完毕
+ (void)commonBundleFinished{
    //开始执行buz bundle代码
     [RCTJavaScriptLoader loadBuzBundle:self.bridge.bundleURL onComplete:^(NSError *error, RCTSource *source){
        loadCallback(error,newSource);
    }];
}

//RCTJavaScriptLoader.mm
+ (void)loadBuzBundle:(NSURL *)buzURL
           onComplete:(WBSourceLoadBlock)onComplete{
    //执行buz包代码
    [self executeSource:buzURL onComplete:^(NSError *error){
      //执行完毕        
      onComplete(error);
    }];
}
```

## 五、按需加载

其实我们通过前面拆包的方案，已经减少了动态下载的业务代码包的大小。但是还会存在部分业务非常庞大，拆包后业务代码包的大小依然很大的情况，依然会导致下载速度较慢，并且还会受网络情况的影响。

因此，我们可以再次针对业务代码包进行拆分，将一个业务代码包拆分为一个主包和多个子包的方式。在进入页面后优先请求主包的 JavaScript 代码资源，能够快速地渲染首屏页面，紧接着用户点击某一个模块时，再继续下载对应模块的代码包并进行渲染，就能再进一步减少加载时间。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7cde31128cd4cd3bd7efc4b24b3ef55~tplv-k3u1fbpfcp-zoom-1.image" />

那么，什么时候需要把业务代码包拆分成一个主包和多个子包呢？把什么模块作为主包，什么模块作为子包比较合适呢？

其实，当业务逻辑比较简单的时候，我们并不需要对业务代码包进行拆分，当时当业务比较复杂的时候，特别是一些大型的项目就有可能需要进行拆包，而拆包的逻辑，通常是按照业务进行拆分的。举个例子，我们有一下这个包含 Tab 的业务页面。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d9f4d0b6cac4ef683b55e0cc0f68c03~tplv-k3u1fbpfcp-zoom-1.image" />

可以看到，页面的首页包含三个 Tab，分别表示三个不同的业务模块。如果这三个 Tab 中的内容相似，我们当然就不需要对业务代码包进行拆分了。但是如果这三个 Tab 中的内容差异化较大，页面模版完全不相同，我们就可以对业务代码包进行拆分。

## 六、其他优化

在 React Native 移动端的性能优化中，除了 React Native 环境创建、bundle 文件、接口数据等方面的优化外，还有一个大的优化点，就是 React Native 运行时优化。

众所周知，React Native 旧版本的运行效率有两大痛点：一是 JSC 引擎解释执行 JavaScript 代码效率低，引擎启动速度慢；二是 JavaScript 与 Native 通信效率低，特别是涉及批量地 UI 交互更是如此。

所以，React Native 新架构采用了 JSI 进行通信，替换了 JSBridge，无异步地序列化与反序列化操作、无内存拷贝，可以做到同步通信。

除此之外，React Native 0.60 及以后的版本开始支持 Hermes 引擎。对比 JSC 引擎，Hermes 引擎在启动速度、代码执行效率上都有大幅提升，所以接下来我们就来重点讲解 Hermes 引擎的特点、它的优化手段以及如何在移动端启用。

### 6.1 开启Hermes 引擎

Facebook 在 ChainReact 2019 大会上正式推出了新一代 JavaScript 执行引擎 Hermes。Hermes 是个轻量级的 JavaScript 引擎，专门对移动端上运行 React Native 进行了优化，Hermes 可执行字节码，也可以执行 JavaScript。

在分析性能数据时，Facebook 团队发现 JavaScript 引擎是影响启动性能和应用包体积的重要因素。JavaScriptCore 最初是为桌面浏览器端设计，相较于桌面端，移动端能力有太多的限制。所以，为了能从底层对移动端进行性能优化，Facebook 团队选择自建 JavaScript 引擎 Hermes。

依据Chain React 大会上官方给出了 Hermes 引擎一组数据，可以看出Hermes确实强大：\
从页面启动到用户可操作的时间长短（Time To Interact：TTI），从 4.3s 减少到 2.01s；\
App 的下载大小，从 41MB 减少到 22MB；\
内存占用，从 185MB 减少到 136MB。

Hermes 的优化主要体现在字节码预编译和放弃 JIT 这两点上。首先，来看下字节码预编译。现代主流的 JavaScript 引执行一段 JavaScript 代码的大概流程是：【读取源码文件】 ->【 解析转换成字节码】 ->【 执行字节码】。

不过，在运行时解析源码转换字节码是一种时间浪费，所以 Hermes 选择预编译的方式在编译期间生成字节码。这样做，一方面避免了不必要的转换时间；另一方面，多出的时间可以用来优化字节码，从而提高执行效率。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fd55c1fba9243a28c380a2037acecae~tplv-k3u1fbpfcp-zoom-1.image" />

第二点是放弃了 JIT。为了加快执行效率，现在主流的 JavaScript 引擎都会使用一个 JIT 编译器，在运行时通过转换成机器码的方式优化 JavaScript 代码。Faceback 团队认为 JIT 编译器主要有两个问题：\
要在启动时候预热，对启动时间有影响；\
会增加引擎 size 大小和运行时内存消耗。

但是这里需要注意一点，放弃了 JIT，纯文本 JavaScript 代码执行效率会降低。放弃 JIT，是指放弃运行时 Hermes 引擎对纯文本 JavaScript 代码的编译优化。当然，Hermes 也会带来一些问题，首先就是 Hermes 编译的字节码文件比纯文本 JavaScript 文件增大不少，第二点就是执行纯文本 JavaScript 耗时长。

那我们如何开启 Hermes 呢，除了可以参考官方文档快速开启 Hermes，下面我们重点看一下如何在混合工程中开启 Hermes 引擎，以 Android 为例。

1，第一步，获取 hermes.aar 文件 （目录node_modules/hermes-engine）。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20803e75f3614fddaa6950749214750f~tplv-k3u1fbpfcp-zoom-1.image" />

2，第二步，将 hermes-cppruntime-release.aar 与 hermes-release.aar 放到工程的 libs 目录总，然后在模块的 build.gradle 中添加依赖，这两个 aar 中主要是 hermes 和 libc++_shared的so文件。

```
dependencies {
    implementation(name:'hermes-cppruntime-release', ext:'aar')
    implementation(name:'hermes-release', ext:'aar')
}
```

3，第三步，设置 JavaScript 引擎。

```
ReactInstanceManagerBuilder builder = ReactInstanceManager.builder()
    .setApplication((Application) context.getApplicationContext())
    .addPackage(new MainReactPackage()) 
    .setRedBoxHandler(mExceptionHandler)
    .setUseDeveloperSupport(RNDebugSwitcher.getInstance().isDebug())
    .setInitialLifecycleState(LifecycleState.BEFORE_CREATE)
    .setJavaScriptExecutorFactory(new HermesExecutorFactory()); // 设置为 hermes
```

最后，运行 hermes 编译出的字节码 bundle 文件即可。而这一步又分为了几个小步骤：\
将 JavaScript 打包成 bundle 文件。

```
 react-native bundle --platform android --entry-file index.android.js 
--bundle-output ./bundles/index.android.bundle --assets-dest ./bundles 
--dev false
```

使用 hermes-engine 将 bundle 文件转换成字节码文件。下载 hermes-engine，使用 hermesc 命名进行转换。

```
 ./hermesc -emit-binary -out index.android.bundle.hbc 
xxx/react-native/app/bundles/index.android.bundle
```

最后，还需要重命名 bundle 文件。做法是，将之前 bundle 目录下的 index.android.bundle 删掉，然后将当前的 index.android.bundle.hbc 重命名为 index.android.bundle。

### 6.2 引擎复用

在混合应用中，React Native 由应用级的使用变更为页面级，每一个页面都使用一个 React Native 引擎 (包括 JSC/Hermes、Bridge/JSI)，除了内存占用高以外，React Native 引擎的创建耗时也是比较严重的。因此React Native另一个常见的优化就是引擎复用优化。

以 Android 为例，React Native 引擎的直接表现就是 ReactInstanceManager，内部会初始化 React Native 相关的环境。而在混合应用中，一般会配合热更新策略进行页面加载，所以使用的是 JSC/Hermes 动态加载脚本的能力。从这个场景来看，似乎一个引擎可以运行不同的 bundle 文件，即可达到复用的目的。引擎复用的坑也非常多，比如常见的有如下几个：

-   创建和复用引擎的成本可能会导致不少页面，第一次进入和后续进入的速度，表现不一致，因此这类体验问题还需要专项排查并优化；
-   在多页面同时在前台的状态下，比如首页 TAB 不同页面使用的都是 React Native 页面，会存在莫名的同步问题；
-   复用 React Native 容器内容时，会保持上一次会话的全局变量，容易造成业务逻辑错误。同一个引擎加载不同 bundle，JavaScript 上下文与新加载进去的代码能否实现 100% 隔离无污染可能是未知数。同时多页面 JavaScript 上下文隔离。目前引起复用的一大坑其实来源于 JavaScript 上下文多个页面混在一起，容易出错；
-   JSC/Hermes 随时有可能发生不可逆转的异常，因此引擎维护的过程中异常状态识别也是一个问题。

以上就是今天讲的React Native优化的一些常见点，包括环境预创建、异步更新、接口预缓存、拆包、按需加载、Hermes 引擎、引擎复用等。这些手段在实际业务中非常实用，当然 React Native 框架也在从自身上不断优化、迭代，追求性能的更高水平。
